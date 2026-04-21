# FTFR ≥ 90% Achievement Spec — Phase 4 North Star Milestone
## Step 45 — Phase 4, Metrics 1

**Roadmap reference:** Phase 4, Metrics 1
**Status:** `[ ]` Not started
**Owner:** Product + Data Analytics + Sales
**Target:** FTFR ≥ 90% demonstrated (n ≥ 200 repairs) by Month 20 (Phase 4 exit gate)
**Prerequisite:** FTFR baseline methodology complete (`docs/metrics/ftfr_baseline_spec.md`); Phase 3 baseline data collected

---

## Objective

Demonstrate that X-Ray Vision Diagnostics lifts First-Time Fix Rate (FTFR) to ≥ 90% on enrolled fault types, measured across ≥ 200 completed repairs at Phase 4 commercial sites. This is the product's North Star metric and the primary commercial claim in sales and marketing materials.

**Industry baseline context:**
- EV battery faults (TMS, cell, inverter): 67% FTFR industry average (AutoMD/IHS Markit technician survey, 2023)
- Traditional ICE faults: 78% FTFR industry average
- Target lift: 67% → 90% = 23 percentage point improvement for EV fault types

---

## Definition of First-Time Fix Rate

**FTFR (%) = (Repairs resolved on first visit / Total enrolled repairs) × 100**

### What Counts as "First-Time Fix"

A repair is counted as a first-time fix if:
- The vehicle is presented with a specific DTC or symptom from the enrolled fault type list
- The technician uses X-Ray Vision Diagnostics during the repair session
- The vehicle is not returned for the same fault within 30 days of repair completion
- The repair was marked "complete" by the technician (step completion event in job card integration)

### What Excludes a Repair from FTFR Denominator

| Exclusion Condition | Rationale |
|---|---|
| Vehicle presented without an enrolled DTC | System cannot claim credit for faults outside enrolled scope |
| Technician used headset for < 5 minutes during repair | Insufficient system engagement; repair attributed to technician skill, not tool |
| Vehicle returned within 30 days for unrelated fault | Different fault; not a re-fix event |
| Part not available — repair deferred (not a diagnostic failure) | Parts availability is outside system scope |

### 30-Day Return Window

The 30-day window is the industry standard re-fix window used by CDK Drive and Reynolds & Reynolds DMS systems. Sourced from: NADA operational guidelines for service department KPIs.

---

## Data Collection Architecture

### Primary Data Sources

| Source | What It Provides | Agent / System |
|---|---|---|
| Job card integration | Repair completion timestamp, fault code at presentation, DTC cleared confirmation | Job card integration (`docs/features/job_card_integration_spec.md`) |
| DMS 30-day return flag | Whether same vehicle returned with same or related DTC within 30 days | CDK Drive / Reynolds & Reynolds API query |
| DiagnosticCore session log | Session duration, confidence score at repair completion, which fault flow was used | DiagnosticCore audit log on edge server |
| SensorFusion session log | OBD-II DTC state at session start and end (confirms DTC cleared) | SensorFusion edge log |

### FTFR Calculation Pipeline

1. **Daily batch job** — queries job card data for all completed repairs in enrolled fault types
2. **30-day lookback query** — for each repair completed > 30 days ago, checks DMS for return visit with same/related DTC
3. **FTFR roll-up** — aggregated by: site, technician, fault type, OEM, week, and cumulative
4. **Manager dashboard** — FTFR displayed as primary metric on manager dashboard top-level view (`docs/features/manager_dashboard_spec.md`)

---

## Statistical Significance Requirements

Phase 4 commercial launch requires demonstrating FTFR ≥ 90% at statistical significance before making the claim in public marketing materials.

### Sample Size

| Scenario | Required n | Rationale |
|---|---|---|
| Point estimate (internal dashboard) | 50 repairs | Sufficient for operational monitoring |
| External marketing claim | 200 repairs | 95% confidence interval at ±4pp given σ = 0.3 |
| OEM case study publication | 200 repairs per OEM line | OEM requires per-brand validation |

**Confidence interval formula at n=200:**
- Assumed true FTFR: 90%
- Standard deviation: √(0.9 × 0.1) ≈ 0.30
- 95% CI: ±1.96 × (0.30 / √200) = ±4.2 percentage points
- At n=200, demonstrated FTFR of 90% → reported as "90% ±4pp (95% CI)"

