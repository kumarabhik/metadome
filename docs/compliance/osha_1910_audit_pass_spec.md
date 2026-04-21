# OSHA 1910.147 Audit Pass Spec — 2 Beta Sites
## Step 44 — Phase 4, Certification 2

**Roadmap reference:** Phase 4, Certification 2
**Status:** `[ ]` Not started
**Owner:** Compliance Lead + Site Safety Officers + Outside Counsel
**Target:** OSHA audit passed at both beta sites by Month 17
**Prerequisite:** OSHA audit readiness package complete (`docs/compliance/osha_audit_readiness.md`); beta deployment live (`docs/deployment/beta_site_install_spec.md`)

---

## Objective

Successfully pass an OSHA 29 CFR 1910.147 (Control of Hazardous Energy — Lockout/Tagout) compliance audit at the two beta sites where X-Ray Vision Diagnostics is deployed in an EV service context. Passing this audit provides the regulatory validation required for commercial expansion and is a prerequisite for Phase 4 exit.

**What OSHA 1910.147 requires in our context:**
- Written energy control procedures for each EV model serviced
- Employee training on lockout/tagout procedures documented
- Periodic inspections (annual minimum) of energy control procedures
- Audit log records for each hazardous energy control event
- Employer certification that all requirements are met

**Audit type:** OSHA compliance audits can be voluntary (employer-initiated) or triggered by a complaint, incident, or industry sweep. This spec covers a voluntary audit conducted by a certified OSHA consultant — the recommended approach before any enforcement-triggered audit.

---

## Site Selection

| Site | OEM | Location | Rationale |
|---|---|---|---|
| Beta Site 1 | Toyota | [Metro market — state with active OSHA enforcement] | Longest-running beta site; most HV-STOP log data |
| Beta Site 2 | Ford | [Second metro market] | Cross-OEM coverage; Ford Mach-E HV system has different architecture than Toyota — tests OEM-agnostic compliance |

Both sites selected because they have ≥ 3 months of live system operation and ≥ 50 completed EV repairs — sufficient audit log volume to satisfy inspector review.

---

## Pre-Audit Documentation Package

Assembled by compliance lead 30 days before audit date. Each item maps to a specific 1910.147 requirement.

| 1910.147 Requirement | Our Documentation | Location |
|---|---|---|
| §(c)(4)(i) — Written energy control procedures | Per-model lockout/tagout procedures (Toyota bZ4X, Ford Mach-E) | `docs/compliance/osha_audit_readiness.md` §4 |
| §(c)(7) — Employee training records | Signed training acknowledgment forms from all technicians enrolled in onboarding | `docs/deployment/onboarding_program_spec.md` Appendix |
| §(c)(6) — Periodic inspection records | Internal inspection log (at least one formal inspection per site per standard) | Generated from HV-STOP audit log — see below |
| §(d) — Energy control procedure application | HV-STOP event logs with: tech ID, VIN, timestamp, action taken, resolution | SafetyGuard audit log (OSHA format, per `docs/compliance/osha_audit_readiness.md`) |
| §(f)(1) — Certification by employer | Employer certification letter signed by site Safety Officer and GM | Drafted by outside counsel; signed pre-audit |

### HV-STOP Audit Log — OSHA Format

The SafetyGuard agent's audit log was redesigned to OSHA 1910.147 required fields in Phase 3 (`docs/compliance/osha_audit_readiness.md`). Each log entry includes:
- Machine/equipment identifier (VIN + model)
- Employee performing service (tech ID, name)
- Authorized employee who applied lockout/tagout (same tech, or site Safety Officer)
- Date of energy control application
- Means used to isolate HV energy (service disconnect activated, discharge confirmed)
- Verification that energy was isolated and de-energized (voltage reading from CAN bus pre-work confirmation)

**Log volume target:** Each site should have ≥ 25 HV-STOP log entries before audit — provides statistically meaningful sample for inspector review.

---

## Audit Day Procedure

### Inspector Access

| Access Request | What We Provide |
|---|---|
| Bay walkthrough | Service manager escorts inspector through enrolled service bays |
| Live system demonstration | 15-minute demo of HV-STOP protocol on enrolled vehicle (Toyota or Ford per site) |
| Audit log review | Digital export of SafetyGuard logs for prior 90 days; printable format for inspector |
| Training record review | Technician training acknowledgment forms; onboarding completion certificates |
| Written procedures | Printed copies of per-model lockout/tagout procedures posted in service bays |

### Demo Script — HV-STOP Live Demonstration (15 minutes)

