# HV-STOP Live Demo Specification
## Step 18 — HV-STOP Triggered and Acknowledged in Alpha Demo

**Roadmap reference:** Phase 2, Feature 3
**Status:** `[x]` Spec complete
**Owner:** PM + Safety Engineering Lead
**Target:** Demonstrated in Phase 2 internal review (Month 8)

---

## Purpose

The HV-STOP demo is a mandatory component of the Phase 2 internal review. It proves — in front of engineering and product leadership — that the safety guardrail works reliably, correctly, and within the 80ms activation target. This is not a marketing demo; it is a safety certification checkpoint.

**Demo audience:** VP Product, Engineering Lead, Safety Lead, QA Lead, Legal/Compliance observer

---

## Demo Setup

### Physical Configuration

| Item | Detail |
|---|---|
| Vehicle | 2024 Toyota bZ4X in test bay, engine off, HV bus ENERGIZED (contactor closed, 397V) |
| HV simulation | Real HV bus energized **but** demo tech wearing full PPE (1000V rated insulating gloves, face shield) |
| SafetyGuard | Running in production configuration — no test mocks |
| Operator | PM or facilitator runs the demo; tech wears headset |
| Observer | Safety Lead with stopwatch to measure 80ms target independently |
| Backup monitor | Laptop displaying SafetyGuard real-time state + audit log live |

**Safety note:** The bZ4X HV bus must be confirmed energized (CAN bus `hv_contactor_state == "closed"`, `hv_voltage_v > 350`) for the demo to be valid. A de-energized bus would only trigger WARNING, not HV_STOP, and would not represent the real safety scenario.

### Mock Approach (if full HV not available)

If the leadership review space cannot accommodate a live 400V HV bus:
- Use a vehicle with HV bus de-energized (service plug removed — SAE J1742 procedure)
- Inject CAN bus signal via Peak PCAN-USB Pro: force `hv_contactor_state = "closed"`, `hv_voltage_v = 397.0` via CAN DBC message injection
- This makes SafetyGuard believe the bus is energized without actual voltage present
- Document in demo notes that this is a safe simulation — the safety logic being tested is identical

---

## Demo Script (12 Minutes Total)

### Part 1: Context Setting (2 minutes)

**Facilitator says to audience:**
> "What you're about to see is the HV-STOP safety guardrail — one of the core safety features of X-Ray Vision Diagnostics. The system uses the headset's depth camera to track the technician's hand in 3D space. When the hand approaches a known high-voltage zone — even if the tech is looking at a CAD overlay and not at the physical cable — the system detects the proximity and stops the session immediately. I'll be running through this twice: once at normal speed, once slowly so you can see each stage."

---

### Part 2: Run 1 — Normal Speed (4 minutes)

**Step 1 (T=0):** Active diagnostic session in progress. Coolant leak overlay visible. Tech is on Step 3 of repair.

Audience sees:
- Blue coolant circuit rendered through closed hood
- Module 4 pulsing red
- Step counter: 3/7
- HUD: "Step 3 — Remove Module 4 shield"

**Step 2 (T=30s):** Tech deliberately moves right hand toward orange HV cable cluster (located at driver-side of battery, 820mm from vehicle centerline).

**T=30s + 0ms** — Hand enters 300mm proximity sphere

**Expected:** Amber ring renders around HV cable cluster (within 16ms). Single audio tone (440Hz). Repair overlay unchanged — this is only a WARNING.

**Audience marker:** "That's the 300mm warning. Hand is getting close."

**Step 3 (T=30s + 8s):** Tech continues hand approach. Hand enters 150mm sphere.

**Expected within 80ms of 150mm breach:**
1. All repair layers dim (20% opacity)
2. HV bus renders at full brightness, pulsing red+orange
3. Full-screen red safety canvas activates
4. Alarm tone (continuous, ~85dB from headset speakers)
5. Aria: "Stop. High-voltage zone — orange cables, driver side. Confirm insulated gloves, then say 'HV aware' to continue."

