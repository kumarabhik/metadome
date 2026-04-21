# MTTD Measurement Spec — System vs. Tablet Control Group
## Step 32 — Phase 3, Metrics 2

**Roadmap reference:** Phase 3, Metrics 2
**Status:** `[ ]` Not started
**Owner:** PM + Data Analytics
**Target:** MTTD baseline established by Month 10

---

## Objective

Measure Mean Time to Diagnose (MTTD) for X-Ray Vision Diagnostics vs. a matched control group of technicians using standard scan tablets. Target: ≥20% MTTD reduction (3.4h → ≤2.7h for EV TMS faults).

**Why MTTD matters alongside FTFR:** A system can achieve high FTFR by having technicians spend unlimited time diagnosing. MTTD ensures the system is fast *and* accurate — not just accurate.

---

## MTTD Definition

**Mean Time to Diagnose** = average elapsed time from vehicle check-in to technician signing off on the repair plan (before physical repair begins).

```
MTTD = mean(t_repair_plan_signoff - t_vehicle_checkin)
```

Measured separately for:
- **System group:** repairs where headset session was used for diagnosis
- **Control group:** repairs on same fault types using standard scan tool + tablet

**Industry baseline:** 3.4 hours for EV TMS faults (from J.D. Power 2023 EV Diagnostics Report, cited in `design_doc.md`). Phase 3 target: ≤2.7 hours.

---

## Control Group Design

### Matching Strategy
Control group is not a separate dealership — it is the same 3 beta dealerships, same technicians, for repairs that happened before headset availability OR on fault types not yet enrolled in the system.

This within-dealership, within-technician design controls for:
- Technician skill level
- Shop floor culture and pace
- Vehicle model complexity
- Time of year (seasonality)

### Assignment
- System group: any repair where a headset session log exists (VIN + DTC match)
- Control group: repairs at same sites on same fault families, no headset session log
- No randomization needed — system group is self-selected (tech chose to use headset); control is historical

**Confound risk:** Selection bias — skilled techs may adopt headset first. Mitigate by: (a) tracking adopter experience level, (b) comparing each tech's own pre/post MTTD where possible.

---

## Data Sources

### Time Stamps

| Timestamp | Source | Field |
|---|---|---|
| Vehicle check-in | DMS (CDK/R&R) | RO open time |
| Repair plan signoff | DMS | Labor line "diagnosis complete" flag |
| Headset session start | Edge server session log | `session_start_utc` |
| Headset session end | Edge server session log | `session_end_utc` |

**Important:** DMS "diagnosis complete" flag is set by the service writer when the tech verbally signs off on the repair plan, not when the headset session closes. The two timestamps should correlate but are tracked independently to validate accuracy.

### Data Pull Cadence
- Monthly, aligned with FTFR pull (same data pipeline, additional timestamp fields)

---

## Segmentation

### By Fault Type

| Fault | Industry Baseline | Phase 3 Target | Phase 4 Target |
|---|---|---|---|
| EV coolant leak | 3.4h | ≤2.7h | ≤1.8h |
| EV battery cell | 4.1h (estimated) | ≤3.3h | ≤2.5h |
| Hybrid inverter | 3.8h (estimated) | ≤3.0h | ≤2.2h |
| ICE timing chain | 2.1h (estimated) | ≤1.7h | ≤1.4h |
| Brake system leak | 1.2h (estimated) | ≤1.0h | ≤0.8h |

> Industry baselines for non-TMS faults are estimated from NADA service time studies. Phase 3 will establish the true baseline for these faults at beta sites.

### By Technician Experience
- Junior (< 5 years): expect larger MTTD improvement (system closes knowledge gap)
- Senior (≥ 10 years): smaller improvement; primarily time savings on CAD navigation

---

## Statistical Analysis

**Primary test:** Two-sample t-test, system vs. control MTTD distributions
**Secondary test:** Paired pre/post t-test per technician (where pre-headset data exists)
**Power:** 80% to detect 20% MTTD reduction at α = 0.05 requires n ≈ 42 per group
**Outlier handling:** Remove ROs where DMS shows total bay time > 3× median (likely multi-day job held open incorrectly)

---

## Reporting Format

Monthly report card per fault type:

```
Fault: EV Coolant Leak
System group:   n=18, MTTD = 2.4h (SD 0.6h)
Control group:  n=14, MTTD = 3.3h (SD 0.8h)
Reduction:      27% ← exceeds 20% target
p-value:        0.003 ← statistically significant
```

---

## Exit Criteria

- [ ] ≥ 42 repairs per group (system + control) for primary EV TMS fault category
- [ ] MTTD reduction ≥ 20% for EV TMS faults demonstrated
- [ ] Segmentation by fault type and technician experience documented
- [ ] Statistical significance confirmed (p < 0.05) or sample size gap explained
- [ ] Results shared with service managers at all 3 beta sites
