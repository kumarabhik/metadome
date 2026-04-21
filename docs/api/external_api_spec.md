# External API Specification

**Version:** Phase 5
**Audience:** Integration partners (CDK Global, Reynolds & Reynolds, Toyota, Ford), enterprise customers building custom integrations

---

## Overview

X-Ray Vision Diagnostics exposes a REST API for external integrations. All endpoints require API key authentication. All data is JSON. All timestamps are ISO 8601 UTC.

**Base URL:** `https://api.xvd.com/v1`
**Authentication:** `Authorization: Bearer {api_key}` header on all requests
**Rate limits:** 100 requests/minute per API key (contact support for higher limits)

---

## 1. Job Card Integration API

Used by: CDK Drive, Reynolds & Reynolds, DealerSocket, and custom DMS integrations.
Purpose: Push repair session data into dealer management system job cards.

### POST /webhooks/job-card

Register a webhook endpoint to receive job card events.

**Request:**
```json
{
  "url": "https://yourcdkinstance.com/xvd/webhook",
  "events": ["session.completed", "step.completed", "hv_stop.triggered"],
  "dealer_id": "DEALER-001",
  "secret": "your_hmac_secret"
}
```

**Events pushed to your endpoint:**

#### session.completed
```json
{
  "event": "session.completed",
  "session_id": "uuid",
  "vin": "1FMSK8DH4NGA12345",
  "dealer_id": "DEALER-001",
  "tech_id": "TECH-042",
  "fault_type": "ev_battery_coolant_leak",
  "vehicle_model": "Toyota bZ4X 2024",
  "started_at": "2025-03-15T09:14:00Z",
  "completed_at": "2025-03-15T11:22:00Z",
  "duration_minutes": 128,
  "steps_completed": 7,
  "steps_total": 7,
  "oem_bulletin_references": ["TSB-0045-23"],
  "outcome": "completed | abandoned",
  "hv_stop_triggered": false
}
```

#### step.completed
```json
{
  "event": "step.completed",
  "session_id": "uuid",
  "step_number": 3,
  "step_description": "Disconnect coolant inlet line",
  "completed_at": "2025-03-15T10:05:00Z",
  "dwell_seconds": 142
}
```

**CDK Drive mapping:**
- `session.completed` → auto-populate repair order labor time (`completed_at - started_at`)
- `step.completed` events → populate repair notes field with timestamped step log
- `oem_bulletin_references` → populate "Cause" field with TSB reference

**Reynolds & Reynolds XML format:**
For R&R integrations that require XML (not webhook): GET /export/job-card/{session_id}?format=xml returns a Reynolds-compatible XML payload. Schema available at `GET /schemas/reynolds-job-card.xsd`.

---

## 2. OEM Learning Management System (LMS) API

Used by: Toyota TTMS, Ford Motorcraft Training portal, Stellantis CAP
Purpose: Export technician Apprentice Mode scores to OEM certification programs.

### POST /lms/export

Export a completed Apprentice Mode session score to an OEM LMS.

**Request:**
```json
{
  "oem": "toyota | ford | stellantis",
  "tech_oem_employee_id": "TOYOTA-EMP-10042",
  "session_id": "uuid",
  "fault_type": "ev_battery_coolant_leak",
  "vehicle_model": "Toyota bZ4X",
  "score": 87.5,
  "grade": "B",
  "date": "2025-03-15",
  "checkpoint_detail": true
}
```

**Response:**
```json
{
  "export_id": "uuid",
  "oem_confirmation_id": "TTMS-2025-98712",
  "status": "accepted | pending | rejected",
  "oem_module_mapped": "bZ4X EV Battery TMS Certification — Module 3"
}
```

**Toyota TTMS integration notes:**
- REST API, JSON, OAuth 2.0 client credentials flow
- `tech_oem_employee_id` maps to Toyota's T-TEN enrollment ID
- Module mapping: maintained in `POST /lms/oem-module-map/toyota` — product team updates when Toyota adds new modules

**Ford Motorcraft Training notes:**
- REST API (Ford Developer Portal), API key auth
- Grade mapping: A=Mastery, B=Proficient, C=Developing, Not Passed=Not Demonstrated
- Ford requires `checkpoint_detail: true` for Level 3 certifications (only FACT Level 3)

---

## 3. OEM Telematics API (Inbound)

Used by: Toyota Connected Services, Ford BlueCruise Connect
Purpose: Receive between-visit vehicle sensor data for predictive maintenance.

### Inbound webhook: POST /telematics/ingest

OEM telematics platform pushes sensor snapshots to this endpoint on schedule (daily or on-fault-event).

**Authentication:** OEM-provided HMAC-SHA256 signature on `X-OEM-Signature` header; verified against OEM public key stored in our SSM Parameter Store.

