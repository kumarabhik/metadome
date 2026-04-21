# Phase 3 Beta Results Report

**Beta period:** Months 9–14
**Sites:** 3 dealerships (Toyota Austin TX, Toyota Dallas TX, Ford Austin TX)
**Technicians:** 11 total (8 Toyota, 3 Ford)
**Repairs completed using the system:** 73
**Report compiled:** Month 14, prior to Phase 4 GA planning

---

## Executive Summary

Phase 3 beta delivered its primary objective: First-Time Fix Rate on enrolled fault types reached 91.8% across 73 completed repairs, exceeding the ≥90% target one phase early. Mean Time to Diagnose dropped from the 3.4h control group baseline to 2.1h — a 38% improvement, exceeding the 20% target. Zero safety incidents occurred.

Three unexpected findings shaped Phase 4 decisions: (1) service manager adoption of the dashboard was lower than expected, requiring a redesign of the dashboard entry point; (2) voice command failures were concentrated in one physical bay with high ambient noise from a shared compressor, informing the WiFi6E AP placement standard; (3) the confidence score survey revealed that junior technicians gained confidence significantly faster than senior technicians, pointing to a training-mode opportunity that became the Apprentice Mode feature in Phase 5.

---

## Quantitative Results

### First-Time Fix Rate (FTFR)

| Fault Type | Control Group FTFR | Beta FTFR | Delta | Repairs (n) |
|---|---|---|---|---|
| EV battery coolant leak (Toyota bZ4X) | 64% | 93% | +29pp | 18 |
| EV battery cell fault (Toyota bZ4X) | 61% | 89% | +28pp | 14 |
| ICE timing chain (Ford Mach-E ICE) | 72% | 94% | +22pp | 11 |
| Hybrid inverter fault (Toyota bZ4X) | 58% | 90% | +32pp | 9 |
| Brake system leak (Toyota + Ford) | 81% | 95% | +14pp | 21 |
| **All enrolled fault types** | **67%** | **91.8%** | **+24.8pp** | **73** |

Statistical significance: p < 0.001 for all fault types individually; aggregate p < 0.0001. Control group n = 71 (matched repairs at same sites on same fault types using tablet-only workflow).

### Mean Time to Diagnose (MTTD)

| Fault Type | Control (h) | Beta (h) | Improvement |
|---|---|---|---|
| EV battery coolant leak | 3.8 | 2.2 | 42% |
| EV battery cell fault | 4.1 | 2.4 | 41% |
| ICE timing chain | 2.9 | 1.8 | 38% |
| Hybrid inverter fault | 4.4 | 2.7 | 39% |
| Brake system leak | 2.1 | 1.4 | 33% |
| **Overall average** | **3.4h** | **2.1h** | **38%** |

### Voice Command Recognition Rate

| Site | Recognition Rate | Notes |
|---|---|---|
| Toyota Austin — Bay 1 | 98.2% | Standard noise environment |
| Toyota Austin — Bay 2 | 96.1% | Adjacent to compressor — see Finding 2 |
| Toyota Dallas — Bay 1 | 97.8% | Standard |
| Toyota Dallas — Bay 2 | 98.4% | Standard |
| Ford Austin — Bay 1 | 97.3% | Standard |
| **Overall** | **97.6%** | Meets ≥97% target |

### Technician Confidence Score (Post-Repair Survey)

Average composite score across all sessions: **4.1 / 5.0** (target: ≥ 3.8)

By technician cohort:
- Senior techs (> 8 years): 3.7 average (below target — see Finding 3)
- Mid-level techs (3–8 years): 4.2 average
- Junior techs (< 3 years): 4.6 average

### Safety

- HV-STOP events triggered: 4
- All 4 triggered correctly (verified by audit log review — proximate cause was tech hand position during inspection, not system error)
- All 4 acknowledged and sessions resumed correctly
- Zero safety incidents (injuries, near-misses, equipment damage)

---

## Qualitative Findings

### Finding 1: Dashboard adoption by service managers was 45%, below the 70% DAU target

**Observed:** At mid-beta review (Month 11), only 5 of 11 managers had logged into the dashboard in the prior week. Three had never logged in after initial onboarding.

**Root cause interviews revealed:** The dashboard required managers to navigate to a URL and log in separately. Managers with existing DMS workflows (CDK, Reynolds) resented adding another login. The data was valuable but the access friction was too high.

**Phase 4 impact:** Redesign dashboard entry point — add a "Daily digest" email sent each morning with key metrics inline (no login required for the summary). Deep-link from email to specific dashboard views with pre-authenticated token. Result: dashboard engagement increased to 81% within 30 days of email digest launch (piloted with Toyota Dallas in Month 13).

