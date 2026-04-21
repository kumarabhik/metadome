# Developer Unit Procurement — PO Approval and Process

**Phase:** 1
**Status:** Procurement spec + BOM complete (`docs/hardware/procurement_spec.md`); PO approved Month 2; units received Month 3

---

## Summary

10 HoloLens 2 developer units are required before internal prototyping can begin. The headset platform was selected as HoloLens 2 (see `docs/hardware/headset_comparison.md`). This document covers the purchase order process, finance approval path, vendor, and delivery timeline.

---

## Bill of Materials (Final, Finance-Approved)

| Item | Qty | Unit Price | Extended | Vendor |
|---|---|---|---|---|
| Microsoft HoloLens 2 Development Edition | 10 | $3,500 | $35,000 | Microsoft MSFT Direct (government/enterprise channel) |
| HoloLens 2 Carrying Cases (ruggedized) | 10 | $89 | $890 | Pelican Products |
| Cleaning kits (lens + foam brow pad) | 20 | $24 | $480 | Officeworks / Amazon Business |
| USB-C charging hubs (6-port, for bench charging) | 2 | $149 | $298 | Anker |
| HoloLens 2 developer accessories kit (clicker, calibration tool) | 10 | $99 | $990 | Microsoft Direct |
| Spare brow pads (hygiene — shared use in bay) | 50 | $18 | $900 | HoloLens accessories distributor |
| **Total Hardware** | | | **$38,558** | |
| Tax (8.875% NY) | | | $3,422 | |
| Shipping (2-day air, 10 units) | | | $280 | |
| **Total PO Amount** | | | **$42,260** | |

---

## Finance Approval Path

| Step | Owner | Deadline | Status |
|---|---|---|---|
| BOM submitted to Finance | Head of Engineering | Month 1, Week 3 | Complete |
| Budget line confirmed (R&D hardware budget) | VP Finance | Month 2, Week 1 | Approved — $50K allocated for Phase 1 hardware |
| PO raised in accounting system | Finance / AP | Month 2, Week 1 | PO #2024-HW-001 issued |
| PO sent to Microsoft Direct | Procurement | Month 2, Week 2 | Sent |
| Microsoft order confirmation | Microsoft | Month 2, Week 2 | Confirmed — order #MS-7843221 |
| Units shipped | Microsoft | Month 2, Week 4 | Shipped — tracking provided |
| Units received at office | Office Manager | Month 3, Week 1 | Received; serial numbers logged |

---

## Asset Register

All 10 units are logged in the company asset register on receipt:

| Serial # | Asset Tag | Assigned To | Location | Status |
|---|---|---|---|---|
| HL2-SN-001 | XVD-HW-001 | Engineering Lead | HQ Lab Bay | Active — prototype dev |
| HL2-SN-002 | XVD-HW-002 | Software Eng. 1 | HQ Lab Bay | Active — prototype dev |
| HL2-SN-003 | XVD-HW-003 | Software Eng. 2 | HQ Lab Bay | Active — prototype dev |
| HL2-SN-004 | XVD-HW-004 | UX Researcher | HQ Lab Bay | Active — WoZ study |
| HL2-SN-005 | XVD-HW-005 | ML Engineer | HQ Lab Bay | Active — VoiceNLP dev |
| HL2-SN-006 | XVD-HW-006 | QA Engineer | HQ Lab Bay | Active — safety testing |
| HL2-SN-007 | XVD-HW-007 | Spare / Pool | Storage | Spare — loaned for field visits |
| HL2-SN-008 | XVD-HW-008 | Spare / Pool | Storage | Spare — loaned for field visits |
| HL2-SN-009 | XVD-HW-009 | Product Manager | HQ Lab Bay | Active — demo unit |
| HL2-SN-010 | XVD-HW-010 | Field Research | Phase 1 Dealer Site | Active — contextual interviews |

---

## Handling and Care Procedures

- Headsets must be stored in Pelican cases when not in use
- Lens cleaning: only with provided microfiber cloth and lens cleaning solution — no paper towels
- Brow pad: replace between users in shared-use scenarios (hygiene requirement)
- Drops or physical damage: report immediately to Engineering Lead; unit pulled from use pending assessment
- Battery: charge after each use; do not store at <20% or >90% for extended periods (Li-ion best practice)
- Software updates: coordinated by Engineering Lead — do not accept OS updates without team coordination (OS update can break research mode API used by the recording feature)

---

## Phase 2 Hardware Needs (Forward Planning)

Phase 2 requires deploying to a test bay with a real vehicle. The 10 developer units are sufficient for internal prototyping. Phase 2 will not require additional headset units — the same pool rotates.

Phase 3 beta deployment (3 dealership bays) will require 6 additional production units (2 per bay) at full production pricing. This PO is addressed in the Phase 3 deployment spec (`docs/deployment/beta_site_install_spec.md`).
