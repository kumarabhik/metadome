# SensorFusion v1 — Implementation Specification
## Step 12 — OBD-II + Thermal Camera Integration, UnifiedSensorState Output

**Roadmap reference:** Phase 2, Agent Build 2
**Status:** `[x]` Spec complete
**Owner:** Embedded Systems Engineer
**Target:** Working v1 by Month 6 (parallel with DiagnosticCore)

---

## Responsibility

SensorFusion aggregates all real-time vehicle data streams into a single `UnifiedSensorState` struct, published at 10Hz to a shared in-memory state bus on the edge server. It is the only agent that touches raw hardware interfaces. All other agents consume `UnifiedSensorState` — they never read sensors directly.

**Critical property:** SensorFusion must degrade gracefully. If any source goes offline, it continues publishing with that source marked stale and confidence reduced. It never blocks or crashes the pipeline.

---

## v1 Scope

**Sources integrated in v1:**
- OBD-II (J1979 Mode 01, 03, 07, 0A + Toyota UDS 0x22 PIDs)
- CAN bus (filtered high-priority frames via PCAN-USB Pro)
- Thermal camera (FLIR A50 via GigE Vision / FLIR Atlas SDK)
- Headset depth camera feed (hand/tool 3D position, via HoloLens 2 Spatial Perception API)

**Out of scope for v1:**
- OEM telematics REST API (added Phase 3 — requires dealer IT integration)
- Multiple vehicles simultaneously (single-bay, single-VIN in v1)

---

## Runtime Architecture

```
[Thread 1] OBD-II Poller          (asyncio, 10Hz PID queries, 0.2Hz DTC queries)
[Thread 2] CAN Bus Listener       (SocketCAN blocking read, interrupt-driven)
[Thread 3] Thermal Camera Reader  (FLIR SDK callback, 10Hz sub-sampled from 30fps)
[Thread 4] Depth Camera Proxy     (gRPC stream from HoloLens 2 companion app, 60Hz)
[Thread 5] State Publisher        (10Hz timer, merges all sources → publishes)

All threads write to thread-safe SourceBuffers.
State Publisher reads all buffers atomically → builds UnifiedSensorState → publishes.
```

**State bus:** In-process Python `asyncio.Queue` for local agents; gRPC stream for remote consumers (headset companion app). No external message broker in v1 (unnecessary for single-bay deployment).

---

## Source Specifications

### OBD-II Poller

**Library:** `python-obd` 0.7.1 with async extensions

**Query strategy — two loops:**

*Fast loop (10Hz):* Live PIDs — values that change second-to-second
```python
FAST_PIDS = [
    "0x05",    # coolant/TMS temp
    "0x0C",    # motor RPM
    "0x0D",    # vehicle speed
    "0x42",    # control module voltage
    "0x21B4",  # battery pack avg temp (Toyota UDS)
    "0x21B6",  # coolant pump flow rate (Toyota UDS)
    "0x21C0",  # HV pack voltage (Toyota UDS)
    "0x21C2",  # HV pack current (Toyota UDS)
]
```

*Slow loop (0.2Hz / every 5s):* DTCs — don't change between slow polls
```python
DTC_MODES = ["MODE_03", "MODE_07", "MODE_0A"]  # stored, pending, permanent
```

**OBD-II connection sequence:**
1. Connect to Kvaser Leaf Light HS v2 via USB (appears as `/dev/ttyUSB0` on Jetson)
2. Auto-detect protocol (ISO 15765-4 CAN 11-bit 500kbps for bZ4X)
3. Switch to UDS extended addressing for Toyota-specific PIDs after standard PIDs confirmed
4. If connection fails: retry every 2s for 30s; then mark source `OFFLINE`, publish stale flag

**Timeout handling:** Any single PID query > 200ms → skip that PID this cycle, log timeout, decrement confidence

---

### CAN Bus Listener

**Library:** `python-can` 4.3, `SocketCAN` backend (Jetson Linux native)

**Hardware:** Peak PCAN-USB Pro → USB → Jetson → appears as `can0` SocketCAN interface

**Initialization:**
```bash
ip link set can0 up type can bitrate 500000 dbitrate 2000000 fd on
```

**Frame filter (hardware-level, applied at kernel):**
Only these CAN IDs forwarded to userspace — all others dropped at kernel level:
```
0x3B3, 0x3B4  — battery module temps (M1–M12)
0x3B6         — cell voltage samples
0x5C5         — coolant pump status + PWM
0x5D0         — HV contactor state
0x5D1         — insulation resistance
```

**Message decoder:** Lookup table mapping `(can_id, bit_offset, bit_length, scale, offset)` → named signal. Table stored in `configs/can_dbc/toyota_bz4x_2024.dbc` (DBC format, standard automotive).

**Buffer:** Ring buffer of last 100 frames per CAN ID. State Publisher reads latest value from each ID.

---

### Thermal Camera Reader

**SDK:** FLIR Atlas SDK (Python 3.11 bindings, Linux ARM64 build for Jetson)

**Acquisition:**
- Camera connects via GigE Vision (direct Ethernet to Jetson, static IP `192.168.1.100`)
- Stream at 30fps, 320×240 14-bit radiometric
- Sub-sampled to 10fps in callback (every 3rd frame processed)

