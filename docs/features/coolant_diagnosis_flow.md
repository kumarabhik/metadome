# End-to-End Coolant Leak Diagnosis Flow
## Step 16 — Feature Spec: Toyota bZ4X, Full Diagnostic Session

**Roadmap reference:** Phase 2, Feature 1
**Status:** `[x]` Spec complete
**Owner:** PM + Engineering Lead
**Target:** Demonstrated in internal test bay by Month 7

---

## Overview

This document specifies the complete, end-to-end diagnostic flow for a coolant leak fault on a Toyota bZ4X. It is the first full-feature integration test of all five agents working together. Every agent interaction, state transition, timing constraint, and failure branch is defined here.

**Success definition:** A technician with zero prior exposure to the system can diagnose the source of a bZ4X coolant fault, receive 7 guided repair steps, and confirm completion — in under 90 minutes — using only voice and gaze input, with no coaching.

---

## Pre-Conditions

| Condition | Value |
|---|---|
| Vehicle | 2024 Toyota bZ4X, AWD, VIN: JTMEWRFV1PD012345 |
| Active DTCs | P0A93 (coolant temp sensor A high), P1DF1 (coolant pump performance fault) |
| Thermal anomaly | Hotspot at Module 4 (centroid: x=847, y=214, z=320 in vehicle frame) |
| Coolant flow | 1.2 L/min (nominal: 3.8 L/min) — SensorFusion reading |
| HV bus state | Energized (contactor closed, 397V) |
| Edge server | Running; all 5 agents healthy; bZ4X CAD pre-cached |
| Headset | HoloLens 2, calibrated to vehicle ArUco markers, anchor confidence > 0.85 |

---

## Flow: Step-by-Step

### Phase A: Session Initialization (T=0 to T=30s)

**T=0** — Tech dons headset, walks to bay

**T=5s** — AnchorManager detects ArUco markers, resolves vehicle anchor
- VehicleCADRoot positioned at vehicle origin
- Body ghost layer activates (12% opacity white)
- HV bus layer activates (orange, 60% opacity, ambient)

**T=8s** — SensorFusion reads VIN from OBD-II; publishes to all agents

**T=10s** — DiagnosticCore receives UnifiedSensorState:
- Active DTCs: P0A93, P1DF1
- Thermal hotspot: Module 4
- Runs RAG query speculatively (pre-warm before tech asks)
- Speculative manifest pre-computed: `tms_coolant_loop` + `bms_modules` likely needed

**T=12s** — HUD activates:
```
┌────────────────────────────────────────┐
│ Vehicle: Toyota bZ4X 2024 AWD          │
│ 2 Active DTCs detected                 │
│ Say "Hey Aria, start diagnostic"       │
│ or look at [Start] for 2s             │
└────────────────────────────────────────┘
```

---

### Phase B: Diagnostic Activation (T=30s to T=60s)

**T=32s** — Tech says: `"Hey Aria, start diagnostic"`

**T=32.05s** — Wake word detected (< 50ms on HoloLens 2)

**T=32.08s** — Audio capture begins (post-200ms gate, captures "start diagnostic")

**T=32.25s** — Whisper STT: transcript = "start diagnostic" (180ms processing)

**T=32.26s** — Intent: `navigate_step`, step direction = `start`; confidence = 0.94

**T=32.28s** — IntentPacket published to DiagnosticCore

**T=32.35s** — DiagnosticCore returns pre-warmed manifest (speculative inference hit, < 100ms):
```
LayerSelectionManifest:
  layers: [
    {id: "tms_coolant_loop",  state: "active_path",  color: "#2196F3", anim: "flow_direction"},
    {id: "bms_module_04",     state: "active_fault",  color: "#F44336", anim: "pulse_1hz"},
    {id: "coolant_pump_assy", state: "active_fault",  color: "#FF9800", anim: null}
  ]
  guidance_arrows: [
    {anchor: "coolant_inlet_M4", label: "Start here", step: 1}
  ]
  confidence: 0.91
```

**T=32.40s** — CADRenderer applies manifest (< 200ms apply time)
- Blue coolant loop renders through closed hood
- Module 4 pulses red
- Coolant pump glows amber

**T=32.50s** — Aria speaks:
> "I'm detecting a coolant fault. Flow is low and Module 4 is running hot. I've highlighted the coolant circuit in blue. The likely failure is at the Module 4 inlet coupling — it's glowing red at your 2 o'clock, about 18 inches in. Ready to begin? Say 'start repair' or nod."

**T=42s** — Tech nods (head pitch down 18°, returns to neutral within 600ms)
- `confirm_safety` intent detected from head gesture
- Step counter activates: "Step 1 of 7"

---

### Phase C: Guided Repair (T=60s to T=55min)

