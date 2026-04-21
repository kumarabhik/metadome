# Year 3–5 Product Vision

**Horizon:** Months 31–60 (following Phase 5 completion at Month 30)
**Classification:** Strategic planning document — internal use, investor-facing

---

## Where We Are at the End of Phase 5

By Month 30, X-Ray Vision Diagnostics is:
- 500+ bays live across US, Canada, UK
- FTFR ≥ 90% demonstrated at scale
- OEM embedded deal in active negotiation (Toyota or Ford)
- SOC 2 Type II certified
- $1.66M ARR, growing toward cash-flow positive at ~600 bays

The Phase 5 platform is excellent at one thing: helping a trained technician diagnose and fix a known fault type on an enrolled vehicle, the first time, in a service bay.

Year 3–5 expands what "known fault type," "enrolled vehicle," and "service bay" mean — ultimately building toward a platform where the system and the technician are genuine collaborators, not tool-and-user.

---

## Vision Statement

**Year 5 vision:** X-Ray Vision Diagnostics is the diagnostic intelligence layer embedded in every major OEM's dealer tooling contract globally — the platform that transforms the franchised dealership service bay from the slowest, most error-prone part of the automotive ownership experience into its most reliable.

The North Star metric remains FTFR, but the target evolves: ≥ 97% FTFR on enrolled fault types (from 90% at Phase 5), and the enrolled fault list covers 90% of all repair orders by volume (from 5 fault types at Phase 1).

---

## Strategic Bets for Year 3–5

### Bet 1: Predictive Maintenance Becomes the Primary Entry Point

**Phase 5:** Predictive maintenance is a feature — optional, opt-in, surfaces alerts when the tech is already at the vehicle.

**Year 3–5 vision:** Predictive maintenance becomes the reason dealers keep cars longer in the network. A dealer whose vehicles are enrolled in X-Ray Vision Diagnostics telemetry monitoring can proactively schedule service before faults occur. The product transitions from "diagnostic tool used when something is already wrong" to "fleet health intelligence used every time a vehicle is in range."

This requires:
- OEM telematics agreements extended to include all enrolled VINs, not just vehicles currently in for service
- A dealer-facing "fleet health dashboard" (separate from the service bay manager dashboard) showing all enrolled vehicles and their drift scores
- Integration with the dealer's service scheduling system (Xtime, DealerSocket) to auto-create service appointments for high-confidence predictive alerts

**Business model impact:** Fleet health dashboard = new product tier ("Fleet Intelligence") at $6,000/year/dealer regardless of bay count. This changes the revenue model from per-bay to per-dealer — dramatically increases LTV for large dealer groups.

---

### Bet 2: OEM Embedded Becomes the Primary Distribution Channel

**Phase 5:** OEM embedded is a deal in negotiation — one OEM, one contract.

**Year 3–5 vision:** 3 OEM embedded deals active (Toyota, Ford, plus one of Stellantis / Hyundai / GM). X-Ray Vision Diagnostics appears as a line item in every franchised dealer's OEM tooling invoice, similar to how diagnostic software (TechStream, FDRS) is already bundled. The direct sales team shrinks; most bays are acquired through OEM channel at near-zero CAC.

This is the fundamental business model inflection. At 3 OEM embedded deals × ~3,000 enrolled dealer bays each = 9,000 bays at $2,400 wholesale = $21.6M ARR from embedded alone. This alone justifies the valuation thesis.

---

### Bet 3: The Platform Becomes the Technician's Career Record

**Phase 5:** Apprentice Training Mode scores feed OEM LMS — a point-in-time certification.

**Year 3–5 vision:** Every technician who uses X-Ray Vision Diagnostics for 3+ years has an objective, longitudinal record of their diagnostic competency — a "skills passport" that follows them across dealerships. FTFR by fault type, MTTD improvement over time, checkpoint accuracy, voice command efficiency. This is the most complete technician performance record that has ever existed.

Implications:
- **For technicians:** Portable proof of competency; negotiating leverage at hire; OEM certification credit without a formal test
- **For dealers:** Objective hiring and promotion data; no more relying on resume claims
- **For OEMs:** Quality data on the technician workforce that was previously invisible
- **For us:** Creates lock-in that no competitor can replicate — techs don't want to switch to a tool that loses their history

Product required: technician-owned "Skills Profile" — portable, tech controls who can see it, can be shared with prospective employers. Opt-in export to LinkedIn or OEM HR systems.

---

### Bet 4: Expand from Franchised Dealer to Independent Repair Shops

