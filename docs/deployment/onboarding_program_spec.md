# Technician Onboarding Program Spec
## Step 30 — Phase 3, Beta Deployment 2: 3-Hour Orientation + Peer Champion Model

**Roadmap reference:** Phase 3, Beta Deployment 2
**Status:** `[ ]` Not started
**Owner:** PM + UX Research + Training
**Target:** Onboarding complete for all beta technicians before Month 9 live repairs begin

---

## Objective

Prepare service technicians at 3 beta dealerships to use X-Ray Vision Diagnostics independently within a single 3-hour session. Onboarding must be fast (technicians lose a half-day of flat-rate pay), practical (hands-on > lecture), and sticky (techs should remember the system a week later without re-training).

**Key constraint:** Technicians are paid flat-rate. Every hour in training is real money lost. Onboarding must respect this — no filler, no marketing, no corporate slideshows.

---

## Trainee Profile

Based on user research (`docs/ux/interview_protocol.md`):
- 8–15 years dealership experience
- Comfortable with OBD-II scan tools; skeptical of new tech until proven
- Time-pressured; values tangible time savings
- Hands-on learner; doesn't read manuals

**Onboarding success definition:** Technician completes the full coolant diagnosis flow with zero coach assistance within 90 minutes of session end.

---

## Cohort Structure

| Site | Technicians | Sessions | Format |
|---|---|---|---|
| Toyota of Bellevue | 4 technicians | 2 sessions × 2 techs | Pairs — tech coaches each other |
| Toyota of Mission Viejo | 4 technicians | 2 sessions × 2 techs | Same |
| Ford of Kirkland | 3 technicians | 1 session × 2 + 1 solo | Same where possible |

**Peer champion:** Each site has a designated "peer champion" — the most enthusiastic early adopter. Champion attends Session 1, then co-coaches Session 2 alongside the trainer. After onboarding, champion is the first-line support contact for colleagues.

---

## Session Agenda (3 Hours)

### Module 1: Why This Exists (15 minutes)

**Format:** Conversation, not slides. Trainer asks questions; tech answers.

Questions:
- "What's the hardest EV fault you've diagnosed in the last 6 months?"
- "How long did it take? What did you wish you had?"
- "What does it cost you professionally when a car comes back for the same issue?"

Purpose: Establish emotional buy-in by connecting the product to the tech's own pain before showing any technology.

**Trainer closes with one data point:** "Dealerships running this system in pilot target going from 67% first-time fix rate on EV faults to 90%+. That's fewer comebacks, fewer angry customers, more flat-rate hours that stick."

---

### Module 2: Headset Basics (20 minutes)

**Format:** Hands-on. Trainer demonstrates once; tech does it themselves.

Topics:
1. Donning HoloLens 2: adjust fit, adjust IPD, focus wheel
2. Spatial calibration: walk around the bay, let SLAM anchor establish (tech sees "anchor confirmed" badge)
3. Wake word: say "Hey Vision" — hear tone confirmation
4. Voice input: say any 3 voice commands from the prompt card (laminated, kept in bay)
5. Gaze-dwell selection: look at menu item for 1.5 seconds to select
6. Head gesture: two nods = confirm; head shake = cancel

**Hands-on check:** Tech must don headset, initialize anchor, and activate one CAD layer before Module 3 begins.

---

### Module 3: Guided Diagnosis — Coached Run (60 minutes)

**Format:** Tech runs a full coolant diagnosis on a bZ4X (or Mach-E) in the bay. Trainer coaches verbally — does not touch the headset.

Steps (per `docs/features/coolant_diagnosis_flow.md`):
1. Say wake word
2. Say "Scan this vehicle"
3. Confirm VIN read
4. Hear fault summary (DTC list + DiagnosticCore assessment)
5. Say "Show me the coolant system"
6. Navigate 7 guided repair steps (voice navigation: "next step", "back", "explain this")
7. HV-STOP simulation: trainer announces "HV proximity" — tech identifies the alarm and responds correctly
8. Say "Complete repair"
9. Complete confidence survey (5 questions, voice)

**Trainer coaching rules:**
- Correct direction only; no "watch me" takeovers
- If tech is stuck for > 30 seconds → give one hint, then wait
- Log: number of hints per tech; target ≤ 2 hints for full run

**Common issues and how to handle:**
| Issue | Coach Response |
|---|---|
| Wake word not recognized | "Try speaking directly into the mic; pause before the word" |
| Voice command not understood | "Say it more slowly; remember the system uses specific phrases — see the prompt card" |
| Anchor drift (CAD drifts off vehicle) | "Step back 1m; let the headset re-anchor; wait for the badge" |
| Tech gets frustrated | "Take the headset off for 2 minutes. Let's talk about what felt off." |

---

### Module 4: Solo Run (30 minutes)

**Format:** Same diagnosis flow, no coaching. Trainer observes and takes notes only.

**Pass criteria:**
- Tech completes full flow (Steps 1–9) with zero coaching interventions
- Time: ≤ 90 minutes from donning to repair complete
- HV-STOP: tech responds correctly (identifies alarm, acknowledges correctly)
- Survey: tech completes survey without assistance

If tech does not pass: schedule a 30-minute follow-up session within 48 hours.

---

### Module 5: Reference Materials + Q&A (15 minutes)

**Materials left at site:**
- 2× laminated voice command prompt cards (posted in bay)
- 1× headset quick-start card (attached to charging station)
- Peer champion contact card (name + cell number)
- QR code → internal help portal (planned for Phase 4)

**Q&A format:** Open; no prepared slides. Trainer takes written notes on all questions — feeds to PM for UX iteration backlog.

---

## Peer Champion Program

The peer champion's role after onboarding:
1. Answer colleague questions before contacting the vendor support line
2. Flag system issues in the beta Slack channel (shared with engineering team)
3. Attend bi-weekly 30-minute PM check-in call
4. Complete weekly "champion report": one paragraph on what's working, what's frustrating

**Incentive:** Champion receives early access to Phase 4 features and is credited in the case study published for GA launch.

---

## Training Materials

| Material | Format | Owner |
|---|---|---|
| Laminated voice command card | Physical; A5 size | PM |
| Headset quick-start card | Physical; A6 size | Engineering |
| Trainer guide | PDF (internal only) | PM + UX |
| Peer champion onboarding deck | PDF (3 slides max) | PM |
| Beta feedback Slack channel | Slack | PM |

---

## Metrics

| Metric | Target |
|---|---|
| % of techs passing solo run on first attempt | ≥ 80% |
| Median hints needed in coached run | ≤ 2 |
| Post-training confidence score (Q1 after first live repair) | ≥ 3.5/5.0 |
| Tech drop-off (stopped using after onboarding) | < 10% at 2 weeks |

---

## Exit Criteria

- [ ] All beta technicians (11 total) have completed Module 4 (solo run)
- [ ] All 3 peer champions identified and briefed
- [ ] Voice command prompt cards posted in all 3 bays
- [ ] Training feedback collected from all techs; top 3 issues logged to PM backlog
- [ ] No tech begins live vehicle repairs until solo run is passed
