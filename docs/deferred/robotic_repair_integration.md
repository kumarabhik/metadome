# Deferred: Robotic Repair Integration

**Status:** Deferred — 5+ year horizon
**Earliest viable horizon:** 2031+
**Decision date:** Phase 0 product definition

---

## What It Is

Actuator guidance from the spatial overlay: robotic arms or end-effectors physically perform repair steps that the technician currently executes manually, guided by the CAD overlay's spatial coordinates. The technician supervises; the robot executes.

Example: "Remove coolant pump fasteners" — instead of the tech using a torque wrench guided by the overlay, a robotic arm with a torque wrench attachment removes the fasteners at the precise location and torque specified by the CAD data.

---

## Why It's Deferred

### 1. No Viable Service Bay Robotics Platform Exists

The automotive service bay is an unstructured environment. Service robots exist in manufacturing (fixed positions, structured environments, known part tolerances). Service bay robotics requires:
- Mobile manipulation in a variable, human-occupied space
- Precise force control on fasteners that vary in condition (corroded, over-torqued, damaged)
- Handling of components that are not in their manufactured position (worn, misaligned, damaged)

No commercial platform as of 2026 handles all three at service bay fidelity. Boston Dynamics Spot can navigate a bay; it cannot torque a fastener on a corroded EV battery module. The manipulation capabilities needed are 5–10 years from commercialization at the required precision level.

### 2. HV Interaction by Robot Is an Unsolved Regulatory Problem

SAE J1742 and OSHA 1910.147 both assume a human technician performing lockout/tagout. There is no regulatory framework for a robot performing HV isolation procedures. A robot that disconnects an HV connector incorrectly causes the same electrocution/fire risk as a human doing it incorrectly — but the liability attribution is entirely unclear (robot manufacturer? software provider? dealer?).

Until regulators define a framework for automated HV service, any robotic integration with HV components is commercially uninsurable.

### 3. OEM Warranty Implications Are Unresolved

OEM warranties cover repairs performed by certified technicians following OEM procedures. A robot-performed repair almost certainly voids the OEM warranty under current terms — OEM agreements do not contemplate robotic execution of repair steps. Renegotiating warranty terms for robot-assisted repair with three OEMs is a multi-year process.

---

## What Would Change This

| Condition | Estimated Timeline |
|---|---|
| Commercial mobile manipulation platform at service-bay precision | 2029–2032 |
| OSHA/SAE regulatory framework for automated HV service | 2031–2035 |
| OEM warranty terms updated to include robot-assisted repair | 2030–2033 |
| Robot cost below $150K/bay (economic threshold for dealer ROI) | 2030–2035 |

**Earliest credible pilot:** 2031–2032, limited to non-HV repair steps on a specific vehicle model with OEM co-development agreement and waived warranty clause.

---

## How Our Platform Positions for This Future

Even if robotic execution is 10 years away, our spatial overlay is the natural guidance layer for any future service bay robot. The CAD overlay already encodes:
- Precise 3D coordinates of every component
- Torque specifications per fastener
- Sequence and orientation constraints for each step

A robotic arm that can read our LayerManifest format has all the spatial information it needs to execute a repair step. We are not building the robot — we are building the data infrastructure that any future robot will depend on.

**Year 3–4 action:** Establish a robotics partnership with one industrial robot manufacturer (candidate: Universal Robots, FANUC, or a purpose-built automotive service robotics startup) to co-develop the LayerManifest extension for robotic execution. This positions us as the coordination layer when the hardware is ready — without building hardware ourselves.
