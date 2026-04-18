# Tri-Modal Hands-Free Validation Protocol
## Step 8 — Wizard of Oz Simulation Design

**Roadmap reference:** Phase 1, Deliverable 8
**Status:** `[x]` Protocol complete
**Owner:** UX Researcher + PM
**Target:** 2-day WoZ sessions in Month 3 (after interviews inform protocol refinements)

---

## Objective

Validate the Tri-Modal Hands-Free interaction model (Voice → Eye-Tracking → Head Gesture) with real technicians before writing a single line of rendering code. A Wizard of Oz simulation lets us test the interaction logic using a human operator simulating the AI — catching fatal UX flaws for $0 instead of discovering them after 4 months of engineering.

**Specific questions to answer:**
1. Does the voice-first model work in an 80dB+ bay environment? Which commands break down?
2. Is 800ms eye-dwell the right threshold? Too fast (accidental selections) or too slow (frustrating)?
3. Do technicians discover and use the head-gesture (nod/shake) without explicit instruction?
4. What voice commands do technicians *spontaneously* use — do they match our intent taxonomy?
5. Which interaction mode do technicians prefer for which task type?

---

## Wizard of Oz Setup

### Equipment

| Item | Purpose |
|---|---|
| Participant headset | Cheap video passthrough headset (Oculus Quest 2 or cardboard + phone) showing a pre-rendered fake "overlay" still image — tech doesn't know the overlay is static |
| Facilitator laptop | Runs the "overlay control panel" — WoZ operator clicks buttons to change what the participant sees |
| Wireless earpiece | Participant hears AI audio responses (voice responses pre-recorded by product team) |
| Noise canceling mic (lapel) | Captures participant voice commands |
| GoPro on tripod | Records full session including body posture and hand position |
| Bay replica | Use a donated/scrap vehicle hood with fake wiring to simulate repair context |

### Roles

| Role | Responsibility |
|---|---|
| **Facilitator** | Introduces scenario, gives tasks, debriefs. Does NOT operate WoZ panel. |
| **WoZ Operator** | In separate room or behind partition; listens to participant voice, triggers pre-built overlay responses on a tablet that feeds participant's headset display. Simulates DiagnosticCore + CADRenderer. |
| **Observer** | Takes structured notes on interaction patterns, hesitations, errors. |

---

## Session Design

### Participants: 5 technicians (recruited from interview pool)

Selection: 2 apprentices, 2 journeymen, 1 master tech. Gender mix if possible (affects headset fit comfort).

### Scenario Script

**Briefing (5 minutes):**
> "We're testing a new hands-free diagnostic tool. Imagine the headset you're wearing shows you the inside of the car through the panels. The AI assistant is named Aria. You can talk to it, look at menu items to select them, or nod/shake your head to confirm or cancel. Try to work as naturally as possible — don't worry about figuring out the 'right' way to use it."

**Task 1 (5 minutes):** Wake + basic navigation
- "The vehicle has a fault. Start the diagnostic."
- Observe: does participant say "Hey Aria"? Or just "Start"? Or nothing and looks around?
- WoZ response: activates "fault detected" overlay panel

**Task 2 (10 minutes):** Coolant fault diagnosis
- "You've been told there might be a coolant leak. Find it."
- Participant must: invoke coolant layer, navigate to the fault component, confirm they understand what to do next
- Observe: voice command phrasing, dwell threshold comfort, whether they look for gesture input

**Task 3 (5 minutes):** HV-STOP response
- Facilitator says: "Move your hand toward the orange cable cluster" (pre-positioned fake cable)
- WoZ triggers: red overlay + alarm audio
- Observe: does participant stop immediately? Are they startled? Do they understand what to do to continue?
- After stop gate: tell participant "Aria needs you to confirm you have insulated gloves on." Observe how they confirm.

**Task 4 (5 minutes):** Step navigation under load
- "Walk through the next 3 repair steps."
- Tech must advance steps using only voice or eye-tracking (hands "occupied" — we give them a wrench to hold)
- Observe: does "next step" work naturally? Do they use gaze or voice? Is there hesitation?

**Task 5 (5 minutes):** Error recovery
- WoZ simulates: voice recognition failure twice in a row (no response to command)
- Observe: does participant switch to eye-tracking? Do they panic? What error message helps most?

---

## Metrics to Capture (Per Participant)

| Metric | Method | Target |
|---|---|---|
| Voice command success rate | WoZ operator log: command received → correct intent triggered | ≥80% without training |
| Eye-dwell threshold comfort | Post-task survey Q: "Did selections happen at the right time?" 1–5 scale | ≥3.5/5.0 |
| Time-to-first-action (Task 1) | Stopwatch from "Start the diagnostic" instruction to first input | < 15 seconds |
| Head gesture discovery rate | Observation: did participant nod/shake unprompted in Task 3/4? | Track % who discover it |
| Error recovery time | From WoZ failure trigger to participant successfully re-entering correct command | Target < 20 seconds |
| Preferred modality (self-report) | Post-session: "Which method did you use most? Which did you prefer?" | Voice expected; validate |

---

## Post-Session Debrief (15 minutes)

Run with each participant immediately after session:

1. "What felt natural? What felt weird?"
2. "What did you say to Aria that didn't work the way you expected?"
3. "The gaze-selection (looking at something for a moment) — did it feel right? Too fast? Too slow?"
4. "If you could change one thing about how you controlled the system, what would it be?"
5. Show them the intent taxonomy list: "Here are the commands Aria understands. Anything missing that you wanted to say?"
6. "Would you use this tool at work? What would make you NOT use it?"

---

## Decision Criteria

| Finding | Action |
|---|---|
| Voice success rate < 70% in WoZ | Expand intent taxonomy before VoiceNLP engineering begins; re-test |
| Eye-dwell threshold: >50% find 800ms too slow | Reduce to 600ms; test in next round |
| Eye-dwell threshold: >30% find 800ms too fast (accidental) | Increase to 1000ms or add visual progress ring |
| Head gesture: < 20% discover it unprompted | Remove head gesture from v1; voice + eye only |
| HV-STOP: any participant does not immediately stop | Redesign alert modality (louder, more intrusive) before Safety Guardrail engineering |
| Preferred modality ≠ voice | Reassess primary/secondary hierarchy |

---

## WoZ Overlay Control Panel (Operator Reference)

Pre-built "scenes" the WoZ operator can trigger:

| Scene ID | Visual Description | Audio Response | Trigger Condition |
|---|---|---|---|
| `idle` | Soft ambient glow on vehicle exterior | "Vehicle identified. 3 active DTCs." | Session start |
| `coolant_full` | Blue coolant lines, red M4 pulse, yellow pump | "Coolant fault confirmed at Module 4 inlet." | "show coolant" command |
| `step_01` | Arrow at inlet coupling, step counter "1/7" | "Step 1: Inspect the Module 4 inlet coupling at your 2 o'clock." | "start diagnostic" or "step 1" |
| `step_02` | Arrow at hose clamp, step counter "2/7" | "Step 2: Check the hose clamp for corrosion." | "next step" |
| `hv_stop` | Full red overlay, HV bus pulsing | [ALARM TONE] "Stop. High-voltage zone. Confirm insulated gloves before proceeding." | Facilitator hand signal to operator |
| `voice_fail` | Small "?" icon flashes | "I didn't catch that. Try 'next step' or 'show coolant'." | Operator chooses to simulate failure |
| `session_end` | Overlay fades | "Diagnostic session complete. Repair logged." | "close" or "done" command |
