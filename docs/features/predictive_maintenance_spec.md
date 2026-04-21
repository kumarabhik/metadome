# Predictive Maintenance Mode

**Phase:** 5
**Roadmap Item:** Feature: Predictive maintenance mode — DiagnosticCore surfaces likely failures before DTC is thrown, based on sensor drift patterns

---

## Objective

Extend DiagnosticCore beyond reactive diagnosis (DTC thrown → tech investigates) to proactive surfacing of likely faults based on sensor drift patterns observed over time. A technician bringing in a vehicle for routine service sees a flag: "Battery coolant flow rate trending 12% below baseline — coolant pump wear pattern consistent with early-stage failure." The repair happens before the vehicle breaks down.

This is the product's highest-value differentiation: moving from a diagnostic tool to a predictive service platform.

---

## How It Works

### Data Foundation
Predictive maintenance requires longitudinal sensor data — the same vehicle's sensor readings observed across multiple service visits. Data sources:

1. **OBD-II historical telemetry** — each time a vehicle is connected in a bay, SensorFusion records a snapshot of all available PIDs (sensor values) tagged with VIN + timestamp
2. **OEM telematics feed** — Toyota Connected Services, Ford BlueCruise/FordPass Connect, and equivalent OEM telematics APIs provide continuous between-visit sensor readings for enrolled vehicles (requires OEM data agreement extension from Phase 1)
3. **Fleet baseline** — aggregate of all same-VIN-class vehicles in the system; provides population norms for "healthy" sensor values by mileage band

### DiagnosticCore Predictive Extension

DiagnosticCore v1 (Phase 2) processes DTCs and generates a fault-specific layer manifest. The predictive extension adds a parallel pipeline:

```
VIN lookup → fetch longitudinal sensor history (bay visits + telematics)
           → run drift detection model against fleet baseline
           → generate PredictiveAlertManifest if drift score > threshold
           → surface alert in headset overlay pre-repair
```

**Drift Detection Model:**
- Input: time-series of sensor values for a given PID (e.g., coolant flow rate, cell voltage variance, battery inlet temp delta)
- Method: exponentially weighted moving average (EWMA) control chart — flags when sensor reading moves outside 2σ band of fleet baseline for same mileage cohort
- No LLM in the drift detection path — statistical model only; interpretable and auditable
- LLM used only for: generating the plain-language alert description shown to the technician ("coolant pump wear pattern consistent with early-stage failure") — grounded in OEM service bulletin, not freeform generation

**PredictiveAlertManifest Schema:**
```json
{
  "vin": "string",
  "alert_id": "uuid",
  "sensor_pid": "string",
  "current_value": "float",
  "fleet_baseline_p50": "float",
  "drift_score": "float",
  "confidence": "low | medium | high",
  "likely_fault_family": "string",
  "recommended_action": "string",
  "oem_bulletin_reference": "string | null",
  "generated_at": "ISO8601"
}
```

---

## UX — In-Headset Experience

### Entry Point
Predictive alerts surface at the start of a service visit, before the technician opens any specific fault flow:

1. Tech scans VIN (or vehicle auto-detected by ArUco marker)
2. DiagnosticCore checks: any active DTCs? Any predictive alerts above threshold?
3. Headset shows summary card: "No active faults. 1 predictive alert available — coolant system."
4. Tech says "Show predictive alerts" → expands to full alert card

### Alert Card
- Plain-language description: "Coolant flow rate has trended 12% below fleet average over 3 service visits. Consistent with early-stage coolant pump wear. Recommend: inspect coolant pump and lines during this visit."
- OEM bulletin reference (if available): "See TSB 0045-23 for coolant pump inspection procedure"
- Confidence indicator: LOW / MEDIUM / HIGH (based on drift score and data volume)
- Tech actions: "Add to repair order" / "Flag for review" / "Dismiss"

### "Add to Repair Order" Integration
- Tapping "Add to repair order" fires an event to the job card integration (Phase 3 spec) with `event_type: predictive_alert_accepted`
- Repair order line item pre-populated: "Inspect coolant pump — predictive alert basis"
- Timestamps recorded for metrics: alert_shown → tech_accepted → repair_completed → outcome_confirmed

---

## Confidence Levels and Thresholds

| Confidence | Drift Score | Data Requirement | Display Behavior |
|---|---|---|---|
| HIGH | > 3σ from fleet baseline | ≥ 3 prior service visits OR OEM telematics data ≥ 90 days | Alert shown proactively on visit start |
| MEDIUM | 2–3σ | ≥ 2 prior visits | Alert shown on visit start with "medium confidence" label |
| LOW | 1.5–2σ | 1 prior visit | Available on demand only ("Check historical trends") — not shown proactively |

LOW confidence alerts are not surfaced proactively to avoid alert fatigue. Technicians can access them via "Sensor history" voice command.

---

## Privacy and Data Governance

- Vehicle sensor history is indexed by VIN — no driver PII is collected or needed
- OEM telematics data: de-identified before ingestion (VIN retained, no owner name/location/trip history)
- Predictive alert data not shared with OEM without dealer opt-in (alerts are dealer competitive advantage)
- Retention: sensor snapshots retained 36 months per vehicle; purged on VIN decommission

---

## Rollout Plan

| Month | Milestone |
|---|---|
| 22 | Extend OEM data agreements to include telematics feed (Toyota Connected Services, Ford BlueCruise) |
| 23 | Build longitudinal sensor history store (VIN-indexed time-series, Timescale DB on edge server) |
| 24 | Build drift detection model — EWMA control chart; validate against Phase 2–4 bay data |
| 25 | Build PredictiveAlertManifest generator and DiagnosticCore integration |
| 26 | Build in-headset alert card UX |
| 27 | Internal testing: run against 200 vehicles with known outcomes (retrospective validation) |
| 28 | Closed beta: 3 dealerships from Phase 3 beta cohort — measure alert acceptance rate and outcome accuracy |
| 30 | GA launch alongside Phase 5 commercial expansion |

---

## Success Metrics

| Metric | Target |
|---|---|
| Alert acceptance rate (tech adds to RO) | ≥ 35% of HIGH confidence alerts |
| Alert accuracy (accepted alert → fault confirmed at inspection) | ≥ 70% |
| False positive rate (alert accepted → no fault found) | ≤ 30% |
| Reduction in unplanned breakdowns for enrolled vehicles | ≥ 20% vs. control group (tracked via OEM telematics) |
| FTFR on predictive-maintenance repairs | ≥ 95% (these are planned repairs with pre-diagnosis) |

---

## Risks

| Risk | Mitigation |
|---|---|
| Insufficient longitudinal data in early months | Retrospective data from Phase 2–4 bays seeds the model; telematics data fills gaps |
| OEM telematics API reliability / data quality | Build with degraded mode: if telematics unavailable, alert still possible from bay-visit snapshots only |
| Alert fatigue if thresholds too low | LOW confidence not shown proactively; weekly review of acceptance rate → tune thresholds |
| Liability: tech acts on alert, no fault found, dealer disputes labor cost | Alert is a recommendation, not a directive; disclaimer on alert card; repair order records tech's acceptance decision |
