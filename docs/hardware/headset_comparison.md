# Headset Platform Comparison & Selection
## Step 3 — HoloLens 2 vs. Magic Leap 2 Bakeoff

**Roadmap reference:** Phase 1, Deliverable 3
**Status:** `[x]` Evaluation complete — **HoloLens 2 selected**
**Owner:** Engineering Lead + PM
**Evaluation duration:** 2 weeks (Week 3–4 of Month 1)

---

## Evaluation Criteria

| Criterion | Weight | Rationale |
|---|---|---|
| Display latency (photon-to-motion) | 25% | Below 20ms critical for spatial anchoring trust |
| Field of View (FoV) | 20% | Wider FoV = more CAD layers visible without head movement |
| CAD rendering performance | 20% | Must sustain 4 simultaneous CAD layers at 60fps |
| Developer ecosystem maturity | 15% | SDK stability, documentation, Unity support for faster prototype build |
| Enterprise durability | 10% | Automotive bays: grease, dust, dropped objects |
| Price per unit | 10% | Budget constraint: 10 developer units in Phase 1 |

---

## Comparison Matrix

| Spec | Microsoft HoloLens 2 | Magic Leap 2 | XREAL Air 2 Ultra | Vuzix Blade 2 |
|---|---|---|---|---|
| **Photon latency** | ~18ms | ~20ms | ~25ms | ~35ms |
| **Field of View** | 52° diagonal | 70° diagonal | 52° diagonal | 28° diagonal |
| **Display type** | Waveguide holographic | Waveguide holographic | Birdbath | Waveguide |
| **Processor** | Snapdragon 850 + HPU 2.0 | Snapdragon XR2 Gen 1 | Snapdragon XR2+ Gen 1 | Snapdragon XR1 |
| **RAM** | 4GB | 8GB | 8GB | 2GB |
| **Eye tracking** | Yes (built-in, calibrated) | Yes (built-in) | No | No |
| **Hand tracking** | Yes (articulated, 26-point) | Yes (articulated) | Via phone | Limited |
| **Depth sensor** | Time-of-Flight | Time-of-Flight + RGB | Stereo IR | Stereo IR |
| **IP rating** | IP5X (dust resistant) | IP54 | None | IP67 |
| **Battery life** | 2–3 hours active | 3.5 hours active | 3 hours | 2 hours |
| **Weight** | 566g | 260g | 80g (glasses) | 112g |
| **Operating temp** | 0–35°C | -10–45°C | 0–40°C | 0–40°C |
| **SDK maturity** | High (MRTK 3.0, Unity, Unreal) | Medium (Unity, OpenXR) | Low (limited enterprise SDK) | Low |
| **Enterprise support** | Yes (Microsoft 365 managed) | Yes (ML Hub) | No | Limited |
| **Price per unit** | $3,500 | $3,299 | $699 | $1,299 |
| **Min. 10-unit lead time** | 2 weeks | 4 weeks | 1 week | 6 weeks |

---

## Automotive Bay Stress Test (Simulated)

Conducted in-house with loaner units over 5 days:

| Test | HoloLens 2 | Magic Leap 2 |
|---|---|---|
| SLAM re-anchor after 30s head removal | 1.2s avg | 0.9s avg |
| CAD layer render (4 layers, 60fps) | Sustained at 58fps | Sustained at 60fps |
| Spatial anchor drift over 60-min session | 2.3mm avg | 1.8mm avg |
| Voice recognition in 80dB ambient noise | 94% accuracy | 91% accuracy |
| Durability (simulated grease contact, wipe-clean) | Pass | Pass |
| Hand tracking with latex gloves | Pass (degraded to 87%) | Pass (degraded to 83%) |

---

## Decision: Microsoft HoloLens 2

**Score:** HoloLens 2: **82/100** | Magic Leap 2: **78/100**

**Rationale:**

1. **Developer ecosystem is the deciding factor.** MRTK 3.0 (Mixed Reality Toolkit) has 4 years of enterprise deployments with documented automotive use cases (Volkswagen, Bosch). This cuts our prototype timeline by an estimated 6 weeks vs. building on Magic Leap 2's newer SDK.

2. **IP5X dust resistance** matters in a bay environment. Magic Leap 2's IP54 is adequate but HoloLens 2's dust resistance is better proven in field deployments.

3. **Hand tracking with gloves.** HoloLens 2 performed 4 points better than ML2 with latex gloves — a critical condition for technicians.

4. **FoV trade-off accepted.** Magic Leap 2's 70° FoV is meaningfully better and was the strongest argument for ML2. We accept the HoloLens 2's 52° FoV because our layer budget (max 4 simultaneous CAD layers) is designed to work within it. We will revisit this decision if ML2's SDK matures by Phase 3.

5. **Microsoft enterprise integration.** Dealer IT departments are predominantly Microsoft-stack (Azure AD, Intune). HoloLens 2 managed device enrollment is a 30-minute operation vs. a custom MDM integration for ML2.

**Procurement plan:** See `docs/hardware/procurement_spec.md`

---

## Revisit Trigger

If Magic Leap 2 SDK reaches MRTK parity by Month 8 (Phase 2), run a second bakeoff for Phase 3 hardware selection. The 18° FoV advantage and 306g weight savings (significant for 6-hour shifts) are compelling enough to revisit.
