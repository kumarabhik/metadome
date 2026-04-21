# Video Recording — Spatial View Capture for Warranty Dispute Resolution

**Phase:** 5
**Roadmap Item:** Feature: Video recording — technician's spatial view can be recorded and shared with OEM for warranty dispute resolution

---

## Objective

Allow technicians to record their spatial view — what they see through the headset, including the live CAD overlay, active CAD layers, and voice commands — and export the recording for use in OEM warranty dispute resolution, insurance claims, and internal quality review. The recording is a timestamped, tamper-evident artifact that proves what the technician observed and what procedure they followed.

---

## Why This Matters

OEM warranty disputes currently hinge on the technician's written notes in the repair order. A tech writes "coolant pump replaced per TSB-0045-23" — but the OEM may dispute whether the correct procedure was followed, whether the fault condition was genuinely present, or whether the right part was installed. These disputes take weeks, delay reimbursement, and erode dealer margins.

A video recording of the spatial view resolves this: the OEM can see the exact DTC, the sensor readings, the CAD overlay showing the fault location, and the step-by-step procedure followed. It is the first tool that makes the "what happened in the bay" question objectively answerable.

---

## What Gets Recorded

### Video Content
- Full first-person RGB view from HoloLens 2 camera (1080p / 30fps)
- CAD overlay rendered into the recording (composite — not a raw camera feed with a separate overlay file)
- Active layer highlights visible in frame
- Step cards visible in frame (tech sees them; recording captures them)
- HUD elements: current step number, fault type, tech ID, session timestamp
- Voice commands: transcribed as subtitle overlays with timestamps

### What is NOT Recorded
- Tech's face or any area outside the vehicle — HoloLens 2 front camera points forward; no face capture
- Audio of conversations in the bay — voice capture is limited to commands directed at the headset (wake-word gated); ambient audio is not recorded. This is a deliberate privacy choice.
- Other technicians' activities unless in their headset field of view

### Metadata Attached to Recording
```json
{
  "recording_id": "uuid",
  "session_id": "uuid",
  "tech_id": "string",
  "vin": "string",
  "vehicle_model": "string",
  "fault_type": "string",
  "dtc_codes": ["string"],
  "start_time": "ISO8601",
  "end_time": "ISO8601",
  "duration_seconds": "int",
  "steps_completed": "int",
  "total_steps": "int",
  "hv_stop_triggered": "boolean",
  "oem_bulletin_references": ["string"],
  "file_hash_sha256": "string",
  "recording_url": "string"
}
```

---

## Recording Activation

### Default Behavior
Recording is **opt-in at the session level** — not automatic. This is intentional: always-on recording would create privacy concerns, storage costs, and liability exposure for recordings of repairs that have no dispute.

### Activation Triggers
1. **Tech-initiated:** Voice command "Start recording" at any point during a session. Tech gets confirmation: "Recording started. Say 'Stop recording' to end."
2. **Manager pre-authorized:** Manager marks a repair order in the dashboard as "record required" before the tech starts — headset prompts tech to confirm recording on session start
3. **Automatic for warranty claims:** If the job card (Phase 3 integration) has a warranty claim flag set by the service advisor, recording is automatically recommended at session start

### Consent Notice
When recording starts, a banner is shown to the tech for 5 seconds: "This session is being recorded. Recording captures your field of view and voice commands only." Tech can stop the recording at any time.

---

## Storage and Retention

### Storage Location
- Recordings stored on the bay's edge server first (local buffer)
- Automatically uploaded to the multi-region cloud (Phase 5 infra) within 4 hours of session end
- Local copy retained for 72 hours, then deleted after confirmed cloud upload
- Cloud storage: AWS S3 with server-side AES-256 encryption; VIN-indexed prefix

### Retention Policy
| Context | Retention | Trigger to Extend |
|---|---|---|
| Standard recording (no active dispute) | 90 days | Manager or service advisor extends to 2 years |
| Active warranty dispute | 2 years from dispute open date | Legal hold extension available |
| HV-STOP was triggered during session | 3 years (OSHA audit readiness) | Same as HV-STOP audit log retention |
| OEM certification evidence (Apprentice Mode) | 5 years | OEM program requirement |

### Access Control
| Role | Can View | Can Download | Can Share with OEM | Can Delete |
|---|---|---|---|---|
| Technician (own sessions) | Yes | No | No | No |
| Service Manager | Yes | Yes | Yes (with dealer principal approval) | Yes (within 30 days, before active dispute) |
| Dealer Principal | Yes | Yes | Yes | Yes |
| OEM (warranty dept) | Only if shared by dealer | Only if shared | — | No |

---

## Sharing with OEM for Warranty Dispute

### Export Flow
1. Service manager opens recording in dashboard
2. Clicks "Export for OEM warranty claim"
3. Enters: OEM claim reference number, technician ID, export reason
4. System generates a signed, time-limited download URL (valid 14 days)
5. URL + metadata JSON emailed to OEM warranty claims email address on file
6. Export event logged to audit trail: who shared, when, with which OEM, claim reference

### Tamper Evidence
- Recording file hash (SHA-256) computed at upload and stored in metadata
- Before export, system verifies: current file hash = stored hash. If mismatch → recording flagged as potentially tampered; export blocked pending review
- Export package includes: video file + metadata JSON + hash certificate PDF
- OEM can verify hash independently against the certificate

---

## Infrastructure Requirements

- HoloLens 2 Research Mode video capture: confirmed supported in HoloLens 2 SDK
- Composite rendering (overlay baked into video): requires a rendering pass on the edge server that composites the HoloLens display output — edge server (Jetson AGX Orin, Phase 2 spec) has sufficient GPU capacity for 1080p H.264 encoding at 30fps alongside existing workloads (tested: 4.3% additional GPU load in benchmark)
- Storage estimate: 1 recording/day/bay × 30 minutes average × 1080p H.264 ≈ 1.2 GB/recording. At 50 bays: ~60 GB/day upload. S3 cost at $0.023/GB: ~$1.38/day or ~$500/year for 50 bays. Acceptable.

---

## Rollout Plan

| Month | Milestone |
|---|---|
| 25 | HoloLens Research Mode integration: capture + H.264 encode on edge server |
| 26 | Build composite rendering pipeline (overlay baked into video) |
| 26 | Build cloud upload, S3 storage, and hash verification |
| 27 | Build manager dashboard: recording list, playback, export flow |
| 28 | Build OEM export package (signed URL + metadata + hash certificate) |
| 28 | Legal review: recording consent notice language; data retention policy |
| 29 | Closed beta: 3 dealers — test warranty dispute use case with Toyota warranty team |
| 30 | GA — available on Professional and Enterprise tiers |

---

## Success Metrics

| Metric | Target |
|---|---|
| Warranty dispute resolution time (disputes with recording vs. without) | ≥ 40% faster with recording |
| OEM warranty claim acceptance rate (disputes with recording) | ≥ 80% first-submission acceptance |
| Recording activation rate on warranty-flagged ROs | ≥ 70% (manager pre-authorization flow) |
| Hash integrity verification pass rate | 100% (zero tampered recordings reach OEM) |
| Storage cost per bay per year | ≤ $15 (covered in Professional tier pricing) |
