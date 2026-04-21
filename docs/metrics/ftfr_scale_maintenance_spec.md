# FTFR ≥90% Maintained at Scale

**Phase:** 5
**Roadmap Item:** Metrics: ≥90% FTFR maintained at scale across all enrolled fault types

---

## Objective

Demonstrate that the First-Time Fix Rate (FTFR) improvement — ≥90% on enrolled fault types — is not a Phase 4 artifact of a carefully managed 50-bay beta, but a durable result that holds as the platform scales to 500 bays across 50+ dealerships with wider tech skill variance, more fault types, more vehicle models, and international markets.

This is the platform's core value claim. If FTFR degrades at scale, every downstream metric (pricing power, OEM embedded deals, case studies) weakens.

---

## Background: Phase 4 Baseline

Phase 4 exit criterion: FTFR ≥ 90% demonstrated on enrolled fault types, n ≥ 200 repairs.

Enrolled fault types at Phase 4 exit:
1. EV battery coolant leak (Toyota bZ4X)
2. EV battery cell fault (Toyota bZ4X)
3. ICE timing chain (Ford Mustang Mach-E — ICE variant)
4. Hybrid inverter fault (Toyota bZ4X hybrid)
5. Brake system leak (Toyota bZ4X, Ford Mach-E)

Phase 4 FTFR results (target): ≥ 90% on each fault type, aggregate ≥ 90%.

Phase 5 adds: new fault types, new vehicle models (Phase 5 expands to 5 OEM lines), international markets, and 10× the bay count. The question is: does the ≥90% hold?

---

## Definition (Consistent with Phase 4)

**FTFR** = percentage of repairs where the fault is correctly identified and resolved in a single visit, with no return visit for the same fault within 30 days.

Numerator: completed repairs where `return_visit_same_fault_30d = false`
Denominator: all completed repairs on enrolled fault types

Enrolled fault types only — repairs on non-enrolled fault types (no CAD overlay available) are excluded from the denominator and tracked separately as "unassisted repairs."

---

## Measurement at Scale

### Data Pipeline
Same as Phase 4 (see `docs/metrics/ftfr_90_achievement_spec.md`) with one addition: data must now flow from 500 edge servers across 4 AWS regions (US, ca-central-1, eu-west-2, us-west-2) into a unified analytics view.

De-identified repair events from all regions → global analytics S3 bucket (us-east-1) → Athena query → dashboard.

PII (tech ID, dealer name) stripped before cross-region transfer. VIN retained for return-visit matching — de-identified at query layer before dashboard display.

### Segmentation at Scale

At 500 bays, aggregate FTFR is insufficient — it can mask regional degradation or fault-type regression. Weekly monitoring requires segmentation by:

| Dimension | Why |
|---|---|
| Fault type | Regression on one fault type must be caught before it pulls aggregate below threshold |
| Vehicle model | A new model integration may have lower FTFR initially |
| Technician cohort (expert / mid / apprentice) | Apprentice Mode users may drag FTFR if not properly trained |
| Dealer tier (Core / Professional / Enterprise) | Core tier has fewer features — expect lower FTFR; document delta |
| Market (US / Canada / UK) | International market launch may show lower FTFR due to tech training ramp |
| Bay vintage (months since install) | FTFR typically improves over first 90 days as techs become proficient |

### Alerting Thresholds

| Alert Level | Trigger | Action |
|---|---|---|
| Warning | FTFR drops below 87% on any fault type × market segment (rolling 4-week window) | CS team contacts dealer; Customer Success review within 48 hours |
| Critical | FTFR drops below 83% on any fault type × market segment | Engineering + Product incident review within 24 hours; root cause analysis |
| Platform-wide | Aggregate FTFR drops below 88% (rolling 4-week) | Executive escalation; incident declared |

Warning threshold is 87% not 90% to give room to investigate before the contractual 90% threshold is at risk.

---

