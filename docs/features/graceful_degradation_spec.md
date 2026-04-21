# Graceful Degradation UX Spec
## Step 28 — Phase 3, Feature 4

**Roadmap reference:** Phase 3, Feature 4
**Status:** `[ ]` Not started
**Owner:** PM + UX + Engineering
**Target:** Implemented before beta deployment, Month 9

---

## Objective

Define the exact UX behavior when the system cannot meet its latency SLA or loses sensor fidelity. Technicians must never be left with a frozen, confusing, or silently degraded experience. Degradation must be honest, legible, and safe.

**Core design principle:** A degraded system that clearly communicates its limitations is safer than a system that silently lies about its confidence.

---

## Degradation Trigger Conditions

| Condition | Trigger Threshold | Severity |
|---|---|---|
| End-to-end render latency | p95 > 40ms over 10s window | Yellow |
| LLM response time | > 1,500ms per call | Yellow |
| Network packet loss | > 5% over 30s window | Yellow |
| Ambient lighting (headset cameras) | < 30 lux detected | Yellow |
| SLAM anchor confidence | < 0.65 (scale 0–1) | Orange |
| Thermal camera signal lost | Camera offline > 3s | Orange |
| OBD-II / CAN bus signal lost | No data for > 5s | Orange |
| Edge server CPU saturation | > 95% for > 10s | Orange |
| SensorFusion agent offline | Agent health check fails | Red |
| DiagnosticCore agent offline | Agent health check fails | Red |
| CADRenderer frame rate | < 30fps for > 5s | Red |
| HV bus state unknown | OBD-II disconnected | Red — safety critical |

---

## Degradation Levels

### Yellow — "Reduced Performance"

**What happens to UX:**
- Small amber indicator badge appears in top-right of headset view (⚠ icon, 20px)
- CAD overlays continue to render; no visual change to content
- Voice commands continue to function
- If tech asks "What's wrong with the system?" → VoiceNLP responds: "Running a bit slow right now. I'll catch up in a moment."

**What happens to system:**
- CADRenderer drops to LOD 1 for all layers (reduces GPU load)
- Speculative pre-rendering disabled (conserves CPU)
- DiagnosticCore disables streaming tokens; waits for full response before display

**Logging:** Yellow events logged to edge server with duration and trigger condition.

---

### Orange — "Limited Mode"

**What happens to UX:**
- Full-width orange banner at top of headset view: **"Limited mode — [specific reason]. Some features may be delayed."**
- Examples of reason strings:
  - "Move to better lighting to improve accuracy"
  - "CAD position may drift slightly — anchor re-establishing"
  - "Thermal imaging offline — visual guidance reduced"
- Voice commands continue
- CAD overlay remains but anchor confidence warning badge added to overlay border

**What happens to system:**
- CADRenderer renders in 2D schematic mode if SLAM anchor confidence < 0.65 (flat diagram pinned to headset view rather than 3D overlay on vehicle)
- SensorFusion: if thermal camera lost, publishes `UnifiedSensorState` with `thermal_available: false` — DiagnosticCore adjusts diagnosis confidence score accordingly
- If OBD-II lost: DiagnosticCore pauses DTC-based logic; continues with last known state; timestamps it

**Voice notification:** One-time audio: "I've switched to limited mode — [reason]. I'll let you know when I recover."

---

### Red — "Minimum Safe Mode"

**What happens to UX:**
- Full-screen red banner: **"System reduced to safety mode. CAD overlay paused."**
- All CAD layers hidden except: HV bus visualization (always shown if bus state known) and HV-STOP proximity alerts
- Voice commands restricted to: "Status?", "Resume?", "Exit session"
- If tech says "Resume?" → system checks health and recovers to Orange or Yellow if possible

**What happens to system:**
- CADRenderer stops rendering content layers; maintains HV-STOP visual layer only
- DiagnosticCore pauses — no new manifest generation
- SafetyGuard continues to run at full priority (never degraded)

**Voice notification:** "I've paused the overlay for safety. HV proximity alerts are still active. Say 'resume' when you're ready to try again."

---

### Red — HV Bus State Unknown (Special Case)

This is a special-case Red that triggers immediately regardless of other system health:

**Behavior:**
1. Audio alarm: three ascending tones
2. Full-screen red banner: **"High voltage bus status unknown. Do not approach HV components. Verify bus isolation manually."**
3. HV-STOP activates: all CAD overlays freeze, audio/visual stop signal plays
4. Session cannot resume until OBD-II reconnects and HV bus voltage confirmed < 5V

This scenario is treated as a potential safety hazard. It cannot be dismissed by the technician — only by the system confirming HV bus state.

---

## "Move to Better Lighting" UX

When ambient lighting < 30 lux (Orange trigger):

1. CAD overlay dims to 50% opacity
2. Directional arrow overlay appears pointing toward the nearest lit area (determined by headset photosensor)
3. Banner: **"Move to better lighting to improve overlay accuracy"**
4. When lighting recovers to > 50 lux: banner auto-dismisses; overlay returns to 100% opacity

This is the most common degradation scenario in real dealership bays (bay lighting is often inconsistent).

---

## Recovery Behavior

When the trigger condition clears:
- System automatically attempts recovery (no user action required)
- Recovery confirmed when condition reads nominal for 10 consecutive seconds
- Orange → Yellow: banner changes from orange to amber; reduced LOD continues for 30s then re-enables
- Yellow → Normal: badge disappears; LOD resets; speculative pre-rendering re-enables
- Red → Orange/Yellow: requires explicit tech "Resume?" command before overlay re-renders

---

## Logging Schema

```json
{
  "event_type": "degradation_start" | "degradation_end",
  "session_id": "session_uuid",
  "timestamp_utc": "...",
  "trigger": "ambient_lighting" | "slam_anchor" | "hv_unknown" | ...,
  "severity": "yellow" | "orange" | "red",
  "trigger_value": 22,
  "trigger_threshold": 30,
  "duration_seconds": 47
}
```

---

## Exit Criteria

- [ ] All 13 trigger conditions implemented and tested in isolation on test hardware
- [ ] Yellow / Orange / Red visual states verified on HoloLens 2 display (QA checklist)
- [ ] "Move to better lighting" arrow renders correctly in < 2s of trigger
- [ ] HV unknown state cannot be dismissed manually — verified in 10 consecutive test runs
- [ ] Recovery from Orange to Normal occurs within 15s of condition clearing
- [ ] Degradation events logged correctly and visible in Manager Dashboard
