# Safety Architecture Document

**Version:** Phase 5 complete
**Audience:** SAE J1742 review panel, OSHA compliance auditors, OEM safety engineers, insurance underwriters
**Classification:** Technical Safety Design Documentation

---

## Purpose

This document describes the complete safety architecture of X-Ray Vision Diagnostics, with particular focus on high-voltage (HV) hazard mitigation. It is intended as the primary safety design reference for regulatory submissions and third-party safety audits.

The platform operates in the presence of high-voltage automotive battery systems (nominally 400V–800V DC). Any safety failure is a potential electrocution or fire hazard. Every architectural decision is evaluated against this constraint first.

---

## Applicable Standards

| Standard | Scope | Status |
|---|---|---|
| SAE J1742 | HV safety procedures for hybrid/EV service | Phase 4 certified (see `docs/compliance/sae_j1742_certification_spec.md`) |
| OSHA 1910.147 | Control of hazardous energy (lockout/tagout) | Phase 4 audit passed (see `docs/compliance/osha_1910_audit_pass_spec.md`) |
| OSHA 1910.269 | Electric power generation/distribution | Referenced for HV proximity requirements; J1742 is primary |
| IEC 61010-1 | Safety for electrical equipment in measurement | HoloLens 2 hardware compliance — Microsoft certified |
| UN ECE R100 | EV battery safety (UK market) | UK deployment compliant |
| Canada Labour Code Part II, S.125 | HV workplace safety (Canada) | Canada deployment compliant |

---

## Hazard Identification

### H-001: Direct HV Contact
**Description:** Technician makes physical contact with HV conductors (battery terminals, inverter bus bars, HV wiring orange cables) while system is energized.
**Severity:** Catastrophic (potentially fatal at 400V+ DC)
**Probability without mitigation:** Low-medium (experienced technicians follow LOTO; junior technicians are higher risk)

### H-002: Induced Voltage from HV Proximity
**Description:** Technician works in proximity to HV components without making direct contact; induced current causes physiological effect.
**Severity:** Serious (involuntary muscle contraction; secondary fall hazard)
**Probability without mitigation:** Low (requires very close proximity to unshielded HV components)

### H-003: Thermal Runaway Ignition During Service
**Description:** Battery thermal management fault present at start of service; additional disturbance during repair accelerates runaway.
**Severity:** Catastrophic (fire, explosion)
**Probability without mitigation:** Very low (TMS fault must pre-exist; handling disturbance is minor trigger)

### H-004: Incorrect Repair Instruction Causes Re-Energization
**Description:** Technician follows a repair step that inadvertently re-energizes the HV system (e.g., closes a relay during isolation check).
**Severity:** Catastrophic
**Probability without mitigation:** Low (trained technicians know LOTO); elevated for junior technicians following software-provided steps

### H-005: System Software Crash Removes Safety Overlay During HV Work
**Description:** Edge server or headset software crashes while technician is in proximity to HV components; safety zone overlay disappears.
**Severity:** Moderate (technician loses visual safety reference; does not create immediate hazard by itself)
**Probability without mitigation:** Low-medium (software systems can crash)

---

## Safety Controls

### Control Layer 1: Architectural Isolation (SafetyGuard Agent)

SafetyGuard runs as an isolated OS process on the edge server. It does not share thread pools, memory allocators, or IPC channels with DiagnosticCore, CADRenderer, or VoiceNLP. A crash, deadlock, or resource exhaustion in any other agent cannot prevent SafetyGuard from executing.

**Process isolation guarantees:**
- SafetyGuard is scheduled at real-time priority (Linux SCHED_FIFO) — it preempts all other processes
- SafetyGuard memory is locked in RAM (mlockall) — no paging delay
- SafetyGuard sensor polling runs at 10Hz regardless of system load; tested at 100% CPU utilization of all other cores

**Addresses:** H-001, H-002, H-003, H-004

### Control Layer 2: Dual-Sensor Confirmation

HV-STOP triggers when **either** sensor independently crosses its threshold. The system does not require both sensors to agree. This is a conservative (fail-safe) design.

**Sensor 1: OBD-II HV System Monitor**
- Polling: 10Hz via CAN bus
- Data: HV bus voltage, isolation resistance (IR), HV relay state, BMS fault codes
- Threshold: HV bus > 60V DC AND IR < 500 Ω (indicating compromised isolation), OR BMS fault code in HV_STOP_FAULT_LIST
- Failure mode: if OBD-II data stream drops > 500ms → warning shown to tech; SafetyGuard switches to thermal-only mode

**Sensor 2: Thermal IR Camera**
- Polling: 10Hz (30fps camera, sampled every 3rd frame)
- Data: thermal image of defined HV zone(s) per vehicle model (zone coordinates loaded from CAD data on session start)
- Threshold: human body heat signature detected within 0.5m of any defined HV zone
- Failure mode: if thermal camera feed drops > 500ms → warning shown to tech; SafetyGuard switches to OBD-II-only mode

**Dual-sensor rationale:** OBD-II alone would miss proximity to a powered-down but physically accessible HV component. Thermal alone would trigger on non-HV proximity. Together they provide both electrical state and physical proximity information.

**Addresses:** H-001, H-002 (proximity detection), H-003 (BMS fault codes detect pre-runaway)

### Control Layer 3: HV-STOP Protocol

When SafetyGuard fires HV-STOP:

1. **Immediate (<20ms):** Full-screen red overlay on all connected headsets. Text: "HV-STOP — Step back from vehicle. Do not touch any orange cables or components." All other UI elements hidden.
2. **Immediate (<20ms):** Vibration haptic on HoloLens 2 (USB-C HID command)
3. **< 30ms:** If collaborative session active, all participant headsets receive HV-STOP simultaneously
4. **< 40ms:** Append HV-STOP event to local audit log (append-only, tamper-evident)
5. **< 80ms:** Transmit HV-STOP event to cloud telemetry

