# Technician Confidence Score — Post-Repair Survey Spec
## Step 25 — Phase 3, Feature 1

**Roadmap reference:** Phase 3, Feature 1
**Status:** `[ ]` Not started
**Owner:** PM + UX Research
**Target:** Live in beta deployment by Month 10

---

## Objective

Capture technician confidence after each repair session to measure whether X-Ray Vision Diagnostics increases diagnostic certainty and reduces second-guessing. Confidence score is a leading indicator of FTFR (First-Time Fix Rate) — a tech who is confident in their diagnosis is more likely to complete the repair without a return visit.

**North Star connection:** FTFR target ≥ 90%. Confidence score target ≥ 3.8/5.0 in beta.

---

## Survey Design

### Delivery
- Triggered automatically when technician says "Complete repair" or closes the diagnostic session
- Rendered in-headset as a full-screen modal overlay (no hand controller required)
- Input method: voice response ("one", "two", "three", "four", "five") or gaze-dwell on option (1.5 second dwell = selection)
- Total time target: < 60 seconds for all 5 questions

### Questions

| # | Question | Scale | Rationale |
|---|---|---|---|
| Q1 | "How confident are you that the fault was correctly identified?" | 1–5 | Diagnostic accuracy signal |
| Q2 | "How confident are you that the repair was completed correctly?" | 1–5 | Repair execution signal |
| Q3 | "Did the system help you find information you wouldn't have found with a standard scan tool?" | 1 (No) / 5 (Yes, significantly) | Incremental value signal |
| Q4 | "How easy was it to navigate the system during this repair?" | 1–5 | UX friction signal |
| Q5 | "Would you use this system on your next EV repair?" | 1–5 | Adoption intent signal |

### Scale Labels (displayed in-headset)

| Value | Label |
|---|---|
| 1 | Not at all confident / Not at all |
| 2 | Slightly |
| 3 | Moderately |
| 4 | Quite confident |
| 5 | Completely confident / Absolutely |

### Skip Logic
- If HV-STOP was triggered during the session: Q2 is replaced with "Was the HV safety alert appropriate and clear?" (same 1–5 scale) — safety validation signal
- If tech abandons session before repair completion: only Q4 and a free-text field are shown

---

## Data Schema

```json
{
  "survey_id": "uuid",
  "session_id": "session_uuid",
  "technician_id": "anon_hash",
  "dealership_id": "dlr_001",
  "vehicle_vin": "JTMEWRFV1PD012345",
  "fault_type": "coolant_leak",
  "oem": "toyota",
  "survey_timestamp_utc": "2025-10-14T16:32:11Z",
  "q1_confidence_diagnosis": 4,
  "q2_confidence_repair": 5,
  "q3_incremental_value": 4,
  "q4_ux_ease": 3,
  "q5_adoption_intent": 5,
  "hv_stop_triggered": false,
  "session_duration_min": 47,
  "completion_status": "completed"
}
```

Stored on edge server, synced to cloud analytics pipeline at end of session (or on next connectivity window if offline).

---

## Composite Confidence Score

The single reported "Confidence Score" is a weighted composite:

```
ConfidenceScore = (Q1 × 0.4) + (Q2 × 0.4) + (Q3 × 0.1) + (Q4 × 0.1)
```

Q5 (adoption intent) is tracked separately — it is a business metric, not a confidence metric.

**Target:** ConfidenceScore ≥ 3.8 across all beta sessions.

---

## Privacy & Anonymization

- Technician ID is stored as a salted SHA-256 hash of their employee ID — not their name
- Dealer name is stored; technician name is never stored
- Aggregate scores displayed in Manager Dashboard (see `docs/features/manager_dashboard_spec.md`)
- Individual scores visible only to: QA team, PM, and the technician's own history view

---

## Manager Dashboard Integration

Confidence Score feeds the Manager Dashboard as:
- Rolling 7-day average by fault type
- Rolling 7-day average by technician (anonymized in aggregate view)
- Trend chart over beta period
- Alert if any fault type scores < 3.0 for 3 consecutive sessions — triggers PM + UX review

---

## Exit Criteria

- [ ] Survey renders and responds to voice input in < 3 seconds from session close
- [ ] Survey completes in median < 45 seconds across 10 test runs
- [ ] Data schema writes correctly to edge server and syncs to cloud
- [ ] ConfidenceScore composite calculation verified in unit tests
- [ ] Manager Dashboard displays rolling score correctly