**Minimum threshold for claim:** Demonstrated FTFR must be ≥ 86% at lower bound of 95% CI. If point estimate is 88%, CI lower bound would be 83.8% — below threshold; marketing claim delayed until more data collected.

### Attribution: Separating Tool Effect from Technician Improvement

The control group design from Phase 3 (`docs/metrics/mttd_control_group_spec.md`) provides the attribution framework:

| Variable | Control | Treatment |
|---|---|---|
| Tool | Tablet + OBD-II scanner | X-Ray Vision Diagnostics headset |
| Technicians | Matched by experience tier | Same experience tier distribution |
| Vehicle models | Same OEM lines, same fault types | Same OEM lines, same fault types |
| Time period | Same calendar period | Same calendar period |

If control group FTFR improves concurrently with treatment group, some of the improvement may be attributed to other factors (technician skill development, parts availability improvements, OEM TSB updates). The difference-in-differences estimate isolates the tool effect.

**Reporting standard:** All external claims use the difference-in-differences estimate. "X-Ray Vision Diagnostics improved FTFR by X percentage points vs. tablet-based control group" is more credible than an absolute FTFR number.

---

## FTFR by Fault Type — Phase 4 Targets

| Fault Type | Enrolled Phase | Baseline FTFR | Target FTFR |
|---|---|---|---|
| EV battery coolant leak | Phase 2 | 67% | ≥90% |
| EV battery cell fault | Phase 3 | 62% | ≥88% |
| Hybrid inverter fault | Phase 3 | 70% | ≥90% |
| ICE timing chain | Phase 3 | 78% | ≥92% |
| Brake system leak | Phase 3 | 82% | ≥94% |

**Why fault-type targets differ:** Faults with more complex spatial diagnosis (battery TMS, inverter) benefit most from spatial overlay; simpler faults with known root causes (brake leak) have higher pre-tool baseline, leaving less headroom.

---

## Monitoring & Alerting

### Real-Time Dashboard (Manager View)

- FTFR rolling 30-day trend line per site and per technician
- Alert: if FTFR drops below 85% for any enrolled fault type over a 14-day window → product team notified within 24 hours
- Drill-down: for each re-fix event, DiagnosticCore confidence score at first repair and second repair compared — identifies systematic diagnostic errors

### Monthly Product Review

- Product team reviews FTFR data monthly
- Actions triggered by FTFR decline:
  - < 85%: DiagnosticCore RAG corpus audit — are OEM service bulletins current?
  - < 80%: Engineering deep dive into re-fix cases; SensorFusion data quality review
  - < 75%: Escalate to VP Product; consider pause on new fault type enrollments

### Phase 4 Exit Gate

- n ≥ 200 completed repairs across enrolled fault types
- Demonstrated FTFR ≥ 90% point estimate (or lower CI bound ≥ 86%)
- FTFR data presented to board at Month 20 business review
- If gate not met: Phase 5 expansion delayed; engineering sprint opens to address root cause

---

## FTFR in Sales Narratives

| Audience | Framing |
|---|---|
| Dealer principals | "Your technicians will fix it right the first time, 9 times out of 10 — reducing the cost of comeback repairs and improving customer satisfaction scores" |
| Service managers | "FTFR of 90%+ means fewer same-vehicle appointments, more throughput per bay, and less warranty dispute exposure" |
| OEM regional managers | "Our platform improves OEM-brand service quality metrics — FTFR improvement translates directly to Customer Service Index (CSI) scores" |
| Investors | "North Star metric achieved: 23 percentage point FTFR improvement demonstrated on n=200 repairs at statistical significance" |

---

## Links to Related Specs

- FTFR baseline methodology: `docs/metrics/ftfr_baseline_spec.md`
- Control group design (MTTD + FTFR): `docs/metrics/mttd_control_group_spec.md`
- Manager dashboard (FTFR display): `docs/features/manager_dashboard_spec.md`
- Job card integration (repair completion events): `docs/features/job_card_integration_spec.md`
- DiagnosticCore (RAG pipeline, confidence scoring): `docs/agents/diagnostic_core_v1.md`
