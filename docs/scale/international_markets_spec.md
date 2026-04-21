# International Markets Launch — Canada & UK

**Phase:** 5
**Roadmap Item:** Scale: Launch in 2 international markets (Canada, UK) — regulatory compliance adapted

---

## Objective

Expand X-Ray Vision Diagnostics beyond the US market into Canada and the United Kingdom. Both markets were selected as launch-1 targets because they share English as the primary language, have established franchised dealer networks with OEM relationships already in place from US partnerships, and have regulatory environments that map closely (but not identically) to US frameworks already addressed in Phases 1–4.

Target: Both markets live and billing within Months 27–30.

---

## Canada Market Spec

### Why Canada First
- Same OEM dealer networks (Toyota Canada, Ford Canada, Stellantis Canada) — legal entities separate from US but relationship already established through Phase 1 OEM partnerships
- English-primary with French-secondary requirement (Quebec)
- Regulatory environment: Canada Labour Code Part II governs HV safety; maps closely to OSHA 1910.147 with minor delta
- No FCC recertification needed — IC (Innovation, Science and Economic Development Canada) certification required for HoloLens 2 radio; Microsoft already holds IC certification

### Compliance Adaptations

| Requirement | US Standard | Canada Adaptation |
|---|---|---|
| HV lockout/tagout | OSHA 1910.147 | Canada Labour Code Part II, Section 125(1)(x) — functionally identical; update procedure header |
| EV battery safety | SAE J1742 | Transport Canada Motor Vehicle Safety Act — SAE J1742 recognized as equivalent; no re-certification required |
| Privacy / data | State-level (varies) | PIPEDA (federal) + Quebec Law 25 — requires explicit consent banner, data residency in Canada for Quebec deployments |
| Headset radio | FCC Part 15 | IC RSS-247 — Microsoft HoloLens 2 already IC-certified |
| Language | English | Bilingual (English + French) required for Quebec deployments only |

### French Localization (Quebec)
- Scope: UI labels, voice wake word, onboarding materials, safety warnings
- French voice model: leverage VoiceNLP bilingual architecture from Phase 4 Spanish expansion — same pattern, different language pair
- Wake word for Quebec deployments: "Allez Aria" (French equivalent of "Hey Aria")
- OEM vocabulary localization: Canadian French automotive terminology review with Toyota Canada and Ford Canada technical training teams
- Timeline: 6-week localization sprint, parallel to infrastructure work

### Data Residency
- Canadian customer data must remain in Canada — AWS ca-central-1 (Montreal) region
- Edge server telemetry: local buffering, batch upload to ca-central-1 only
- HV-STOP audit logs: stored in ca-central-1, 7-year retention per Canada Labour Code
- No cross-border data transfer to US servers for PII or repair event data

### Dealer Network Entry
- Launch partners: Toyota Canada dealer group (Ontario — 3 bays), Ford Canada dealer group (BC — 2 bays)
- OEM-assisted sales motion: Toyota Canada and Ford Canada regional training teams co-sell as part of technician upskilling programs
- Pricing: CAD-denominated subscription at 1:1 USD parity (no currency discount) — revisit at Month 30 if CAD weakens >8%

---

## UK Market Spec

### Why UK Second (Same Wave as Canada)
- English-primary — no localization delta beyond UK automotive vocabulary ("bonnet" not "hood", "tyre" not "tire", etc.)
- Toyota GB, Ford UK, Stellantis UK all have existing OEM relationships
- Post-Brexit regulatory environment: UK-specific standards have diverged from EU; UK PSSR 2000 governs HV safety in workplace

### Compliance Adaptations

