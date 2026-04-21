# Deferred: Consumer / At-Home Diagnostic Version

**Status:** Deferred — not in current roadmap
**Earliest viable horizon:** Year 5+ (post-2030 for HV vehicles)
**Decision date:** Phase 0 product definition

---

## Why It's Deferred

The franchised dealer version of X-Ray Vision Diagnostics operates under a specific set of conditions that make it safe and legally viable: trained automotive technicians, OSHA-compliant workplaces, OEM-licensed data, and SAE J1742-certified safety procedures. A consumer at-home version removes all of these conditions simultaneously.

**Three blockers that cannot be shortcut:**

### 1. Safety Framework Does Not Exist for Consumer HV Work

OSHA 1910.147 (lockout/tagout) applies to workplace environments with trained workers. There is no equivalent federal safety standard for a homeowner working on their own EV battery. SAE J1742 is written for professional service environments.

A consumer at-home product that guides someone through EV battery repair without the OSHA framework has no safety certification anchor. If someone follows our instructions and is electrocuted, the liability exposure is catastrophic and uninsurable at a startup scale.

**What would change this:** A new consumer EV safety standard from SAE, ANSI, or NHTSA that establishes a homeowner-serviceable framework for EVs. No such standard exists as of 2026. The earliest plausible timeline for such a standard is 2030+.

### 2. OEM CAD Licensing Agreements Are Franchise-Only

All three OEM data agreements (Toyota, Ford, Stellantis) license CAD data for use in franchised dealer service bays only. Extending to consumer use would require renegotiating all three agreements — OEMs have strong commercial reasons to resist (franchise dealers pay OEMs for service revenue; empowering DIY repair cannibalizes that).

**What would change this:** OEMs losing franchise network control as EV ownership grows and out-of-warranty DIY repair becomes economically significant. This is a 2028–2032 dynamic at the earliest.

### 3. Different Product, Different Company

The consumer segment requires:
- Different UX (no assumed professional training; must be idiot-proof)
- Different pricing (consumer SaaS, not B2B per-bay)
- Different sales motion (App Store / consumer marketing vs. OEM-assisted B2B)
- Different support model (millions of consumers vs. hundreds of dealers)
- Different legal entity structure (consumer product liability insurance is categorically different)

Building this alongside the B2B product would split focus at the moment when the B2B moat needs to be consolidated. It is a separate company, not a feature.

---

## What It Would Take to Build

If the safety, licensing, and focus conditions were met:

| Requirement | Effort | Timeline |
|---|---|---|
| Consumer EV safety standard published | External dependency | 2030+ |
| OEM CAD licensing renegotiated for consumer use | 18-month negotiation | After safety standard |
| HoloLens 2 price drops to consumer-accessible range (< $500) | Hardware market | Apple Vision Pro trajectory suggests possible by 2028 |
| Consumer UX redesign (no professional assumption) | 12-month product sprint | After above |
| Consumer liability insurance secured | 6 months | After safety standard + legal framework |

**Earliest viable:** 2031–2033 for a limited pilot on out-of-warranty, non-HV repair tasks (ICE + mild hybrid only). Full EV HV consumer repair: 2035+.

---

## Interim Opportunity

While the full consumer version is deferred, one adjacent use case is viable in Year 3–4 without the safety/licensing blockers:

**Read-only diagnostic mode for EV owners:** An iPhone/Android app that reads OBD-II data and shows plain-language fault descriptions ("Your battery coolant pump is running 12% below normal — schedule service within 30 days"). No repair guidance, no CAD overlay, no HV work. Pure information.

This requires no HV safety framework, no OEM CAD licensing (OBD-II data is standardized), and no professional training assumption. It creates a consumer data flywheel (more vehicles reporting sensor data = better fleet baseline for predictive maintenance) and is a natural top-of-funnel for dealer referrals.

This is worth a 6-month evaluation in Year 3. If pursued, it is a separate SKU — not a feature of the dealer platform.