1. Technician activates headset and begins simulated diagnostic session on enrolled vehicle
2. Technician moves hand toward HV bus zone (300mm boundary) → amber ring and audio tone demonstrated
3. Technician moves hand to 150mm boundary → HV-STOP triggers: red overlay, alarm, voice alert ("HV STOP — High Voltage Hazard Detected")
4. System rendered in stopped state; manager dashboard notification demonstrated
5. Safety Officer performs remote HV-STOP clearance → system resumes
6. Compliance lead narrates each step against 1910.147 requirements during demo

### Inspector Questions — Prepared Answers

| Likely Inspector Question | Prepared Answer |
|---|---|
| "How do you know the HV bus is de-energized before the technician proceeds?" | "SensorFusion agent reads live CAN bus HV voltage; HV-STOP does not clear until voltage reading confirms bus is below 60V. Discharge verification is logged." |
| "What if the system fails — does the tech still have a safe fallback?" | "SafetyGuard runs as an independent real-time process. If the main diagnostic pipeline fails, SafetyGuard remains active. The written lockout/tagout procedure posted in the bay is the fallback — it does not depend on the headset system." |
| "Are all technicians trained on the written procedure, not just the headset system?" | "Yes — onboarding program Module 2 covers written lockout/tagout procedure. Signed acknowledgment is in the training record we're providing." |
| "What happens if a technician removes the headset mid-repair?" | "The system enters a graceful degradation state. SafetyGuard cannot enforce proximity alerts without the depth camera. Posted procedure requires tech to follow written lockout/tagout when headset is off." |

---

## Expected Findings — Risk Assessment

Based on Phase 3 OSHA readiness review, anticipated findings and remediation readiness:

| Risk Area | Likelihood | Pre-remediated? | Response if Cited |
|---|---|---|---|
| Training record completeness — missing signatures | Medium | Partially | Collect any missing signatures before audit date; add to onboarding checklist |
| Written procedure not posted in bay | Low | Yes | Procedures printed and laminated in each bay at install |
| HV-STOP log missing required field | Low | Yes | Log format redesigned to OSHA spec in Phase 3; all required fields present |
| Technician unable to verbally explain lockout/tagout procedure | Medium | Partially | Pre-audit refresher session with all enrolled technicians 1 week before audit |
| Energy isolation verification not documented for past repairs | Medium | No | Cannot retroactively fix historical logs; demonstrate current log format is compliant; accept finding for historical records only |

**Priority remediation before audit date:**
1. Audit all training records for missing signatures — chase down any gaps
2. Verify printed procedures are posted, laminated, and in current revision in all enrolled bays
3. Run pre-audit technician refresher (30 min per site) focused on verbal procedure explanation

---

## Post-Audit Actions

### If Audit Passes (No Citations)

1. Compliance lead documents audit outcome: date, inspector name, OSHA area office, result
2. Filed in `docs/compliance/` as official audit record
3. Incorporated into Phase 4 sales materials: "OSHA 1910.147 compliant — audit passed at [Site 1 name] and [Site 2 name], [Month/Year]"
4. Annual re-audit scheduled (required by §(c)(6) periodic inspection requirement)
5. GA expansion sites (Phase 4, 50 bays) follow same pre-audit checklist from this spec

### If Citations Issued

OSHA citations are issued with a "gravity-based penalty" — severity determines response urgency:

| Citation Type | Response Requirement | Our SLA |
|---|---|---|
| Other-Than-Serious | Abate by citation deadline (typically 30–90 days) | Remediate within 14 days; close citation early |
| Serious | Abate by citation deadline; potential fine | Immediate engineering sprint; outside counsel engagement |
| Willful or Repeat | Highest fines; potential business impact | Escalate to CEO + board; pause new deployments until resolved |

**If any Serious or higher citation is issued:** Phase 4 commercial expansion is paused until citation abated and compliance re-verified. No new customer deployments during open citation period.

---

## Annual Inspection Cadence (Post-Pass)

Per 1910.147 §(c)(6), energy control procedures must be inspected at least annually:
- Inspection performed by site Safety Officer + compliance lead
- Covers: procedure review, log spot-check, technician interview (verbal procedure knowledge)
- Results documented in OSHA inspection log
- Any procedure changes (new vehicle model, new fault type) trigger interim inspection within 30 days of change

---

## Links to Prerequisite Docs

- OSHA audit readiness package: `docs/compliance/osha_audit_readiness.md`
- Onboarding program (training records): `docs/deployment/onboarding_program_spec.md`
- SafetyGuard HV-STOP audit log format: `docs/agents/safety_guard_v1.md`
- SAE J1742 certification (parallel compliance track): `docs/compliance/sae_j1742_certification_spec.md`
- HV-STOP safety audit (50-scenario test data): `docs/qa/hv_stop_safety_audit.md`