**Hotspot detection algorithm:**
```python
def detect_hotspots(frame: np.ndarray, threshold_delta_c: float = 15.0) -> list[ThermalHotspot]:
    # frame: 320x240 array of temperatures in Celsius
    local_mean = gaussian_filter(frame, sigma=30)  # large-kernel smoothing = local background
    delta = frame - local_mean
    hotspot_mask = delta > threshold_delta_c

    # Label connected regions
    labeled, n_regions = label(hotspot_mask)
    hotspots = []
    for region_id in range(1, n_regions + 1):
        region = (labeled == region_id)
        if region.sum() < 9:  # minimum 3x3 pixels to filter noise
            continue
        centroid_px = center_of_mass(region)
        max_temp = frame[region].max()
        # Project pixel centroid to 3D vehicle frame via pre-calibrated extrinsics
        centroid_3d = project_to_vehicle_frame(centroid_px, extrinsics=CAMERA_EXTRINSICS)
        hotspots.append(ThermalHotspot(
            centroid_3d=centroid_3d,
            temp_c=max_temp,
            area_px=region.sum(),
            confidence=min(1.0, region.sum() / 100)  # larger region = higher confidence
        ))
    return hotspots
```

**Camera extrinsics:** Pre-calibrated matrix stored in `configs/thermal/bay01_extrinsics.json`. Re-calibration required if camera is moved (ArUco-based calibration procedure in `docs/prototype/overlay_spec.md`).

---

### Depth Camera Proxy (Hand/Tool Tracking)

**Source:** HoloLens 2 Spatial Perception API via Unity C# companion component

**What it provides:** 3D position of detected hands (wrist, palm, fingertips — 26 joints per hand) in world coordinate frame, at 60Hz

**Transport:** gRPC stream from HoloLens companion app → Edge server `SensorFusion` listener

**On edge server:** Transforms hand positions from world frame to vehicle frame (inverse of the ArUco-derived vehicle-to-world transform). Publishes closest hand distance to each known HV bus anchor point — consumed by SafetyGuard at 60Hz.

**Critical path:** Hand position data for SafetyGuard is NOT routed through the 10Hz State Publisher. It has its own dedicated 60Hz push channel directly to SafetyGuard to meet the < 80ms HV-STOP activation target.

---

## UnifiedSensorState — Full Schema

```python
@dataclass
class UnifiedSensorState:
    # Metadata
    timestamp_ms: int
    session_id: str
    vin: str
    sequence_number: int          # monotonic, for detecting dropped updates

    # OBD-II / UDS — vehicle dynamics
    vehicle_speed_kmh: float
    motor_rpm: int
    control_voltage_v: float

    # OBD-II / UDS — TMS specific
    battery_temp_avg_c: float
    coolant_flow_lpm: float
    hv_voltage_v: float
    hv_current_a: float

    # OBD-II DTCs
    stored_dtcs: list[str]
    pending_dtcs: list[str]
    permanent_dtcs: list[str]
    dtc_timestamp_ms: int         # when DTC list was last refreshed

    # CAN — module-level
    module_temps_c: dict[str, float]         # "M1"–"M12"
    module_voltages_v: dict[str, float]      # "M1"–"M12"
    coolant_pump_pwm_pct: float              # 0–100%
    hv_contactor_state: str                  # "open"|"pre_charge"|"closed"|"fault"
    insulation_resistance_kohm: float

    # Thermal camera
    thermal_hotspots: list[ThermalHotspot]

    # Hand tracking (60Hz channel, latest value here for slow consumers)
    nearest_hand_to_hv_mm: float             # minimum distance, any hand, any HV anchor
    hand_positions_3d: dict[str, Point3D]    # "left_palm", "right_palm", etc.

    # Data quality
    source_status: dict[str, SourceStatus]   # per-source: ONLINE | STALE | OFFLINE
    confidence: float                         # 0.0–1.0; 1.0 = all sources fresh

@dataclass
class SourceStatus:
    status: str          # "ONLINE" | "STALE" | "OFFLINE"
    last_update_ms: int
    error_message: str | None
```

---

## Confidence Calculation

```python
def compute_confidence(source_status: dict[str, SourceStatus], now_ms: int) -> float:
    weights = {"obd2": 0.35, "can_bus": 0.30, "thermal": 0.25, "depth_camera": 0.10}
    score = 0.0
    for source, weight in weights.items():
        status = source_status[source]
        age_s = (now_ms - status.last_update_ms) / 1000
        if status.status == "ONLINE" and age_s < 1.0:
            score += weight * 1.0
        elif age_s < 3.0:
            score += weight * 0.5   # stale but recent
        else:
            score += weight * 0.0   # offline or very stale
    return round(score, 2)
```

---

## Failure Modes & Recovery

| Failure | Behavior | Recovery |
|---|---|---|
| OBD-II disconnect | Mark STALE after 1s; OFFLINE after 3s; confidence drops | Auto-reconnect every 2s; resume immediately on reconnect |
| CAN bus silent (no frames > 3s) | Mark can_bus STALE; alert DiagnosticCore | Check `can0` interface up; attempt `ip link set can0 down/up` reset |
| Thermal camera offline | Mark thermal STALE; suppress thermal hotspot data | Alert tech: "Thermal view unavailable — check camera cable" |
| Depth camera stream drop | Mark depth_camera STALE; SafetyGuard falls back to conservative mode (assume hand near HV) | Alert headset: "Hand tracking lost — HV safety in conservative mode" |
| All sources offline | Publish empty state, confidence=0.0; DiagnosticCore escalates | Alert tech; alert manager dashboard |

---

## Performance Targets

| Metric | Target |
|---|---|
| State publish rate | 10Hz (100ms period) |
| OBD-II PID round-trip | < 80ms per PID (ISO 15765-4 spec) |
| CAN frame to state | < 5ms (interrupt-driven) |
| Thermal hotspot detection | < 30ms per frame |
| Hand position to SafetyGuard | < 5ms (dedicated 60Hz channel) |
| Memory footprint (Jetson) | < 200MB resident |
