# SafetyGuard v1 — Implementation Specification
## Step 15 — HV Proximity Detection, HV-STOP Protocol, Audit Logging

**Roadmap reference:** Phase 2, Agent Build 5
**Status:** `[x]` Spec complete
**Owner:** Safety Engineering Lead + Embedded Systems Engineer
**Target:** Working v1 by Month 6 (must pass safety audit before Phase 2 demo)

---

## Responsibility

SafetyGuard is the only agent with unconditional authority over the render pipeline. It runs as a separate OS process with real-time scheduling priority (`SCHED_FIFO`, priority 80 on Linux) — it cannot be starved by the main application.

**Non-negotiables:**
1. SafetyGuard cannot be paused, stopped, or overridden by any other agent or by the technician
2. HV-STOP activation must complete (headset alert delivered) within **80ms** of hand entering 150mm proximity zone
3. All events logged to tamper-evident append-only log — no log entry may ever be deleted or modified
4. If SafetyGuard process crashes, the edge server watchdog detects the crash within 1 second and triggers a fail-safe alert on the headset

---

## v1 Scope

- HV bus proximity detection (Toyota bZ4X HV bus anchor positions only)
- HV contactor state awareness (energized vs. de-energized)
- Three-tier alert system (warning → stop → escalation)
- Technician acknowledgment via voice ("HV aware") or head nod
- Tamper-evident audit log
- Manager escalation via HTTP push to dashboard

---

## HV Anchor Map (Toyota bZ4X)

All HV bus locations in vehicle frame (from CAD data), stored in `configs/safety/hv_anchors_toyota_bz4x.json`:

```json
{
  "vehicle": "toyota_bz4x_2024",
  "hv_zones": [
    {
      "zone_id": "hv_main_bus_front",
      "description": "Main HV bus connector, front of battery pack",
      "centroid": {"x": 450, "y": 0, "z": 280},
      "radius_mm": 120,
      "nominal_voltage_v": 400,
      "color": "#FF6F00"
    },
    {
      "zone_id": "hv_orange_cables_left",
      "description": "Orange HV cables, driver side routing",
      "centroid": {"x": 600, "y": 820, "z": 250},
      "radius_mm": 80,
      "nominal_voltage_v": 400,
      "color": "#FF6F00"
    },
    {
      "zone_id": "hv_inverter_terminals",
      "description": "Inverter HV terminals (DC link)",
      "centroid": {"x": 380, "y": 0, "z": 420},
      "radius_mm": 100,
      "nominal_voltage_v": 400,
      "color": "#FF6F00"
    },
    {
      "zone_id": "hv_charger_port",
      "description": "On-board charger HV connection",
      "centroid": {"x": 200, "y": -750, "z": 350},
      "radius_mm": 90,
      "nominal_voltage_v": 400,
      "color": "#FF6F00"
    }
  ]
}
```

---

## HV-STOP State Machine

```
                    ┌──────────┐
                    │  NOMINAL  │  All hands > 300mm from all HV zones
                    └────┬─────┘
                         │ hand enters 300mm sphere
                         ▼
                    ┌──────────┐
                    │  WARNING  │  Amber ring renders around HV zone
                    └────┬─────┘  Audio: single tone (440Hz, 200ms)
                         │ hand enters 150mm sphere AND hv_contactor == "closed"
                         ▼
                    ┌──────────┐
                    │ HV_STOP  │  Full red overlay, alarm, pipeline paused
                    └────┬─────┘
          ┌──────────────┴──────────────┐
          │ tech acknowledges            │ 10s timeout, no ack
          │ (voice "HV aware" or nod)    ▼
          ▼                        ┌──────────────┐
     ┌──────────┐                  │  ESCALATED    │
     │  NOMINAL  │ (if hand         │  Manager alerted,
     └──────────┘  withdrawn)       │  bay locked in dashboard
```

**Special case — de-energized HV bus:**
- If `hv_contactor_state == "open"` AND `hv_voltage_v < 60V` (low-voltage safe threshold per SAE J1742): downgrade HV_STOP to WARNING only
- Always log de-energized proximity events regardless

---

## Proximity Detection Implementation

**Dual-source confirmation required for HV_STOP (prevents false positives):**

| Source | Data | Rate |
|---|---|---|
| HoloLens 2 depth camera | Hand palm position in world frame, via gRPC | 60Hz |
| SensorFusion CAN data | `hv_contactor_state` (energized/de-energized) | 10Hz (CAN source) |

**Algorithm (runs at 60Hz in dedicated loop):**

```python
class SafetyGuardLoop:
    def __init__(self, hv_anchors: list[HVZone], config: SafetyConfig):
        self.hv_anchors = hv_anchors
        self.state = SafetyState.NOMINAL
        self.nearest_distances: dict[str, float] = {}  # zone_id → mm

    def tick(self, hand_positions: list[Point3D], hv_energized: bool) -> None:
        min_distance = float('inf')
        nearest_zone = None

        for zone in self.hv_anchors:
            for hand_pos in hand_positions:
                dist = euclidean_distance(hand_pos, zone.centroid_world) - zone.radius_mm
                dist = max(0.0, dist)  # clamp to 0 if inside zone
                if dist < min_distance:
                    min_distance = dist
                    nearest_zone = zone

        self.nearest_distances[nearest_zone.zone_id if nearest_zone else "none"] = min_distance

        # State transitions
        if min_distance > 300:
            self._transition_to(SafetyState.NOMINAL)
        elif min_distance <= 300 and min_distance > 150:
            self._transition_to(SafetyState.WARNING, zone=nearest_zone)
        elif min_distance <= 150 and hv_energized:
            self._transition_to(SafetyState.HV_STOP, zone=nearest_zone)
        elif min_distance <= 150 and not hv_energized:
            self._transition_to(SafetyState.WARNING, zone=nearest_zone)  # de-energized: warning only

    def _transition_to(self, new_state: SafetyState, zone=None):
        if new_state == self.state:
            return  # no transition, no log noise
        old_state = self.state
        self.state = new_state
        self._log_transition(old_state, new_state, zone)
        self._notify_cad_renderer(new_state, zone)
        if new_state == SafetyState.HV_STOP:
            self._start_acknowledgment_timer()
```