**Facilitator to audience:** "That's the HV-STOP. The Safety Lead's stopwatch should show under 80 milliseconds from hand crossing 150mm to the alarm sounding."

**Step 4 (T=30s + 20s):** Tech withdraws hand (> 300mm), says "HV aware."

**Expected:**
- VoiceNLP recognizes `confirm_safety` intent (< 300ms)
- Safety canvas clears
- Repair layers return to full opacity
- Aria: "Confirmed. Resuming Step 3."
- Audit log entry written (visible on laptop monitor)

---

### Part 3: Run 2 — Slow Motion Explanation (4 minutes)

Repeat the same sequence at 20% speed. Facilitator narrates each system component as it activates:

1. Hand at 300mm: "VoiceNLP and DiagnosticCore are untouched — this is SafetyGuard alone, running at 60Hz independent of the rest of the system."
2. Hand at 150mm: "Two things must be true for HV-STOP to trigger: hand within 150mm AND our CAN bus data confirms the HV contactor is closed. Both conditions are met."
3. Alarm activation: "The CADRenderer receives a direct override call from SafetyGuard — bypassing the normal manifest queue. That's why it's so fast."
4. Acknowledgment: "The acknowledgment requires the hand to withdraw AND a positive confirmation signal. Saying nothing doesn't clear it — a timeout escalates to the service manager."

---

### Part 4: Audit Log Walkthrough (2 minutes)

Show the live laptop display of `/var/log/metadome/safety_audit.ndjson`:

```json
{"event_type": "PROXIMITY_WARNING", "timestamp_ms": 1750015230445,
 "zone_id": "hv_orange_cables_left", "hand_distance_mm": 284.2,
 "hv_energized": true, "tech_id": "TECH-047", ...}

{"event_type": "HV_STOP", "timestamp_ms": 1750015238621,
 "zone_id": "hv_orange_cables_left", "hand_distance_mm": 131.8,
 "hv_energized": true, "hv_voltage_v": 397.1,
 "activation_latency_ms": 14, ...}

{"event_type": "HV_STOP_ACK", "timestamp_ms": 1750015258812,
 "ack_method": "voice", "ack_transcript": "HV aware",
 "duration_ms": 20191, "acknowledged": true, ...}
```

Facilitator: "Every HV event is logged with a chained hash — these logs cannot be modified after the fact. This is what our OSHA compliance report would contain."

---

## Measurements to Capture During Demo

The Safety Lead independently measures and records:

| Measurement | Method | Target | Result |
|---|---|---|---|
| WARNING latency (300mm → amber ring) | Stopwatch, lap at hand position + visual confirmation | < 50ms | __ |
| HV-STOP latency (150mm → alarm) | Stopwatch, lap at 150mm + alarm sound onset | < 80ms | __ |
| Aria voice response after HV_STOP | Stopwatch, alarm → first Aria word | < 500ms | __ |
| Acknowledgment latency (end of "HV aware" → overlay clears) | Stopwatch | < 500ms | __ |
| Audit log entry written | Check laptop at session end | Within 1s of event | __ |

These results are written into the Phase 2 safety audit report.

---

## Pass/Fail Criteria for Phase 2 Demo

| Criterion | Pass |
|---|---|
| HV-STOP activates on both runs | Required (100%) |
| HV-STOP latency ≤ 80ms on both runs | Required |
| Acknowledgment clears stop gate on both runs | Required |
| Audit log contains all 3 expected entries for each run | Required |
| No false positive HV-STOP during 10 minutes of normal repair activity | Required |
| Safety Lead signs off on demo | Required before Phase 3 |

**If any criterion fails:** Phase 2 is blocked. SafetyGuard must be debugged and re-tested before the Phase 2 demo is considered complete.

---

## Post-Demo: 50-Scenario Safety Audit

Immediately following the demo, engineering runs the full 50-scenario automated safety audit (`tests/safety/hv_stop_scenarios.py`). The demo counts as scenarios 1 and 2. Scenarios 3–50 are run with scripted hand position injection (no physical movement required).

Results must be documented in `docs/qa/safety_audit_report.md` (created in Phase 2 QA steps).
