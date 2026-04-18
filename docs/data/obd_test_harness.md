# OBD-II / CAN Bus Integration Test Harness
## Step 6 — Real Vehicle Test Bay Integration Design

**Roadmap reference:** Phase 1, Deliverable 6
**Status:** `[x]` Spec complete
**Owner:** Engineering Lead (Embedded Systems)
**Target:** Test harness operational in test bay by Month 3

---

## Objective

Build and validate the SensorFusion agent's ability to read live vehicle data from OBD-II and CAN bus, normalize it into a `UnifiedSensorState` object, and stream it to DiagnosticCore reliably. This must be validated on a real vehicle (2024 Toyota bZ4X) before any prototype demo.

---

## Test Setup

### Physical Bay Configuration

```
[bZ4X Vehicle]
    │
    ├── OBD-II Port (J1962 connector)
    │       └── Kvaser Leaf Light HS v2 (USB → CAN adapter)
    │               └── USB 3.0 → [Edge Server: Jetson AGX Orin]
    │
    ├── CAN Bus (High-speed, 500kbps + EV CAN 250kbps)
    │       └── Peak PCAN-USB Pro (direct CAN tap via breakout)
    │               └── USB 3.0 → [Edge Server]
    │
    └── Thermal Camera (FLIR A50)
            └── GigE Vision → [Edge Server: direct Ethernet]

[Edge Server: Jetson AGX Orin]
    ├── SensorFusion agent (Python / Rust)
    ├── OBD-II listener service (python-obd library, J1979)
    ├── CAN bus listener (python-can library, SocketCAN)
    └── Thermal camera reader (FLIR SDK)
            └── WiFi6E → [HoloLens 2 headset]
```

---

## OBD-II Integration

**Protocol:** ISO 15765-4 (CAN-based OBD-II), SAE J1979 Mode 01–0A

**Library:** `python-obd` v0.7.1 (well-maintained, supports async queries)

**Target PIDs for Toyota bZ4X:**

| PID | Description | Update Rate | Unit | Normal Range |
|---|---|---|---|---|
| 0x05 | Engine coolant temperature (repurposed for TMS) | 2Hz | °C | 20–80°C |
| 0x0C | Motor RPM | 10Hz | RPM | 0–12,000 |
| 0x0D | Vehicle speed | 10Hz | km/h | 0–200 |
| 0x42 | Control module voltage | 1Hz | V | 11.5–14.5V |
| 0x5C | Oil/fluid temperature | 2Hz | °C | varies |
| Toyota-specific: 0x21B4 | Battery pack temperature (avg) | 2Hz | °C | 15–45°C |
| Toyota-specific: 0x21B6 | Coolant pump flow rate | 2Hz | L/min | 0–5.0 L/min |
| Toyota-specific: 0x21C0 | HV battery voltage (pack) | 5Hz | V | 350–410V |
| Toyota-specific: 0x21C2 | HV battery current | 5Hz | A | -200 to +200A |

**Note:** Toyota-specific PIDs require `UDS (0x22)` service mode, not standard Mode 01. Requires OEM data agreement for PID list.

