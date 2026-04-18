# Regulatory Compliance Matrix
## Step 2 — OSHA/SAE J1742 Compliance Framework

**Roadmap reference:** Phase 1, Deliverable 2
**Status:** `[x]` Framework complete
**Owner:** Compliance Lead + Outside Counsel (recommend Bowman and Brooke LLP — automotive product liability specialists)
**Target:** Framework signed off before Phase 3 beta deployment

---

## Objective

Map every relevant regulation to a specific product feature and identify gaps. Any feature that creates a gap must be resolved before the relevant phase ships. Safety is not a Phase 3 problem — it is designed in from Phase 1.

---

## Regulatory Scope

| Regulation | Jurisdiction | Domain | Relevance |
|---|---|---|---|
| OSHA 29 CFR 1910.147 | USA | Lockout/Tagout (LOTO) | HV-STOP protocol must satisfy LOTO logging and procedural requirements |
| OSHA 29 CFR 1910.303 | USA | Electrical safety | HV zone demarcation and approach distance requirements |
| SAE J1742 | International | EV safety — HV systems | Defines approach limits, PPE requirements, and warning system specs for HV work |
| NFPA 70E (2024 edition) | USA | Electrical safety in workplace | Arc flash and shock hazard boundary definitions; affects HV-STOP trigger distances |
| ISO 45001:2018 | International | Occupational health & safety | OHS management system standard; affects how we document and audit incidents |
| FCC Part 15 | USA | RF emissions (headset radio) | Headset must meet FCC Part 15 Class B for use in commercial environments |
| FDA (21 CFR Part 892) | USA | Medical device (NOT applicable) | Confirmed not a medical device; no patient data, no clinical diagnosis |
| GDPR / CCPA | EU / California | Data privacy | Technician biometric data (eye-tracking, voice) classified as personal data; requires consent + deletion rights |

---

## Gap Analysis by Feature

### HV-STOP Safety Guardrail

| Requirement | Source | Our Implementation | Gap? |
|---|---|---|---|
| Approach boundary warning at ≥300mm | SAE J1742 §6.3 | 300mm amber warning, 150mm hard stop | None — exceeds requirement |
| Audible warning for HV approach | SAE J1742 §6.4 | Audio alarm on amber + red trigger | None |
| Written acknowledgment before HV work | OSHA 1910.147(c)(4) | Verbal "HV aware" + nod confirmation logged | **Gap:** OSHA prefers *written* sign-off. Mitigation: voice confirmation logged with technician ID + timestamp satisfies spirit; recommend outside counsel confirm |
| LOTO documentation | OSHA 1910.147(c)(4)(i) | HV-STOP log with tech ID, VIN, timestamp, zone | None — log format must be reviewed by compliance counsel pre-launch |
| PPE verification | NFPA 70E Table 130.5(G) | System asks "Confirm insulated gloves" before HV zone entry | **Gap:** We cannot physically verify PPE. Mitigation: display PPE checklist; log tech acknowledgment. Disclaimer required in EULA that PPE verification is technician's responsibility |

### Spatial Overlay (General)

| Requirement | Source | Our Implementation | Gap? |
|---|---|---|---|
| Overlay must not obscure safety labels | OSHA 1910.145 | Layer budget rule: safety labels in physical world always visible through overlay | Design rule only — needs QA validation in test bay |
| Overlay must not cause distraction during ignition/movement | Internal + SAE best practice | Session ends / dims when vehicle in motion (CAN bus speed signal) | None |

### Data Privacy

| Requirement | Source | Our Implementation | Gap? |
|---|---|---|---|
| Voice data (biometric) consent | CCPA §1798.100 | Consent screen at first use; voice processed locally on edge | **Gap:** Voice data sent to cloud for NLP model improvement must be opt-in; default = local only |
| Eye-tracking data (biometric) | GDPR Art. 9 (EU expansion) | Eye-tracking data never leaves headset; used for dwell selection only | None for US; EU deployment requires formal DPA with each dealer |
| Technician repair history | CCPA | Stored on edge server per bay; accessible by dealer admin only | **Gap:** Define data retention policy; recommend 90-day rolling window with dealer option to extend |
| Right to deletion | CCPA §1798.105 | Not implemented | **Gap:** Must build tech ID deletion workflow before GA |

---

## Open Items for Outside Counsel

1. Does voice confirmation of "HV aware" satisfy OSHA 1910.147(c)(4) in lieu of written sign-off?
2. What is the exposure if a technician bypasses the HV-STOP at the manager level and is injured?
3. Does the system constitute a "safety device" under OSHA standards, creating additional certification obligations?
4. What product liability shield exists if a technician follows system guidance that is incorrect?
5. For EU launch: does eye-tracking dwell constitute biometric data processing requiring explicit consent under GDPR Art. 9?

---

## Compliance Calendar

| Milestone | Target Date | Dependency |
|---|---|---|
| Outside counsel retained | Month 1 | Budget approval |
| LOTO log format reviewed and approved | Month 3 | Counsel + HV-STOP spec |
| SAE J1742 self-assessment complete | Month 4 | Engineering + Compliance |
| SAE J1742 review panel submission | Phase 3 (Month 9) | External |
| OSHA audit readiness review | Phase 3 (Month 13) | Beta deployment |
| SOC 2 Type II audit | Phase 5 (Month 24) | Cloud platform |
