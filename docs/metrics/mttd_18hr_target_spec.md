# MTTD ≤ 1.8 Hours Spec — Phase 4 Diagnostic Speed Target
## Step 46 — Phase 4, Metrics 2

**Roadmap reference:** Phase 4, Metrics 2
**Status:** `[ ]` Not started
**Owner:** Product + Data Analytics
**Target:** MTTD ≤ 1.8 hours for EV TMS faults, demonstrated by Month 20
**Prerequisite:** Phase 3 MTTD baseline + control group data (`docs/metrics/mttd_control_group_spec.md`)

---

## Objective

Demonstrate that X-Ray Vision Diagnostics reduces Mean Time to Diagnose (MTTD) for EV thermal management system (TMS) faults from the 3.4-hour industry baseline to ≤ 1.8 hours — a 47% reduction. This is the secondary efficiency metric alongside FTFR and translates directly into bay throughput and technician labor cost.

**Why EV TMS faults specifically:** EV TMS faults (battery coolant leak, thermal runaway precursors, coolant pump failures) are the highest-complexity, highest-MTTD category in EV service. They involve multiple interacting subsystems (battery, coolant loop, thermal sensors, CAN bus), making them the best showcase for spatial overlay's value. ICE and simpler EV faults have lower baseline MTTD and leave less improvement headroom.

---

## MTTD Definition and Measurement

**MTTD = elapsed time from DTC first read (session start) to diagnosis confirmed (technician confirms root cause + selects repair action)**

### Timestamp Sources

| Event | Timestamp Source | Agent |
|---|---|---|
| Session start | OBD-II DTC pull timestamp | SensorFusion |
| Voice query (first layer activation) | VoiceNLP intent packet timestamp | VoiceNLP |
| Diagnosis confirmed | Technician selects "Confirm diagnosis" in headset step flow | DiagnosticCore step event |
| Repair action selected | Job card step 1 written (repair begins) | Job card integration |

**MTTD window:** session start → diagnosis confirmed. Does not include repair execution time (that is a separate "Mean Time to Repair" metric, not in scope for Phase 4).

### What Is Excluded from MTTD

| Excluded Event | Rationale |
|---|---|
| Waiting for parts (parts not in stock) | Parts availability is not a diagnostic variable |
| Tech interrupted by another repair | Elapsed wall-clock time paused during interruption > 15 min (session idle detection) |
| System graceful degradation events > 5 min | If system was in degraded mode (latency > 40ms) for > 5 minutes, that window is excluded from MTTD as system-attributable delay |
| Safety stop events | HV-STOP durations excluded from MTTD calculation — safety events are correct behavior, not inefficiency |

---

## Phase-Over-Phase Progression

| Phase | MTTD Target | Description |
|---|---|---|
| Baseline (pre-product) | 3.4 hours | Industry average for EV TMS faults (IHS Markit, 2023) |
| Phase 3 exit | ≤ 2.7 hours | 20% improvement; control-group validated |
| Phase 4 exit | ≤ 1.8 hours | 47% improvement vs. baseline; GA launch target |
| Phase 5 target (aspirational) | ≤ 1.2 hours | 65% improvement; requires predictive pre-fetch + apprentice mode |

### Why ≤ 1.8 Hours Is Achievable by Phase 4

Phase 3 demonstrated ≤ 2.7 hours with a single-vehicle, single-fault deployment. Phase 4 improvements that contribute an additional ~55 minutes of reduction:

| Improvement | Estimated MTTD Reduction |
|---|---|
| Multi-fault expansion (Phase 3): DiagnosticCore trained on 5 fault types → narrower initial hypothesis space | ~15 minutes |
| Ford Mach-E addition: DiagnosticCore trained on cross-OEM fault patterns → better initial DTC disambiguation | ~10 minutes |
| RLHF confidence re-ranking (Phase 4): re-ranker surfaces correct repair path earlier in session | ~15 minutes |
| VoiceNLP Spanish support: eliminates cognitive translation step for 19% of technicians | ~5 minutes |
| Multi-OEM handoff: no session restart overhead between vehicles | ~10 minutes (amortized across shift) |

**Total projected additional reduction:** ~55 minutes. Phase 3 demonstrated 2.7h → targeting 2.7h - 0.9h = 1.8h.

---

## Measurement Protocol