| Requirement | US Standard | UK Adaptation |
|---|---|---|
| HV lockout/tagout | OSHA 1910.147 | UK Provision and Use of Work Equipment Regulations 1998 (PUWER) + PSSR 2000 for pressurised systems; HV-STOP procedure requires PSSR-compliant written scheme of examination |
| EV battery safety | SAE J1742 | UK adopts UN ECE R100 for EV battery safety — SafetyGuard proximity thresholds and HV-STOP activation unchanged; documentation updated to cite R100 |
| Privacy / data | State-level (varies) | UK GDPR (post-Brexit retained law) — same rights as EU GDPR; lawful basis for processing = legitimate interests (repair safety); DPA 2018 registration required |
| Headset radio | FCC Part 15 | UK Conformity Assessed (UKCA) marking required post-Brexit; Microsoft holds UKCA for HoloLens 2 |
| Language | English | UK English vocabulary review; no new voice model needed |

### UK Vocabulary Localization
- Automotive vocabulary delta: ~40 terms (bonnet, boot, tyre, windscreen, gearbox, gear lever, handbrake, etc.)
- Approach: VoiceNLP entity extraction vocabulary file — add UK-EN variant alongside US-EN; intent classification unchanged
- OEM vocabulary source: Toyota GB and Ford UK technical glossaries

### Data Residency
- UK GDPR requires data not leave UK without adequate safeguards — use AWS eu-west-2 (London) region
- Separate data silo from US and Canada — no cross-region replication of PII
- HV-STOP audit logs: London region, 3-year minimum retention per PUWER

### Dealer Network Entry
- Launch partners: Toyota GB dealer group (South East England — 3 bays), Ford UK dealer group (Midlands — 2 bays)
- Right-hand drive vehicles: ArUco marker placement specs must be mirrored for RHD — update `docs/prototype/aruco_anchoring_spec.md` with RHD variants for each enrolled vehicle model
- Pricing: GBP-denominated, equivalent to USD Professional tier

---

## Shared Go-To-Market

### Sales Motion
Both markets enter via OEM-assisted channel (same model as GA Launch):
1. OEM national training team introduces X-Ray Vision Diagnostics at regional dealer meeting
2. Dealer principal receives ROI calculator output in local currency
3. 30-day pilot (2 bays) → expand to full site on positive FTFR data

### Rollout Sequence
| Month | Milestone |
|---|---|
| 24 | Legal entity registered in Canada (required for PIPEDA compliance officer); UK ICO registration complete |
| 24 | AWS ca-central-1 and eu-west-2 infrastructure provisioned; data residency tested |
| 25 | French localization complete and QA'd; RHD ArUco specs updated |
| 25 | Canada pilot sites installed (Ontario Toyota group, BC Ford group) |
| 26 | UK pilot sites installed (SE England Toyota, Midlands Ford) |
| 27 | Pilot data reviewed — FTFR delta, voice recognition rate, technician confidence score |
| 28 | Full commercial launch both markets; OEM-assisted sales motion activated |
| 30 | 20 bays live across Canada + UK combined |

### Success Criteria
- 20 bays live and billing in Canada + UK by Month 30
- FTFR improvement ≥ 20% over local tablet control group within 60 days of site activation
- Voice Command Recognition Rate ≥ 97% (English) / ≥ 95% (French Quebec) within 30 days post-launch
- Zero HV-STOP failures in either market during first 500 repairs
- No data residency violations (confirmed by quarterly compliance audit)

---

## Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| UK UKCA re-certification delay for HoloLens 2 accessories | Low — MS holds UKCA | Verify with Microsoft UK procurement team Month 22; buffer 8 weeks |
| Quebec French voice model quality below 95% target | Medium | Begin French voice data collection Month 22 alongside Spanish lessons learned from Phase 4 |
| RHD ArUco marker placement requires new vehicle-specific calibration | Medium | Toyota GB and Ford UK service training teams provide bay layout drawings Month 23 |
| CAD data licensing for UK/Canada not covered by existing OEM agreements | Medium | Phase 1 OEM agreements should include global rights; legal to confirm scope Month 22 |
| Currency risk (GBP/CAD vs USD) erodes margin | Low (short term) | Annual contracts lock rate at signing; review pricing annually |