**Toyota Connected Services payload:**
```json
{
  "source": "toyota_connected",
  "vin": "JTDKARFP5N3123456",
  "snapshot_timestamp": "2025-03-15T06:00:00Z",
  "sensors": {
    "battery_soc_percent": 78.2,
    "battery_temp_c": 22.1,
    "coolant_flow_rate_lpm": 4.8,
    "cell_voltage_min_v": 3.71,
    "cell_voltage_max_v": 3.79,
    "cell_voltage_variance": 0.08,
    "isolation_resistance_kohm": 892,
    "odometer_km": 24150,
    "charging_cycles": 187
  },
  "dtcs_active": [],
  "dtcs_pending": ["P0A80"]
}
```

**Response:** `202 Accepted` with `{"ingest_id": "uuid"}`. Processing is async; sensor data available in predictive maintenance pipeline within 30 minutes.

**Data residency:** Canadian VINs are routed to ca-central-1 ingest endpoint (`https://api-ca.xvd.com/v1/telematics/ingest`); UK VINs to eu-west-2 (`https://api-uk.xvd.com/v1`).

---

## 4. Manager Dashboard API

Used by: Dealer group custom reporting, DMS integrations that want to pull FTFR data.

### GET /analytics/dealer/{dealer_id}/ftfr

Returns FTFR data for a dealer over a specified period.

**Request:**
```
GET /analytics/dealer/DEALER-001/ftfr?start=2025-01-01&end=2025-03-31&fault_type=all
Authorization: Bearer {api_key}
```

**Response:**
```json
{
  "dealer_id": "DEALER-001",
  "period": {"start": "2025-01-01", "end": "2025-03-31"},
  "ftfr_overall": 0.918,
  "repairs_total": 73,
  "by_fault_type": [
    {
      "fault_type": "ev_battery_coolant_leak",
      "ftfr": 0.933,
      "repairs": 18
    }
  ],
  "trend": [
    {"week": "2025-W01", "ftfr": 0.87, "repairs": 8},
    {"week": "2025-W02", "ftfr": 0.92, "repairs": 11}
  ]
}
```

### GET /analytics/dealer/{dealer_id}/sessions

Returns session-level data (no PII, no VIN in response by default).

**Parameters:** `start`, `end`, `fault_type`, `tech_cohort` (senior/mid/junior), `include_vin=true` (requires additional data agreement)

---

## 5. SOC 2 Audit Log API (Read-only, Compliance)

Used by: OSHA inspectors (on request), enterprise customers exercising audit rights, internal compliance team.

### GET /audit/hv-stop/{session_id}

Returns the complete HV-STOP audit log for a session. Requires `X-Audit-Reason` header and `audit_access` API key scope.

**Response:**
```json
{
  "session_id": "uuid",
  "hv_stop_events": [
    {
      "event_id": "uuid",
      "triggered_at": "2025-03-15T10:42:17.223Z",
      "trigger_source": "thermal",
      "activation_latency_ms": 42,
      "proximity_distance_m": 0.38,
      "hv_zone": "battery_module_left",
      "acknowledged_at": "2025-03-15T10:42:45.001Z",
      "acknowledged_by": "TECH-042",
      "acknowledgment_method": "voice+gesture",
      "session_resumed_at": "2025-03-15T10:43:12.444Z"
    }
  ],
  "file_hash_sha256": "a3f8...",
  "log_generated_at": "2025-03-15T11:22:00Z"
}
```

**Tamper evidence:** File hash is computed at log generation and stored immutably in S3. API response includes hash; caller can verify by re-computing hash of the event payload.

---

## 6. Webhook Event Reference (Full List)

| Event | Trigger | Consumers |
|---|---|---|
| `session.started` | New session begins | DMS, analytics |
| `session.completed` | Session ends — all steps done | DMS (job card), OEM LMS |
| `session.abandoned` | Session ended before completion | CS health score, analytics |
| `step.completed` | Individual step confirmed | DMS (repair notes) |
| `hv_stop.triggered` | SafetyGuard fires HV-STOP | Compliance, CS alerts |
| `hv_stop.acknowledged` | Tech confirms isolation | Compliance audit log |
| `hv_stop.overridden` | Supervisor override (emergency) | Compliance — immediate alert |
| `predictive_alert.accepted` | Tech adds predictive alert to RO | DMS (job card pre-population) |
| `recording.uploaded` | Session recording available | CS, manager dashboard |
| `nps_survey.completed` | Tech submits NPS response | CS health score, Salesforce |
| `apprentice_score.submitted` | Apprentice Mode session scored | OEM LMS export |

---

## API Versioning and Deprecation

- Current version: `v1`
- Breaking changes: minimum 6-month deprecation notice; old version supported for 12 months after new version ships
- Non-breaking changes (new fields, new events): added without version bump; documented in changelog at `GET /changelog`
- Deprecation notices sent via email to all registered API key holders

## SDKs

Official SDKs available on request (GitHub private repos, access granted to API key holders):
- Python SDK: `xvd-python` — covers all endpoints; async support
- Node.js SDK: `xvd-node` — webhook event parsing + typed response models
- No Java/C# SDK currently (Phase 6 roadmap based on enterprise customer demand)