### Treatment Group (X-Ray Vision Diagnostics)
- All enrolled technicians at Phase 4 commercial sites (50 bays, 10 dealerships)
- Fault scope: EV TMS faults only (DTC families: P0A00–P0AFF, U3000-series TMS-related)
- Minimum sample: 100 EV TMS repair sessions for statistical significance at Phase 4 scale

### Control Group (Tablet + OBD-II Scanner)
- Same matched control group design as Phase 3 (`docs/metrics/mttd_control_group_spec.md`)
- Control technicians at same sites but non-enrolled bays, or at control dealerships with matched characteristics
- Control MTTD measured using same timestamp methodology: DTC pull → repair order written

### Difference-in-Differences Analysis
- Compare MTTD change from Phase 3 baseline to Phase 4 end:
  - Treatment: 3.4h (baseline) → X.Xh (Phase 4)
  - Control: 3.4h (baseline) → Y.Yh (Phase 4)
  - DiD estimate: (X.X - 3.4) - (Y.Y - 3.4) = true tool effect

---

## Agent-Level MTTD Attribution

Understanding which agents contribute most to MTTD reduction informs Phase 5 investment priorities.

| Agent | MTTD Contribution | How Measured |
|---|---|---|
| DiagnosticCore | Time from DTC pull to first correct layer manifest generated | DiagnosticCore reasoning log: DTC ingestion timestamp vs. LayerSelectionManifest v1 timestamp |
| CADRenderer | Time from layer manifest to first spatial overlay rendered | CADRenderer log: LayerSelectionManifest received → first frame rendered |
| VoiceNLP | Time from tech utterance to DiagnosticCore receiving intent | VoiceNLP: wake word detected → IntentPacket delivered |
| SensorFusion | Time from session start to UnifiedSensorState stable (all sensors locked) | SensorFusion: session start → first stable UnifiedSensorState |

**Target sub-timings:**
- SensorFusion stable state: ≤ 30 seconds from bay entry
- DiagnosticCore first manifest: ≤ 45 seconds from DTC pull
- CADRenderer first render: ≤ 200ms from manifest (already in spec)
- VoiceNLP intent delivery: ≤ 150ms from utterance (already in spec)

The majority of MTTD is technician reasoning time, not system latency. The product reduces technician reasoning time by surfacing the correct spatial context faster — not by improving pipeline latency alone.

---

## Dashboard Display

MTTD surfaces in the manager dashboard (`docs/features/manager_dashboard_spec.md`) as:
- Rolling 30-day MTTD trend (line chart) — broken down by fault type
- MTTD vs. control group (paired bar chart, updated monthly)
- Individual technician MTTD vs. site average (table — used for coaching, not punitive)
- Alert: if site MTTD increases > 15% above rolling average for 7 consecutive days → product team and site manager notified

---

## Statistical Significance Requirements

| Claim | Required n | 95% CI |
|---|---|---|
| Internal operational tracking | 50 EV TMS repairs | ±20 min |
| External marketing claim | 100 EV TMS repairs | ±12 min |
| OEM case study | 150 EV TMS repairs per OEM line | ±10 min |

At n=100, assumed σ = 45 minutes (typical MTTD standard deviation for EV faults):
95% CI = ±1.96 × (45 / √100) = ±8.8 minutes

**Marketing claim threshold:** Demonstrated MTTD must have upper bound of 95% CI ≤ 2.0 hours to claim "MTTD under 2 hours." At n=100, this means point estimate must be ≤ 111 minutes.

---

## Phase 4 Exit Gate Criteria

- n ≥ 100 EV TMS fault repair sessions with complete MTTD measurement
- Point estimate MTTD ≤ 1.8 hours (108 minutes)
- 95% CI upper bound ≤ 2.0 hours (120 minutes)
- Improvement vs. control group statistically significant (p < 0.05)
- No concurrent confound identified (e.g., OEM TSB update that independently reduced diagnosis time)

If exit gate not met by Month 20: Phase 5 expansion delayed; DiagnosticCore RAG corpus audit and RLHF re-ranker performance review opened.

---

## Links to Related Specs

- Phase 3 MTTD baseline + control group design: `docs/metrics/mttd_control_group_spec.md`
- DiagnosticCore reasoning pipeline: `docs/agents/diagnostic_core_v1.md`
- SensorFusion session initialization: `docs/agents/sensor_fusion_v1.md`
- RLHF pipeline (re-ranker improvement): `docs/features/rlhf_pipeline_spec.md`
- Manager dashboard (MTTD display): `docs/features/manager_dashboard_spec.md`
