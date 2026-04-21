# Apprentice Training Mode

**Phase:** 5
**Roadmap Item:** Feature: Apprentice training mode — step validation slower, more explanation, score tracked for OEM certification programs

---

## Objective

Create a distinct operating mode for technicians-in-training (apprentices, lube techs advancing to diagnostic roles, new hires) where the system:
1. Provides richer explanations at each step — *why*, not just *what*
2. Validates each step before proceeding — the tech must confirm completion and answer a brief checkpoint question
3. Scores the repair session and exports the score to OEM technician certification programs

This mode does not change the underlying diagnostic logic or safety guardrails — HV-STOP operates identically. It changes pacing, depth of instruction, and evaluation.

---

## Mode Activation

### Who Uses It
- Apprentice technicians in their first 12 months
- Lube technicians cross-training to diagnostic roles
- Returning technicians recertifying on new vehicle model after OEM curriculum update

### How It's Activated
- **Manager-assigned:** Manager dashboard (Phase 3) allows service manager to assign Apprentice Mode to a technician's headset profile — all sessions for that tech run in Apprentice Mode until reassigned
- **Session-level override:** Senior tech can temporarily enable Apprentice Mode for a training session without changing the profile (voice command: "Enable training mode" while supervising)
- Visual indicator: headset HUD shows a graduation cap icon in the corner while Apprentice Mode is active — tech and any observing manager can see mode is on

---

## UX Differences vs. Standard Mode

### Standard Mode
- Step card appears → brief instruction → tech performs action → says "Done" → next step
- Explanation depth: minimal (assumed expert knows why)
- Pacing: tech-controlled, no gate

### Apprentice Mode

**Step card structure (expanded):**
```
STEP 3 OF 7: Disconnect coolant inlet line

[WHY] The coolant inlet must be depressurized before disconnection.
      Residual pressure can spray coolant, creating burn risk.
      This vehicle's TMS runs at 18 PSI nominal.

[WHAT] Locate the blue coolant inlet line at the top of the battery module.
       Use the quick-release tool (Part #TY-4402). Twist 90° counterclockwise.

[WATCH FOR] Coolant drip is normal (< 50ml). Significant flow = system not
            depressurized. Stop and re-verify Step 2.

[Diagram: exploded CAD view with inlet line highlighted in blue]
```

**Checkpoint gate (before advancing):**
- After tech says "Done", a checkpoint question appears before the next step unlocks
- Questions are short, contextual to the step just completed
- Example: "What pressure does this TMS run at?" → tech selects from 3 options
- Correct: "Good — proceed to Step 4" → next step unlocks
- Incorrect: explanation shown + tech can retry OR flag for supervisor review
- Questions generated from OEM service bulletin content (not freeform LLM) — same source-of-truth as DiagnosticCore

**Pacing:**
- Minimum dwell time per step: 15 seconds (prevents button-mashing through checkpoints)
- No maximum — tech can stay on a step as long as needed
- Timer visible to tech: "You've spent 4:32 on this step"

---

## Scoring System

### Session Score Components

| Component | Weight | Description |
|---|---|---|
| Checkpoint accuracy | 40% | % of checkpoint questions answered correctly on first attempt |
| Step completion order | 20% | Were all steps completed in sequence without skipping |
| Time efficiency | 20% | Session time vs. expected time for novice (p50 of first-session techs for this fault type) |
| Safety compliance | 20% | HV-STOP protocol followed correctly if triggered; no bypassed safety steps |

### Score Output
- Session score: 0–100
- Passed: ≥ 70
- Letter grade: A (90–100), B (80–89), C (70–79), Not Passed (< 70)
- Fault-type specific: scored per fault type, not overall — a tech can be A-grade on coolant leaks and C-grade on inverter faults

### Score Record Schema
```json
{
  "session_id": "uuid",
  "tech_id": "string",
  "fault_type": "string",
  "vehicle_model": "string",
  "date": "ISO8601",
  "overall_score": "float",
  "grade": "A | B | C | Not Passed",
  "checkpoint_accuracy": "float",
  "step_sequence_score": "float",
  "time_efficiency_score": "float",
  "safety_score": "float",
  "checkpoints": [
    {
      "step_number": "int",
      "question": "string",
      "tech_answer": "string",
      "correct": "boolean",
      "attempts": "int"
    }
  ],
  "supervisor_review_flags": ["step_number"]
}
```

---

## OEM Certification Integration

### Why OEMs Care
Toyota, Ford, and Stellantis all run mandatory technician certification programs (Toyota T-TEN, Ford FACT, Stellantis CAP). These programs require technicians to demonstrate competency on specific repair tasks. Today this is done via written tests and supervised in-bay assessments — labor-intensive for OEM training teams.

X-Ray Vision Diagnostics Apprentice Mode produces an objective, timestamped, per-fault record of competency. We can replace or supplement the in-bay assessment component.

### Integration Approach
- Score records exported via API to OEM's Learning Management System (LMS)
- Toyota: integrates with Toyota Training Management System (TTMS) — REST API, JSON payload
- Ford: integrates with Ford Motorcraft Training portal — existing API, same payload structure
- Export trigger: session complete + score ≥ 70
- OEM maps our fault-type scores to their certification module IDs — mapping table maintained in OEM partnership config

### What OEMs Receive Per Session
- Tech ID (mapped to OEM employee ID via enrollment)
- Fault type + vehicle model
- Score and grade
- Date/time
- Pass/fail
- Checkpoint detail (optional — OEM configures whether they want granular or summary only)

### Privacy Note
- Score data is the dealer's / tech's data — OEM only receives it with tech consent (obtained at enrollment)
- Tech can view their own complete history; manager sees team history; OEM sees only enrolled techs

---

## Manager Dashboard Integration

Add "Training" tab to the manager dashboard (Phase 3 spec):
- View all techs in Apprentice Mode
- Per-tech score history by fault type
- "Ready to promote" flag: tech has ≥ B grade on ≥ 3 fault types in last 30 sessions
- Alert: tech has ≥ 3 consecutive "Not Passed" sessions on same fault type → flag for supervisor review

---

## Rollout Plan

| Month | Milestone |
|---|---|
| 24 | UX design: expanded step card, checkpoint question format |
| 25 | Build checkpoint question library for all 5 enrolled fault types (sourced from OEM bulletins) |
| 25 | Build scoring engine and session score schema |
| 26 | Build manager dashboard training tab |
| 27 | Toyota TTMS API integration (pilot) |
| 28 | Closed beta: 2 Toyota dealers with active apprentice programs |
| 29 | Ford LMS integration |
| 30 | GA — available on all plans; OEM certification integration on Professional/Enterprise tiers |

---

## Success Metrics

| Metric | Target |
|---|---|
| Time-to-competency (first B grade) | ≤ 8 sessions for any fault type (vs. no baseline currently) |
| Checkpoint accuracy improvement session-over-session | ≥ 5% improvement per session for first 5 sessions |
| Manager adoption of training tab | ≥ 60% of accounts with ≥ 1 apprentice view training tab weekly |
| OEM certification pass rate for apprentices using the mode | ≥ 85% first-attempt pass rate |
