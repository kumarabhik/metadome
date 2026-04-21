# Job Card Integration Spec
## Step 27 — Phase 3, Feature 3

**Roadmap reference:** Phase 3, Feature 3
**Status:** `[ ]` Not started
**Owner:** PM + Integration Engineering
**Target:** Live at beta deployment, Month 9

---

## Objective

Auto-populate dealer DMS (Dealer Management System) job card repair timestamps from headset step-completion events. Eliminates the manual time entry that technicians currently perform after every repair — saving 5–10 minutes per job card and producing more accurate labor records for warranty claim audits.

**Business value:** Accurate timestamped job cards reduce warranty claim disputes with OEMs. Toyota and Ford both require step-level timestamps for covered EV battery repairs.

---

## Supported DMS Platforms (Phase 3 Beta)

| DMS | Market Share (US franchised dealers) | Integration Method |
|---|---|---|
| CDK Global | 38% | REST API (CDK Drive API v3) |
| Reynolds & Reynolds | 29% | File-based XML (ERA-IGNITE format) |

> Tekion, DealerSocket, and Serti are deferred to Phase 4. Phase 3 beta sites will be selected to use CDK or R&R.

---

## Job Card Data Model

A "job card" (also called RO — Repair Order) in the DMS represents a single vehicle visit. Each RO has one or more "operations" (repair tasks), each with a labor time entry.

### Current State (manual)
1. Tech completes repair
2. Tech walks to service writer's terminal
3. Manually enters: start time, end time, labor notes
4. Service writer closes RO

### Future State (integrated)
1. Tech completes each step in-headset ("Step complete" voice command)
2. Step completion event written to edge server with timestamp
3. At session close: step log sent to DMS API / file drop
4. DMS auto-populates: start time (step 1), end time (last step), labor time (calculated)
5. Optional: tech voice-dictates a repair note (pushed as labor note text)

---

## Events Published by Headset

Each step completion in the guided repair flow emits:

```json
{
  "event_type": "step_complete",
  "session_id": "session_uuid",
  "technician_id": "anon_hash",
  "vehicle_vin": "JTMEWRFV1PD012345",
  "ro_number": "RO-20251014-0042",
  "fault_type": "coolant_leak",
  "step_number": 3,
  "step_label": "Disconnect coolant pump connector",
  "timestamp_utc": "2025-10-14T15:44:22Z",
  "cumulative_labor_min": 28
}
```

Session-close event:
```json
{
  "event_type": "session_close",
  "session_id": "session_uuid",
  "ro_number": "RO-20251014-0042",
  "session_start_utc": "2025-10-14T15:16:00Z",
  "session_end_utc": "2025-10-14T16:44:22Z",
  "total_labor_min": 88,
  "steps_completed": 7,
  "steps_total": 7,
  "completion_status": "completed"
}
```

---

## CDK Drive API Integration

### Auth
- OAuth 2.0 client credentials flow
- Scope: `repair_orders:write`
- Token refresh: handled by edge server on 1-hour cadence

### Endpoint
```
PATCH /api/v3/repair-orders/{ro_number}/labor-lines/{operation_code}
Content-Type: application/json

{
  "startDateTime": "2025-10-14T15:16:00Z",
  "endDateTime": "2025-10-14T16:44:22Z",
  "laborMinutes": 88,
  "technicianNotes": "Coolant pump replaced per TSB-TMS-2024-01. All 7 guided steps completed.",
  "stepCompletionLog": [
    { "step": 1, "label": "...", "timestamp": "..." },
    ...
  ]
}
```

### Error Handling
- 401 Unauthorized: refresh token and retry once
- 404 RO not found: log error; surface in-headset "Job card not found — enter RO number manually"
- 429 Rate limited: queue and retry with 30s backoff
- Network unavailable: store locally; sync on next connectivity

---

## Reynolds & Reynolds XML Integration

R&R ERA-IGNITE accepts XML file drops to a watched directory (SFTP or local network share):

```xml
<RepairOrder>
  <RONumber>RO-20251014-0042</RONumber>
  <VIN>JTMEWRFV1PD012345</VIN>
  <LaborLine>
    <OperationCode>TMS-COOL-001</OperationCode>
    <TechID>ANON-A4F2</TechID>
    <StartTime>2025-10-14T15:16:00</StartTime>
    <EndTime>2025-10-14T16:44:22</EndTime>
    <LaborHours>1.47</LaborHours>
    <Notes>Guided repair system: 7/7 steps completed</Notes>
  </LaborLine>
</RepairOrder>
```

File placed in SFTP drop directory within 60 seconds of session close.

---

## RO Number Linkage

The system must know which RO to update. RO number is captured at session start:
1. **Primary:** VoiceNLP listens for "RO number [digits]" at session start
2. **Fallback:** Tech scans RO barcode via headset camera (future — Phase 4)
3. **Manual:** Service writer enters RO number in a pre-session web form; edge server picks it up

---

## Privacy & Audit

- Technician ID in DMS write is the anonymized hash — not the real employee ID
- Full step-completion log is retained on edge server for 90 days (warranty audit window)
- DMS receives only aggregate labor time; individual step details stay on edge server
- OEM warranty claim audits can request step logs via PM/Operations team

---

## Exit Criteria

- [ ] CDK API integration tested on CDK sandbox environment — correct RO update confirmed
- [ ] R&R XML format validated against ERA-IGNITE schema (provided by R&R partner)
- [ ] RO number captured via voice in < 5 seconds at session start
- [ ] Labor time accuracy: edge server calculation within ±2 minutes of manual stopwatch across 10 sessions
- [ ] Offline queue: job card data successfully syncs after 1-hour network outage simulation
