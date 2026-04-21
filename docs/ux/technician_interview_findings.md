# Technician Contextual Interview Findings

**Phase:** 1
**Research method:** Contextual interviews (in-bay, while technician performed actual repair work)
**Sample:** 10 service technicians across 3 dealerships
**Conducted:** Month 2
**Interviewer:** UX Researcher + Product Manager

---

## Study Design

### Goals
1. Understand the current diagnostic workflow end-to-end — where does it break down?
2. Identify the moments of highest cognitive load during repair
3. Understand existing tool behavior and what technicians trust / distrust
4. Probe reaction to the spatial overlay concept (show low-fi sketch, not working prototype)
5. Understand barriers to adoption of new technology in the service bay

### Participant Profile

| # | Dealership | OEM | Years Experience | Role | EV Experience |
|---|---|---|---|---|---|
| T01 | Toyota (Austin TX) | Toyota | 14 years | Master Tech | 2 years (Prius, bZ4X) |
| T02 | Toyota (Austin TX) | Toyota | 3 years | Diagnostic Tech | < 6 months (bZ4X) |
| T03 | Toyota (Austin TX) | Toyota | 8 years | Diagnostic Tech | 1 year (Prius) |
| T04 | Toyota (Dallas TX) | Toyota | 22 years | Service Advisor (observed) | N/A |
| T05 | Toyota (Dallas TX) | Toyota | 6 years | Diagnostic Tech | 1 year |
| T06 | Ford (Austin TX) | Ford | 11 years | Master Tech | 3 years (Mach-E) |
| T07 | Ford (Austin TX) | Ford | 1 year | Lube / Junior Tech | None |
| T08 | Ford (Austin TX) | Ford | 5 years | Diagnostic Tech | 1 year (Mach-E) |
| T09 | Stellantis (Houston TX) | Jeep / Ram | 9 years | Diagnostic Tech | < 6 months (4xe) |
| T10 | Stellantis (Houston TX) | Jeep / Ram | 18 years | Master Tech | None (ICE only) |

---

## Key Findings

### Finding 1: The DTC is a starting point, not an answer — and techs know it

**Observed behavior:** All 10 technicians described a workflow where the DTC code from the scanner is a hypothesis, not a diagnosis. T06 (Ford Master Tech): *"The DTC tells you the system that's complaining. It doesn't tell you what's actually broken. You still have to go find it."*

T01 (Toyota Master Tech) estimated that 40% of his EV diagnoses involve a DTC that points to a downstream symptom rather than the root cause. He spends an average of 45 minutes on these cases before he's confident enough to order parts.

**Implication for product:** DiagnosticCore's RAG pipeline must surface the *root cause pathway*, not just the DTC description. Step cards must guide the tech through the disambiguation — "is this the pump or the controller?" — before committing to a repair.

---

### Finding 2: EV high-voltage anxiety is real, especially for techs under 5 years experience

**Observed behavior:** T02, T07, T09 all exhibited hesitation before connecting to HV systems — double-checking glove integrity, re-reading the procedure on the tablet, asking a senior tech to confirm the vehicle was isolated. T02: *"I don't feel like I know enough yet. I check the procedure three times before I touch anything orange."* (Orange = HV wiring color in automotive convention.)

T10 (18-year ICE master tech): *"I'm not afraid of it, but I've seen guys get complacent. It only takes once."*

**Implication for product:** HV-STOP is not just a safety feature — it's a confidence-building feature for junior techs. The overlay that shows the HV zone boundary in real space (not just a diagram on paper) reduces anxiety and builds competency faster. This is a selling point, not just a compliance requirement.

---

### Finding 3: Looking away from the vehicle to read the tablet is the #1 workflow friction

**Observed behavior:** Timed on 7 technicians: average 4.2 glances at tablet per repair step, averaging 8–12 seconds per glance. Total time eyes-off-vehicle per repair: 6–9 minutes for a 45-minute diagnostic session.

T08: *"The worst part is when the diagram is on page 2 and the procedure is on page 1. You're scrolling back and forth while you're trying to hold a tool in position."*

T03 demonstrated a coping strategy: memorizing the first 3 steps before starting, then returning to the tablet. He called it "loading up before I start touching anything."

**Implication for product:** The core value proposition is confirmed — eyes-on-vehicle is the right anchor. Spatial overlay eliminates the glance-to-tablet loop entirely. The layer selection UX must be designed so the tech never needs to look away mid-step.

---

### Finding 4: Technicians distrust information they can't verify — especially on EVs

**Observed behavior:** T01, T06, T10 (all master techs with 10+ years) expressed strong distrust of AI-generated or unverifiable recommendations. T01: *"If the system tells me to replace a part, I need to know where that came from. Is that Toyota's procedure or did some computer make it up? Because if it's wrong, it's my name on the repair order."*

T06: *"I've seen scanner software tell me to replace a sensor that was perfectly fine. The DTC pointed there but the root cause was a chafed wire 3 feet away. I've learned to verify everything."*

This skepticism is highest among master technicians — not because they are resistant to technology, but because they have been burned by incorrect scanner recommendations before.

**Implication for product:** Every repair step and recommendation must cite its source explicitly — "Per Toyota TSB 0045-23, Section 4.2." This is the "Zero LLM-generated repair instructions" constraint from the Phase 2 exit criterion. It is not just a safety rule; it is what earns trust from the technicians who matter most (master techs influence junior techs and service managers).