**Phases 1–5:** 100% franchised dealer focus. OEM CAD data requires OEM partnership. Safety certification required OEM cooperation.

**Year 3–5 opportunity:** The independent repair market is 5× larger than franchised dealers — 75,000 independent shops vs. 17,000 franchised dealers. EV repair at independents is growing as older EVs age out of warranty.

**Barrier:** Independent shops cannot access OEM CAD data under current licensing agreements (OEM agreements are franchise-only). Three paths to unlock:

1. **Negotiate "independent shop" tier in OEM licensing agreements:** OEM licenses to us for franchise + licensed independent network. Precedent: Mitchell1's ALLDATA has OEM-licensed data for independents. This is a 12–18 month negotiation, not a technical problem.

2. **Aftermarket CAD data aggregators:** Companies like Lucid Motors, Rivian, and some OEMs have published open CAD data for community use. Build an "open CAD" tier that works with publicly available data — covers a small but growing subset of vehicles.

3. **Wait:** As EV out-of-warranty volumes grow (2028+), OEM motivation to restrict independent shop access weakens. They need independents to service vehicles they can't reach. Revisit in Year 4.

**Year 5 target:** 200 independent shop bays enrolled under a new "Independent" licensing tier with a subset of enrolled vehicles and fault types.

---

### Bet 5: From Hands-Free Diagnostics to Hands-Free Repair Guidance

**Phase 5:** The overlay guides diagnosis and prescribes steps. The technician executes manually; the system does not know whether the tech executed correctly.

**Year 3–5 vision:** Computer vision on the headset camera validates physical repair actions. "Is the bolt torqued correctly?" — the system analyzes the tech's hand position and the torque wrench indicator. "Is the component seated correctly?" — the system compares the installed position against the CAD model. Step validation moves from "tech says Done" to "system confirms Done."

This is the feature that pushes FTFR from 91% toward 97%: not only is the diagnosis correct, but the execution is verified.

**Technical requirements:**
- HoloLens 2 depth camera (already present) used for component position verification
- Fine-grained object detection model for automotive components (torque wrench, coolant line connector, bolt head)
- Per-fault-type verification rules: "torque wrench angle must be 90° ± 5° when this bolt is tightened"

**Earliest viable milestone:** Year 4 for 1–2 specific high-value verification steps per fault type. Full execution validation across all steps is a Year 5+ problem.

---

## Year 3–5 Metrics Targets

| Metric | Phase 5 End | Year 3 | Year 4 | Year 5 |
|---|---|---|---|---|
| Bays | 500 | 2,000 | 5,000 | 10,000 |
| ARR | $1.66M | $7.2M | $17M | $34M |
| Gross margin | 24% | 38% | 45% | 52% |
| FTFR (enrolled fault types) | 91.8% | 93% | 95% | 97% |
| Enrolled fault types | 5 | 20 | 50 | 100 |
| OEM embedded deals | 1 (negotiating) | 2 | 3 | 4 |
| Markets | US, CA, UK | + Germany | + Japan | + South Korea, Australia |
| Tech skills profiles | 0 | 5,000 | 15,000 | 35,000 |
| NPS (technicians) | 54 | 58 | 62 | 65 |

---

## What We Won't Do in Year 3–5

**Robotic repair integration:** Actuator guidance from spatial overlay — still a 5+ year horizon. The liability framework for robot-assisted HV repair does not exist yet.

**Consumer / home mechanic version:** A different product with a different safety framework and licensing model. Not in scope.

**Fully autonomous diagnosis (no tech in loop):** Regulatory and liability barriers not resolved. OSHA 1910.147 requires a human technician to perform LOTO. This is not viable pre-2030.

**Moving off the Edge:** The 40ms real-time render requirement cannot be met from the cloud. The edge-first architecture is permanent, not a transitional choice.

---

## The 10-Year Outcome

If the Year 3–5 bets land:
- X-Ray Vision Diagnostics is in every major OEM's dealer tooling contract globally
- 50,000+ bays worldwide
- Every EV repair attempted at an enrolled dealership has a > 95% chance of being fixed correctly the first time
- The technician shortage at dealerships is no longer a First-Time Fix Rate problem — it's still a headcount problem, but the headcount that exists is dramatically more effective
- The platform has generated the largest longitudinal dataset of EV repair outcomes in the world — a data asset whose value exceeds the software product itself

The ultimate form of the product is not a diagnostic tool. It is the intelligence layer that makes the global dealer service network as reliable as the vehicles it services.