**Coordinate transform:** Hand positions from HoloLens 2 are in world frame. SafetyGuard maintains the current `world_to_vehicle_transform` (updated whenever AnchorManager reports a new pose) to convert hand positions to vehicle frame for distance calculation against zone centroids.

---

## Acknowledgment Protocol

**Accepted acknowledgment signals:**

| Signal | Detection Method | Confidence Threshold |
|---|---|---|
| Voice: "HV aware" | VoiceNLP `confirm_safety` intent | ≥ 0.80 |
| Voice: "confirmed" / "I confirm" | VoiceNLP `confirm_safety` intent | ≥ 0.80 |
| Head nod | HoloLens 2 head pose: downward pitch ≥ 15° then return to neutral within 800ms | N/A (deterministic) |

**Acknowledgment is NOT accepted if:**
- Hand is still within 150mm of HV zone (must withdraw hand before acknowledging)
- Acknowledgment comes from a recognized non-technician voice (multi-voice disambiguation in Phase 5; in v1, any voice accepted with a logged caveat)

**After acknowledgment:**
- State returns to NOMINAL (if hand withdrawn) or WARNING (if hand still in 300mm zone but > 150mm)
- HV-STOP event logged as `acknowledged`
- Manager dashboard notified: stop event + acknowledgment time

---

## Audit Logging

**Log format:** NDJSON (newline-delimited JSON), append-only, written to `/var/log/metadome/safety_audit.ndjson`

**Tamper-evidence:** Each log entry includes a SHA-256 chained hash of the previous entry. Any modification of a past entry breaks the hash chain — detectable by `audit_verify.py`.

```json
{
  "entry_id": "safe-20250615-00847",
  "prev_hash": "a3f2c1d8...",
  "timestamp_ms": 1750012845231,
  "session_id": "sess-abc123",
  "tech_id": "TECH-047",
  "vehicle_vin": "JTMEWRFV1PD012345",
  "bay_id": "bay-01",
  "event_type": "HV_STOP",
  "zone_id": "hv_orange_cables_left",
  "hand_distance_mm": 112.4,
  "hv_energized": true,
  "hv_voltage_v": 397.2,
  "duration_ms": 8400,
  "acknowledged": true,
  "ack_method": "voice",
  "ack_transcript": "HV aware",
  "manager_escalated": false,
  "entry_hash": "7b91e4a2..."
}
```

**Event types logged:**
- `PROXIMITY_WARNING` — hand enters 300mm zone
- `HV_STOP` — hand enters 150mm zone with energized bus
- `HV_STOP_ACK` — technician acknowledged stop gate
- `HV_STOP_TIMEOUT` — 10s elapsed without acknowledgment
- `MANAGER_ESCALATION` — dashboard notified
- `PROXIMITY_DEENERGIZED` — hand in 150mm zone but bus de-energized (warning only)
- `SESSION_START` / `SESSION_END` — with tech ID, VIN, bay

**Retention:** 12 months on edge server; auto-archived to cloud (encrypted S3) at 30 days; 36-month cloud retention.

---

## Manager Escalation

When HV_STOP is not acknowledged within 10 seconds:

```python
def escalate_to_manager(stop_event: HVStopEvent):
    payload = {
        "alert_type": "HV_STOP_UNACKNOWLEDGED",
        "bay_id": stop_event.bay_id,
        "tech_id": stop_event.tech_id,
        "vehicle_vin": stop_event.vin,
        "zone": stop_event.zone_id,
        "elapsed_ms": stop_event.elapsed_ms,
        "timestamp": stop_event.timestamp_iso8601
    }
    # POST to manager dashboard (local bay network — no cloud dependency)
    requests.post("http://dashboard.bay-local:8080/api/alerts", json=payload, timeout=2.0)
    # Also send push notification via Twilio SMS if configured (optional, Phase 3)
```

**Bay lock:** Sets a software flag `bay_locked = true` in edge server state. Prevents new diagnostic sessions from starting in the bay until a manager clears it via dashboard. Does not interrupt an active repair session that is already in progress.

---

## Watchdog

Separate `systemd` service monitors SafetyGuard process:

```ini
[Unit]
Description=MetaDome SafetyGuard Watchdog
After=safety-guard.service

[Service]
Type=simple
ExecStart=/usr/bin/python3 /opt/metadome/safety_watchdog.py
Restart=always
RestartSec=1

[Install]
WantedBy=multi-user.target
```

If SafetyGuard process PID is not found within 1 second: watchdog sends `HV_STOP` signal directly to CADRenderer (fail-safe to red overlay) and attempts to restart SafetyGuard. Logs `SAFETY_GUARD_CRASH` event.

---

## Phase 2 Safety Audit Requirement

Before Phase 2 demo, SafetyGuard must pass 50 simulated proximity test cases:
- 20 cases: hand approaches at normal speed (200mm/s), stops at 120mm → must trigger HV_STOP
- 15 cases: hand moves fast (500mm/s), crosses 150mm threshold → must trigger within 80ms
- 10 cases: hand at 160mm (just outside stop zone) → must NOT trigger HV_STOP (only WARNING)
- 5 cases: bus de-energized + hand at 100mm → WARNING only, not STOP

Pass rate required: **100%**. Any failure is a blocker for Phase 2 demo.