**Resumption:** Session cannot resume until both steps complete:
- Tech says "HV isolated" (verbal acknowledgment, recorded in audit log)
- Tech performs confirmation action (head nod gesture or physical headset button press) — two-modality confirmation prevents accidental voice-only re-enable

**Rationale for two-step resumption:** A single voice command to dismiss an HV-STOP is too easy to trigger accidentally in a noisy environment. Two distinct modalities require deliberate intent.

**Override (emergency only):** A supervisor with manager dashboard access can remotely acknowledge HV-STOP if the tech is incapacitated and another person has physically confirmed isolation. This override requires a 6-digit PIN (set at account provisioning) and is logged to audit trail with supervisor ID, timestamp, and reason. **This event triggers an immediate alert to the compliance team.**

**Addresses:** H-001, H-002, H-003, H-004

### Control Layer 4: Zero LLM-Generated Repair Instructions

DiagnosticCore is architecturally prevented from generating repair step instructions at inference time. The repair step sequence is:

1. DTC code matched to OEM service bulletin via RAG retrieval
2. Matching bulletin's repair steps extracted (structured extraction, not generation)
3. Steps presented verbatim from OEM source, with TSB reference shown to tech

There is no code path in DiagnosticCore that calls an LLM to generate a repair step for a fault type that is not already indexed in the RAG corpus. If no matching bulletin is found, the system returns: "Fault type not enrolled — use manufacturer service manual procedure."

This control eliminates H-004 entirely for enrolled fault types: a software-invented repair step cannot inadvertently re-energize an HV system because all steps come from OEM-reviewed procedures.

**Tested:** Phase 2 exit criterion: "Zero LLM-generated repair instructions in output" — verified by code audit and output log review.

**Addresses:** H-004

### Control Layer 5: Connection Loss Safety Fallback

If the HoloLens 2 loses connection to the edge server (WiFi drop, edge server crash):

1. Headset detects connection loss within 500ms (WebSocket heartbeat timeout)
2. Headset immediately displays: "Connection lost — do not touch high-voltage components. Resume only after re-connecting."
3. Overlay freezes at last-known state (does not disappear — tech retains last visible safety zone reference)
4. Session resumes automatically if connection restores within 60 seconds (session state preserved on edge server)
5. If connection loss > 60 seconds: session terminated; tech must restart session with fresh HV check

**Addresses:** H-005

### Control Layer 6: Session Start HV Check

Every session begins with a mandatory HV pre-check step before any fault diagnosis proceeds:

1. "Is the vehicle HV system isolated? Confirm LOTO tags are attached."
2. Tech must confirm via voice + gesture
3. SensorFusion confirms: OBD-II shows HV relay open, HV bus < 60V
4. If sensor confirmation fails (HV bus > 60V) → HV-STOP fires immediately, session blocked

This step cannot be skipped. It is hardcoded, not configurable.

**Addresses:** H-001, H-004

---

## Safety Test Matrix Summary

| Test Scenario | Pass Criterion | Result (Phase 2) | Result (Phase 3 / 50 scenarios) |
|---|---|---|---|
| HV-STOP fires on thermal proximity | < 80ms activation | Pass | 50/50 scenarios pass |
| HV-STOP fires on OBD-II HV-live signal | < 80ms activation | Pass | 50/50 scenarios pass |
| HV-STOP fires when only one sensor available | < 80ms activation | Pass | All degraded-sensor scenarios pass |
| Session blocks if HV bus > 60V at start | Block before step 1 | Pass | All 10 test vehicles |
| Connection loss → safety warning < 500ms | < 500ms display | Pass | Tested across all WiFi failure modes |
| Collaborative session: all headsets get HV-STOP simultaneously | < 30ms delta between headsets | Pass (Phase 5 test) | — |
| Override logged to audit trail | Within 5 seconds | Pass | — |

Full test matrices: `docs/qa/hv_stop_safety_audit.md`

---

## Residual Risks

After all controls are applied:

| Risk ID | Residual Risk | Residual Probability | Acceptable? |
|---|---|---|---|
| H-001 | Direct contact if tech ignores HV-STOP alert | Very low (requires active defiance) | Yes — cannot eliminate willful non-compliance |
| H-002 | Proximity hazard during HV-STOP acknowledgment delay (< 80ms window) | Negligible | Yes — 80ms is below physiologically meaningful exposure threshold at typical automotive HV levels |
| H-003 | Thermal runaway from pre-existing TMS fault not detected by OBD-II | Very low (BMS fault codes are primary indicator) | Yes — pre-existing fault should have generated a DTC; SensorFusion cross-checks |
| H-004 | Step in OEM bulletin contains error that re-energizes HV (OEM data quality issue) | Very low | Yes — OEM bears responsibility for bulletin accuracy; indemnification clause in CAD licensing agreement |
| H-005 | Tech removes headset during HV work without notifying the system | Low | Yes — system cannot control tech behavior outside headset session; LOTO procedure is the final backstop |

**Overall residual risk assessment:** Acceptable for commercial deployment in franchised dealer service bays staffed by trained automotive technicians.

---

## Incident Reporting

Any safety incident during an X-Ray Vision Diagnostics session must be reported to:
1. Compliance Manager (internal) — within 1 hour
2. OSHA (if recordable) — within applicable OSHA reporting windows
3. Relevant OEM (if vehicle involved in recall or warranty investigation) — within 24 hours
4. SAE J1742 panel (if incident relates to HV-STOP failure) — within 48 hours

All HV-STOP audit logs are preserved for a minimum of 3 years (OSHA 1910.147 requirement) and are available to inspectors on request with 24-hour notice.