**Step 1 (T=60s):** Inspect Module 4 inlet coupling

Aria: *"Step 1 of 7. Inspect the Module 4 inlet coupling. I've placed an arrow at the coupling — it's at your 2 o'clock, 18 inches in on the driver side. Look for visible cracking or coolant residue around the fitting."*

Guidance arrow renders at `coolant_inlet_M4` anchor point. Step counter: 1/7.

**T=4min** — Tech completes visual inspection; says `"Next step"`
- VoiceNLP: intent=`navigate_step`, direction=`next`
- DiagnosticCore: advances to step 2, updates manifest (arrow moves to hose clamp)

**Step 2 (T=4min):** Check hose clamp integrity

**[... Steps 2–6 follow same pattern — voice navigation, arrow repositions, Aria narrates OEM procedure ...]**

**Step 7 (T=45min):** Post-repair system test

Aria: *"Step 7 of 7. Run the coolant pump test via TechStream to verify flow rate has returned to > 3.5 L/min. Watch the live reading in the top-right corner of your HUD. When it shows green, the repair is complete."*

HUD activates live coolant flow PID reading:
- Current: 1.2 L/min (red) → post-repair: 3.9 L/min (green)

**T=54min** — Flow returns to 3.9 L/min. HUD badge turns green.

---

### Phase D: HV-STOP Interrupt (T=25min, mid-repair)

*During Step 3, tech reaches across bay and moves hand near orange HV cable cluster.*

**T=25min, +0ms** — SensorFusion: hand at 280mm from `hv_orange_cables_left`

**T=25min, +16ms** — SafetyGuard tick: distance 280mm < 300mm → transition NOMINAL → WARNING

**T=25min, +18ms** — CADRenderer: amber ring renders around HV zone. Audio tone plays.

**T=25min + 8s** — Tech continues moving hand. Distance: 130mm < 150mm. HV bus energized.

**T=25min, 8s, +0ms** — SafetyGuard: transition WARNING → HV_STOP

**T=25min, 8s, +14ms** — CADRenderer.ApplyHVStop() called
- All repair layers dim to 20% opacity
- HV bus renders at full brightness (pulsing red + orange)
- Red safety overlay canvas activates
- Alarm tone plays (continuous until acknowledged)

**T=25min, 8s, +18ms** — Aria: *"Stop. High-voltage zone — orange cables. Confirm insulated gloves, then say 'HV aware' to continue."*

**T=25min, 8s + 12s** — Tech withdraws hand (> 300mm) and says *"HV aware"*
- VoiceNLP: intent=`confirm_safety`, confidence=0.95
- SafetyGuard: ACK received, transition HV_STOP → NOMINAL
- Audit log entry written
- Safety overlay clears; repair layers return to full opacity
- Aria: *"Confirmed. Continuing from Step 3."*

---

### Phase E: Session Completion (T=55min)

**T=55min** — Tech says `"Close diagnostic"`
- Intent: `end_session`
- DiagnosticCore: generates session summary
- Audit log: SESSION_END entry

**Session summary written to job card integration (Phase 3 feature; v1: written to local log):**
```json
{
  "session_id": "sess-20250715-001",
  "vin": "JTMEWRFV1PD012345",
  "tech_id": "TECH-047",
  "start_time": "09:02:14",
  "end_time": "09:57:44",
  "duration_min": 55.5,
  "dtcs_resolved": ["P0A93", "P1DF1"],
  "steps_completed": 7,
  "hv_stop_events": 1,
  "hv_stop_ack_method": "voice",
  "final_flow_rate_lpm": 3.9,
  "diagnostic_confidence": 0.91
}
```

**Headset HUD:**
> "Repair complete. 7 steps completed. Session logged. Great work."

---

## Timing Summary

| Milestone | Target Time |
|---|---|
| Vehicle identified + anchor resolved | < 10s from headset don |
| First overlay visible after "start diagnostic" | < 600ms from end of speech |
| Full manifest rendered | < 800ms from voice command |
| HV-STOP activation after 150mm breach | < 80ms |
| Step navigation ("next step" → next arrow) | < 500ms |
| Total diagnostic + repair time | < 90 minutes |

---

## Failure Branches

| Failure | System Response |
|---|---|
| DiagnosticCore confidence < 0.60 | Aria: "This fault pattern is unusual — I recommend calling OEM support. I'll display the relevant DTCs for the call." |
| VIN not in CAD database | Aria: "Vehicle model not yet supported. Displaying DTC codes only." Overlay suppressed; tablet PDF link shown |
| STT fails (voice unrecognized) | Aria: "I didn't catch that — look at [Next] in the HUD for 2 seconds." Eye-tracking fallback |
| Step 7 flow rate does not recover | Aria: "Flow rate still low after repair. This may indicate a second fault — recommend escalating to service manager." |