---

### Finding 5: Voice control in the shop is viable only if it has wake-word discipline

**Observed behavior:** The service bay is consistently noisy — air compressors, impact wrenches, background radio, multiple conversations. T05 tested a voice command prototype (simple Dragon NaturallySpeaking on a tablet) during the session. It misfired 3 times in 10 commands due to ambient noise.

However: T01 and T06 both noted that technicians naturally develop a "bay voice" — a slightly louder, more deliberate register they use when communicating in a noisy environment. T06: *"You learn to project in here. It's not a library."*

T03 asked: *"Can it tell the difference between me talking to you and me talking to the headset?"* — confirming that the wake-word requirement is intuitive to users, not a friction point.

**Implication for product:** Wake-word model must be trained on bay-voice speech patterns, not standard speech. The "Hey Aria" / "Oye Aria" wake words must be evaluated against compressor noise (85–95 dB background) specifically. This is a VoiceNLP training data requirement.

---

### Finding 6: The service advisor / tech handoff is a pain point that creates rework

**Observed behavior (T04, service advisor interview):** The service advisor translates between what the customer reports and what the tech needs to investigate. T04: *"Half the time the customer says 'the car feels weird' and I have to figure out what to put on the work order. And if the tech finds something the customer didn't mention, I have to go back to the customer to get authorization. That takes 45 minutes minimum."*

T01 observed: sometimes he starts a repair and finds a secondary fault that wasn't on the RO. He has to stop, notify the service advisor, wait for customer approval, and restart. *"I've had jobs where I lost an hour just waiting."*

**Implication for product:** The predictive maintenance mode (Phase 5) addresses this upstream. The job card integration (Phase 3) addresses the downstream handoff. Both features are validated by this finding — they're not nice-to-haves.

---

### Finding 7: Physical ergonomics of wearing a headset for 45+ minutes is a real concern

**Observed behavior:** When shown the HoloLens 2 unit and asked to wear it for 5 minutes while simulating a diagnostic task: T02, T07, T09 all mentioned neck or forehead fatigue. T09: *"I could wear this for a 20-minute job. An hour? I'd have to see."*

T06 (heavier user frame) noted: *"The balance isn't great. It tips forward. You'd need a brow pad that doesn't slide."*

T01 was most positive: *"I've worn worse. The welder's mask is heavier than this."*

**Implication for product:** Ergonomics must be explicitly addressed in the onboarding program — proper fitting procedure, maximum recommended wear time per session (suggest: 90-minute sessions with a 10-minute break; most diagnostic sessions are under 60 minutes so this should be a non-issue in practice). Also: the brow pad replacement cadence in the procurement spec is validated — shared-use without replacing brow pads was flagged by 3 participants as a hygiene concern.

---

### Finding 8: Technicians want to know "how confident is the system" — not just what to do

**Observed behavior:** When shown a mockup of the DiagnosticCore step card, T01 immediately asked: *"Is this the most likely cause or could it be something else? What's the probability this is actually the problem?"*

T06: *"I want to know if this is the 80% answer or the 20% answer. If it's 80%, I'll order the part. If it's 20%, I'm going to investigate more first."*

This is a sophisticated request — they are asking for a confidence score on the diagnostic recommendation, not just a yes/no instruction.

**Implication for product:** DiagnosticCore should surface a confidence tier on the layer manifest card ("HIGH confidence — matches 87% of similar DTC cases on this model"). The Phase 5 RLHF pipeline is the mechanism to generate this confidence score accurately over time. Even a rough confidence indicator ("HIGH / MEDIUM / LOW") would satisfy this need in the short term.

---

## Summary: Top 5 Design Requirements Confirmed by Research

| # | Requirement | Source Finding |
|---|---|---|
| 1 | Eyes-on-vehicle interaction — no tablet glance required during active repair | Finding 3 |
| 2 | Every recommendation cites OEM source (TSB or service manual section) | Finding 4 |
| 3 | HV zone visible in spatial overlay, not just described in text | Finding 2 |
| 4 | Wake-word discipline required; trained on bay-voice + compressor noise | Finding 5 |
| 5 | Confidence indicator on diagnostic recommendations | Finding 8 |

---

## Friction Points Not Currently Addressed in Product (Backlog Candidates)

| Friction Point | Who Raised | Potential Feature |
|---|---|---|
| Service advisor / tech authorization delay | T04, T01 | Predictive alerts shown to service advisor before job card is opened (Phase 5 predictive maintenance) |
| Ergonomic fatigue at 60+ minutes | T02, T07, T09 | Session timer + break reminder; addressed in onboarding program |
| Secondary fault discovery mid-repair causes RO rework | T01, T06 | DiagnosticCore scans for co-occurring faults at session start; surfaces as optional investigation |
| Technician certification credit for completed repairs | T02, T07 | Apprentice Training Mode (Phase 5) — scores feed OEM LMS |

---

## Phase 1 Exit Criterion: Interviews Complete

Phase 1 required: 10 technician interviews completed; key friction points documented.

**Status: Complete.** 10 interviews conducted across 3 dealerships (Toyota Austin, Toyota Dallas, Ford Austin, Stellantis Houston). Key friction points documented above. Design requirements confirmed. Findings briefed to product and engineering leads Month 2, Week 4.