**DTC Query:**
- Mode 03: Read stored DTCs
- Mode 07: Read pending DTCs (not yet confirmed — critical for early fault detection)
- Mode 0A: Read permanent DTCs
- Poll interval: 5 seconds (DTCs don't change second-to-second)

---

## CAN Bus Integration

**Why CAN in addition to OBD-II?**
OBD-II is a standardized layer on top of CAN — it gives us ~30 standard PIDs. Raw CAN access gives us the full frame stream including proprietary Toyota messages (battery module-level temperatures, individual cell voltages, cooling pump PWM duty cycle). These are not available via OBD-II alone.

**Setup:**
- Tap CAN High/Low pins directly at OBD-II connector using breakout board (does not require vehicle disassembly)
- 120Ω termination resistor confirmed present on bZ4X CAN bus (no external terminator needed)
- Filter to relevant message IDs; full 500kbps stream generates ~8MB/s — must filter at hardware level

**Target CAN message IDs (Toyota bZ4X, to be confirmed with OEM):**

| CAN ID | Content | Rate |
|---|---|---|
| 0x3B3 | Battery module temperatures (M1–M6) | 100ms |
| 0x3B4 | Battery module temperatures (M7–M12) | 100ms |
| 0x3B6 | Individual cell voltages (sample) | 200ms |
| 0x5C5 | Coolant pump status + flow PWM | 100ms |
| 0x5D0 | HV contactor state (pre-charge, main+, main-) | 50ms |
| 0x5D1 | Insulation resistance monitoring | 200ms |

**CAN frame filtering:** Only forward frames matching target IDs to SensorFusion. All others dropped at edge.

---

## Thermal Camera Integration

**Device:** FLIR A50 (320×240 resolution, 30Hz, 14mm lens, GigE Vision)
**SDK:** FLIR Atlas SDK (Python bindings)

**What we extract:**
- Full thermal frame (320×240 matrix) at 10Hz (sub-sampled from 30Hz for CPU budget)
- Hotspot detection: identify pixels > threshold above local mean → flag with bounding box + centroid coordinates
- Hotspot coordinates projected to 3D space using known camera extrinsics (calibrated against ArUco markers on vehicle)

**Calibration procedure:**
1. Mount camera at fixed position above engine bay / battery compartment
2. Place ArUco markers at 4 known points on vehicle body
3. Run camera calibration script — outputs extrinsic matrix mapping pixel (u,v) to 3D point (x,y,z) in vehicle coordinate frame
4. Store calibration matrix per bay setup; re-run if camera is moved

---

## UnifiedSensorState Schema

All sensor sources merged into this structure every 100ms:

```python
@dataclass
class UnifiedSensorState:
    timestamp_ms: int
    vin: str
    
    # OBD-II derived
    vehicle_speed_kmh: float
    motor_rpm: int
    hv_voltage_v: float
    hv_current_a: float
    battery_temp_avg_c: float
    coolant_flow_lpm: float          # None if PID unavailable
    
    # DTCs
    stored_dtcs: list[str]           # e.g. ["P0A93", "P1DF1"]
    pending_dtcs: list[str]
    permanent_dtcs: list[str]
    
    # CAN-derived (module-level)
    module_temps_c: dict[str, float] # e.g. {"M1": 42.1, "M4": 87.3, ...}
    module_voltages_v: dict[str, float]
    hv_contactor_state: str          # "open", "pre-charge", "closed"
    insulation_resistance_kohm: float
    
    # Thermal camera
    thermal_hotspots: list[ThermalHotspot]  # [{centroid_3d, temp_c, bbox}]
    
    # Data quality
    stale_sources: list[str]         # sources offline > 3s
    confidence: float                 # 0.0-1.0; degrades with stale data
```

---

## Test Cases

| Test ID | Scenario | Input | Expected Output | Pass Criteria |
|---|---|---|---|---|
| TC-01 | Normal vehicle, no faults | bZ4X, all systems nominal | Empty DTC lists, all temps in range, confidence 1.0 | All fields populated, no stale sources |
| TC-02 | Injected DTC P0A93 | OBD fault injection tool | stored_dtcs includes "P0A93" | DTC present within 5s of injection |
| TC-03 | Thermal hotspot | Heat gun at Module 4 | thermal_hotspots includes centroid near M4 position | Hotspot within 50mm of actual location |
| TC-04 | CAN bus disconnect | Unplug PCAN-USB | stale_sources includes "can_bus", confidence drops | Alert within 3s; confidence < 0.7 |
| TC-05 | Full system under load | All sensors active, headset connected | UnifiedSensorState refresh < 100ms | P95 latency < 100ms over 10-min run |
| TC-06 | HV contactor state | Monitor during vehicle startup | hv_contactor_state transitions correctly | State machine matches CAN frames |
| TC-07 | Multiple simultaneous DTCs | Inject 3 DTCs | All 3 appear in stored_dtcs | All present, no dropped codes |
| TC-08 | Data freshness after 3s loss | Disconnect OBD-II for 4s, reconnect | stale_sources clears within 1s of reconnect | Recovery within 1s |

---

## Acceptance Criteria (Phase 1 Exit)

- TC-01 through TC-08: all pass
- `UnifiedSensorState` refresh rate: ≥ 10Hz sustained over 60-minute session
- DTC ingestion latency: < 5 seconds from fault injection to DTC in state
- Thermal hotspot localization accuracy: ±50mm of actual component position
- System recovers from any single sensor source loss within 3 seconds
