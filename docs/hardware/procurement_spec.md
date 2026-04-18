# Hardware Procurement Specification
## Step 4 — Developer Unit Procurement Plan

**Roadmap reference:** Phase 1, Deliverable 4
**Status:** `[~]` Spec complete; PO pending finance approval
**Owner:** Engineering Lead + Procurement
**Target delivery:** End of Month 2

---

## Phase 1 Bill of Materials (10 Developer Units)

### Headsets

| Item | Model | Qty | Unit Price | Total | Vendor |
|---|---|---|---|---|---|
| XR Headset | Microsoft HoloLens 2 Development Edition | 10 | $3,500 | $35,000 | Microsoft Store for Business |
| Carrying case | Pelican 1510 Carry-On (w/ foam insert) | 10 | $120 | $1,200 | Amazon Business |
| Cleaning kit | Zeiss lens wipes + UV sanitizer | 10 sets | $45 | $450 | Fisher Scientific |

### Edge Server (1 per test bay; 2 bays for Phase 1)

| Item | Model | Qty | Unit Price | Total | Vendor |
|---|---|---|---|---|---|
| Edge compute unit | NVIDIA Jetson AGX Orin 64GB | 2 | $1,999 | $3,998 | Arrow Electronics |
| Storage (CAD cache) | Samsung 2TB NVMe SSD | 4 (2 per unit) | $180 | $720 | CDW |
| Enclosure | Pelican iM2950 rack mount | 2 | $350 | $700 | Pelican |
| UPS (power protection) | APC Back-UPS Pro 1500VA | 2 | $220 | $440 | CDW |

### Network (Bay WiFi6E — 2 bays)

| Item | Model | Qty | Unit Price | Total | Vendor |
|---|---|---|---|---|---|
| WiFi6E Access Point | Cisco Catalyst 9136 | 2 | $1,400 | $2,800 | CDW |
| PoE Switch | Cisco SG350-10P | 2 | $450 | $900 | CDW |
| Ethernet cable (Cat6A, 50ft runs) | Monoprice | 20 | $25 | $500 | Monoprice |

### OBD-II / CAN Bus Test Hardware

| Item | Model | Qty | Unit Price | Total | Vendor |
|---|---|---|---|---|---|
| OBD-II interface | Kvaser Leaf Light HS v2 | 4 | $295 | $1,180 | Kvaser |
| CAN bus analyzer | Peak PCAN-USB Pro | 2 | $480 | $960 | Peak-System |
| OBD-II breakout cable set | autonerdz.com kit | 4 | $85 | $340 | autonerdz.com |

### Thermal Camera (SensorFusion integration testing)

| Item | Model | Qty | Unit Price | Total | Vendor |
|---|---|---|---|---|---|
| Thermal camera | FLIR A50 (USB, 14mm lens) | 2 | $3,200 | $6,400 | FLIR / Teledyne |
| Mounting arm | Manfrotto 244 friction arm | 2 | $95 | $190 | B&H Photo |

### Test Vehicle (if not provided by dealer partner)

| Item | Notes | Cost | Source |
|---|---|---|---|
| 2022–2024 Toyota bZ4X | Must be EVA-platform, battery TMS accessible | Lease 1 unit at ~$850/month for 6 months | Toyota Fleet |
| Vehicle lift (2-post) | Required for undercarriage CAD validation | $4,800 (one-time) | BendPak |

---

## Total Phase 1 Hardware Budget

| Category | Cost |
|---|---|
| Headsets | $36,650 |
| Edge servers | $5,858 |
| Network | $4,200 |
| OBD-II / CAN test hardware | $2,480 |
| Thermal cameras | $6,590 |
| Test vehicle (6-month lease) | $5,100 |
| Contingency (10%) | $6,088 |
| **Total** | **$66,966** |

---

## Procurement Timeline

| Week | Action | Owner |
|---|---|---|
| Week 1 | Submit PO request to finance; get vendor quotes | Procurement |
| Week 2 | Finance approval; place orders | Finance + Procurement |
| Week 3 | Headsets delivered (Microsoft 2-week lead time) | Engineering |
| Week 4 | Edge servers + network hardware delivered | Engineering |
| Week 5 | Bay installation: network cabling, edge server rack, headset charging station | Facilities + Engineering |
| Week 6 | OBD-II / thermal camera integration testing begins | Engineering |

---

## Device Management Plan

- All 10 headsets enrolled in **Microsoft Intune** (MDM)
- Headsets assigned to bay rather than individual — check-out log maintained in Notion
- Firmware updates tested on 1 unit before rolling to all 10 (prevent bricking active units)
- Each headset labeled with bay number + asset tag for tracking
- Repair/replacement: Microsoft Complete for Business coverage ($299/device/year) — covers 1 accidental damage incident per device

---

## Phase 3 Hardware Projection (50 bays)

| Item | Unit Cost | Qty | Total |
|---|---|---|---|
| HoloLens 2 (or successor) | $3,200 (volume pricing) | 75 | $240,000 |
| Edge server per bay | $2,200 (volume) | 50 | $110,000 |
| Network per bay | $2,000 | 50 | $100,000 |
| **Estimated Phase 3 hardware** | | | **~$450,000** |

Amortized over 3-year contracts at $18,000/bay/year → hardware is recovered in Month 4 of each bay's contract.
