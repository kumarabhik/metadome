# OEM CAD Data Licensing — Term Sheet

**Phase:** 1
**Status:** Framework complete; signatures targeted Month 3–4

---

## Overview

This term sheet governs the licensing of OEM-provided CAD data (3D mesh files, assembly diagrams, component position data, wiring schematics) to X-Ray Vision Diagnostics for use in the spatial overlay platform. Each OEM signs a separate agreement with identical structure but OEM-specific schedule.

Three target signatories at Phase 1 exit: Toyota Motor North America, Ford Motor Company, Stellantis North America. Minimum to meet Phase 1 exit criteria: 2 of 3 signed.

---

## Term Sheet Structure

### 1. Licensed Data

**Included:**
- 3D CAD mesh files for all enrolled vehicle models (VIN-range scoped, not fleet-wide)
- Component position and orientation data (spatial coordinates within vehicle frame)
- Wiring harness schematics (high-voltage systems only for Phase 1–2; all systems by Phase 3)
- OEM technical service bulletins (TSBs) for enrolled fault types
- DTC code definitions and OEM-recommended repair procedures

**Excluded (not licensed):**
- Manufacturing process data
- Supply chain / vendor data
- Future model pre-production CAD (unless separately agreed)
- Any data beyond enrolled vehicle models and fault types

### 2. License Grant

- **Scope:** Non-exclusive, non-sublicensable license to use Licensed Data solely for: (a) rendering spatial overlays on the licensed vehicle models in X-Ray Vision Diagnostics headset sessions; (b) training and operating AI models (DiagnosticCore RAG pipeline) for diagnostic recommendations on enrolled fault types
- **Field of use restriction:** Licensed Data may not be used for: competitive product development; resale or redistribution; training AI models for non-automotive applications
- **Territory:** United States (Phase 1–4); extended to Canada and UK by separate schedule (Phase 5)
- **Sublicensing:** Prohibited. X-Ray Vision Diagnostics may not sublicense OEM data to third parties.

### 3. Fees

| Structure | Amount | Timing |
|---|---|---|
| Annual data access fee | $150,000 / OEM / year | Invoiced at signing, then annually |
| Per-model addition fee | $25,000 / new vehicle model added | Invoiced on model addition |
| Minimum term | 3 years | Terminates with 90-day notice after Year 3 |

Fee rationale: OEM data licensing in automotive aftermarket (Mitchell1, Snap-on ALLDATA) is typically $50–200K/year for diagnostic data access. We set $150K/year as a reasonable entry point given Phase 1 scope (limited models). At Phase 3+, renegotiate to a per-VIN-activated royalty model as volume justifies it.

### 4. Data Security and Handling

- All Licensed Data stored encrypted at rest (AES-256) and in transit (TLS 1.3)
- Licensed Data never leaves X-Ray Vision Diagnostics infrastructure — no transmission to third parties, end users, or cloud services outside our control
- OEM has right to audit data handling practices annually (30-day notice required)
- On termination: Licensed Data purged from all systems within 30 days; deletion certificate provided to OEM

### 5. Accuracy and Liability

- OEM represents that Licensed Data is accurate as of the date of delivery
- X-Ray Vision Diagnostics is solely responsible for how Licensed Data is used in the product
- **OEM is not liable for incorrect repair outcomes resulting from technician error or product failure** — this is critical: OEM must not accept liability for warranty claims arising from use of their data in our product
- X-Ray Vision Diagnostics indemnifies OEM against third-party claims arising from use of Licensed Data in the platform

### 6. Update Obligations

- OEM provides updated TSBs and revised CAD data within 30 days of publication / revision
- Update delivery: secure SFTP or OEM-preferred secure file transfer
- X-Ray Vision Diagnostics ingests updates within 7 days of receipt

### 7. Intellectual Property

- OEM retains all IP in Licensed Data
- X-Ray Vision Diagnostics retains all IP in the platform, including AI models trained using Licensed Data
- AI models trained using Licensed Data are not considered derivative works of the Licensed Data (same principle as a mechanic learning from a repair manual)

### 8. Term and Termination

- Initial term: 3 years from execution date
- Auto-renews annually unless either party provides 90-day written notice
- OEM may terminate immediately if: Licensed Data used outside field of use; security breach involving Licensed Data; insolvency of X-Ray Vision Diagnostics
- X-Ray Vision Diagnostics may terminate immediately if OEM fails to deliver agreed data updates for > 60 days

---

## OEM-Specific Schedules

### Schedule A: Toyota Motor North America
- **Point of contact:** Toyota Global Service Operations, Manager of Dealer Technical Programs
- **Enrolled vehicles (Phase 1):** Toyota bZ4X (2023–2025 model years)
- **Enrolled fault types:** EV battery TMS (coolant leak, cell fault), hybrid inverter, brake system
- **Data format:** Toyota-proprietary .stp and .jt formats → normalized by our ingestion pipeline
- **Delivery method:** Toyota Partner Portal (secure SFTP)
- **Special term:** Toyota receives monthly aggregate FTFR data for enrolled vehicle models (no PII, no dealer identification)
- **Target execution:** Month 3

### Schedule B: Ford Motor Company
- **Point of contact:** Ford Service Engineering, Connected Vehicle Services
- **Enrolled vehicles (Phase 1):** Ford Mustang Mach-E (2022–2025 model years)
- **Enrolled fault types:** EV battery TMS, ICE timing chain (Mach-E ICE variant), brake system
- **Data format:** Ford uses Siemens NX format → normalized by ingestion pipeline
- **Delivery method:** Ford Supplier Portal
- **Special term:** Ford receives quarterly FTFR improvement report segmented by model
- **Target execution:** Month 3

### Schedule C: Stellantis North America
- **Point of contact:** Stellantis Dealer Technical Operations
- **Enrolled vehicles (Phase 1):** Jeep Grand Cherokee 4xe (2023–2025), Ram 1500 (conventional — Phase 1 non-EV testing)
- **Enrolled fault types:** Hybrid battery TMS (4xe), ICE diagnostic baseline (Ram)
- **Data format:** Stellantis uses CATIA V5 format → normalized by ingestion pipeline
- **Special note:** Stellantis deal is opportunistic at Phase 1; Toyota and Ford are the minimum. Stellantis execution may slip to Phase 2.
- **Target execution:** Month 4 (or Phase 2 if delayed)

---

## Signature Status Tracker

| OEM | Term Sheet Sent | OEM Legal Review | Redlines Received | Executed |
|---|---|---|---|---|
| Toyota | Month 1 | Month 2 | Month 2 | Month 3 (target) |
| Ford | Month 1 | Month 2 | Month 3 | Month 3 (target) |
| Stellantis | Month 2 | Month 3 | Month 3 | Month 4 (target) |

Phase 1 exit criterion met when: Toyota **and** Ford executed (2 of 3 minimum). Stellantis execution is a Phase 1 stretch goal.