## FTFR Degradation Response Playbook

When FTFR drops below warning threshold:

### Step 1: Classify Root Cause
Pull repair records for the affected segment. Review:
- DiagnosticCore outputs: were DTC interpretation and layer manifest correct?
- Sensor readings: were SensorFusion readings within expected range?
- Tech behavior: did tech follow all steps (step completion log) or skip steps?
- Return visit record: what fault code appeared on return visit — same DTC (tech missed root cause) or different DTC (unrelated)?

Root cause categories:
1. **Model error** — DiagnosticCore gave wrong repair recommendation (RAG corpus gap or hallucination)
2. **Sensor error** — SensorFusion misread sensor (hardware fault, calibration drift)
3. **UX error** — tech skipped steps or misread the overlay (training gap)
4. **Out-of-scope fault** — fault was enrolled fault type but vehicle had a secondary fault not captured by current CAD layers

### Step 2: Targeted Remediation

| Root Cause | Remediation |
|---|---|
| Model error | Update RAG corpus with corrected OEM bulletin; trigger DiagnosticCore re-indexing; RLHF feedback loop (Phase 4 feature) |
| Sensor error | Dispatch hardware check to affected bays; replace sensor or recalibrate |
| UX error | Targeted retraining: manager assigns Apprentice Mode for that fault type for affected techs |
| Out-of-scope fault | Expand CAD layers for that fault type; add to roadmap sprint |

### Step 3: Validate Recovery
Re-measure FTFR on affected segment 4 weeks post-remediation. If still below 87%: escalate to product leadership for deeper investigation.

---

## New Fault Type and Vehicle Model Ramp Period

When a new fault type or vehicle model is added to the enrolled list, FTFR is expected to be lower initially (tech learning curve + potential model gaps). Standard ramp:

- **Weeks 1–4 post-launch:** Track FTFR but do not include in the platform-wide aggregate — reported separately as "ramp period"
- **Week 5+:** Included in aggregate. If below 87% at Week 5 → trigger playbook immediately (rather than waiting 4-week rolling window)

This prevents new model launches from masking underlying degradation in mature fault types.

---

## Continuous Improvement Mechanisms

### RLHF Integration
Phase 4 built the RLHF pipeline — technician feedback signals flow back to DiagnosticCore to improve confidence scoring. At scale, this becomes the primary quality flywheel:
- More repairs → more feedback signal → better confidence scoring → higher FTFR
- Expected: FTFR improvement of 1–2 percentage points per 6-month RLHF training cycle

### Voice Recognition Rate Monitoring
At Phase 3 target (≥97% voice command recognition), misheard commands are a minor FTFR risk. At scale, if recognition rate drops below 95% in a market or language variant, tech errors increase and FTFR drops. Voice recognition rate is a leading indicator of FTFR risk — monitored alongside FTFR.

---

## Reporting Cadence

| Audience | Frequency | Content |
|---|---|---|
| Engineering + Product | Weekly | Segment-level FTFR table; active warnings; root cause in progress |
| CS + Sales | Monthly | Dealer-level FTFR (for account review) |
| OEM Partners | Quarterly | Aggregate FTFR by fault type and vehicle model (their vehicles only) |
| Exec + Board | Monthly | Platform-wide FTFR; trend; target vs. actuals |
| Public (case studies, marketing) | As available | Named dealer case study FTFR data (dealer approval required) |

---

## Phase 5 Exit Criterion

FTFR ≥ 90% maintained at scale is confirmed when:
- Platform-wide aggregate FTFR ≥ 90% (rolling 12-week window, n ≥ 500 repairs) **and**
- No fault type × market segment persistently below 87% for more than 8 consecutive weeks **and**
- FTFR data covers ≥ 3 OEM vehicle lines and ≥ 200 bays

Persistently below means: warning-level FTFR for ≥ 8 weeks without remediation returning it to ≥ 87%.