---

### Finding 2: Voice command failures concentrated in one bay due to shared compressor

**Observed:** Toyota Austin Bay 2 had 96.1% recognition rate vs. 97–98% at other bays. Investigation: Bay 2 shares a wall with the compressor room; intermittent 90–95 dB background noise during compressor cycles.

**Root cause:** VoiceNLP training data included general bay noise but not cyclical compressor bursts at this frequency/amplitude combination. The model was not robust to the sharp onset of compressor noise mid-command.

**Phase 4 impact:** Two changes: (1) Add compressor-burst noise augmentation to VoiceNLP training data; (2) Update WiFi6E AP placement standard to include a "noise audit" step — bays adjacent to compressor rooms require acoustic dampening or a dedicated AP to avoid interference. VoiceNLP retrained with augmented data achieved 98.0% at the same bay in Phase 4.

---

### Finding 3: Senior technicians showed lower confidence scores than expected

**Observed:** Techs with > 8 years experience scored 3.7 average on the confidence survey, below the 3.8 target. Exit interviews revealed: senior techs felt the step-by-step overlay "slowed them down" on fault types they already knew well. One master tech (T01, Toyota): *"I know this repair. The system is showing me things I don't need to see."*

This is not a product failure — it's a user segmentation insight. The product was optimized for the average tech, which inadvertently created friction for expert techs.

**Phase 4 impact:** Expert Mode designed and built — after 50 completed sessions on a fault type, techs are offered condensed step cards (single-line instructions, no WHY/WATCH FOR text). Senior techs at Toyota Austin opted into Expert Mode for coolant leak repairs; confidence scores rose to 4.2 in Month 13.

---

### Finding 4: Junior technicians are the highest-value user segment

**Observed:** Junior techs (T02, T07, T09) showed the largest FTFR improvement (+31pp average), the highest confidence score gain over the beta period, and the most enthusiastic NPS responses (all 9–10). T07 (Ford, 1 year experience): *"I went from being scared to do EV work to being confident in three months. I genuinely feel like a different tech."*

This was the highest-signal finding from Phase 3. The product creates the most value for the technicians with the fewest years — exactly the cohort that is growing fastest as dealerships hire for EV-capable staff.

**Phase 4 and 5 impact:** (1) Apprentice Training Mode (Phase 5) directly serves this cohort; (2) GTM messaging updated to lead with the junior tech value story when talking to service managers who are managing a talent gap; (3) OEM certification integration (Toyota T-TEN, Ford FACT) becomes a primary differentiator for Phase 5 enterprise deals.

---

### Finding 5: Job card integration was the feature service advisors cared about most

**Observed:** T04 (service advisor, Toyota Austin) and the service advisor at Ford Austin both identified the job card auto-population as the Phase 3 feature they used and valued most — even over FTFR improvements. T04: *"The system writes the repair notes for me. I used to spend 15 minutes per job card. Now it's automatic."*

This was not anticipated at the level it was observed. Service advisors are not in our primary persona but they are influential in dealer adoption decisions.

**Phase 4 impact:** Job card integration elevated from a "nice-to-have" to a launch feature. CDK and Reynolds & Reynolds integrations both prioritized for Phase 4 GA. Service advisor added as a secondary persona in the GTM messaging.

---

## Phase 4 Decisions Driven by Beta

| Decision | Source Finding |
|---|---|
| Dashboard daily digest email | Finding 1 |
| VoiceNLP compressor-burst noise augmentation | Finding 2 |
| Expert Mode (condensed steps for experienced techs) | Finding 3 |
| Apprentice Training Mode (Phase 5 prioritization) | Finding 4 |
| Job card integration as Phase 4 launch feature | Finding 5 |
| Junior tech as lead GTM persona | Finding 4 |
| AP placement noise audit in install standard | Finding 2 |

---

## Phase 3 Exit Criteria Assessment

| Criterion | Target | Actual | Pass? |
|---|---|---|---|
| 50+ completed repairs | 50 | 73 | ✓ |
| FTFR data (min 50 repairs) | 50 | 73 | ✓ |
| MTTD improvement ≥ 20% over control | ≥ 20% | 38% | ✓ |
| Tech confidence score ≥ 3.8/5.0 | 3.8 | 4.1 | ✓ |
| Zero safety incidents | 0 | 0 | ✓ |
| Latency SLA < 40ms p95 | < 40ms | 31ms p95 | ✓ |

**Phase 3 exit criteria: all passed. Phase 4 GA approved.**
