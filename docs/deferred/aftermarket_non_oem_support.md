# Deferred: Aftermarket / Non-OEM Vehicle Support

**Status:** Deferred — CAD data cannot be sourced reliably without OEM partnership
**Earliest viable horizon:** Year 4–5 for limited scope
**Decision date:** Phase 0 product definition

---

## What It Is

Extending X-Ray Vision Diagnostics to vehicles not covered by an OEM partnership agreement — independent repair shops, fleet operators, and used-vehicle dealers servicing vehicles outside the franchised dealer network, using aftermarket CAD data rather than OEM-licensed data.

---

## Why It's Deferred

### 1. Aftermarket CAD Data Quality Is Inadequate for HV Safety Applications

Aftermarket parts data (AAIA/AutoCare standard, used by ALLDATA, Mitchell1) covers component identification, part numbers, and labor times. It does not include:
- 3D spatial coordinates of components (required for CAD overlay)
- HV zone boundary definitions (required for SafetyGuard)
- Precise wiring harness routing (required for accurate layer rendering)

Without HV zone data specifically, the SafetyGuard agent cannot define the exclusion zone for HV-STOP. Deploying the overlay without a calibrated HV zone is worse than not deploying it — the tech might assume they're safe when they're not.

**This is a safety constraint, not a data quality preference.** Until aftermarket data includes HV zone spatial definitions at OEM-level precision, aftermarket support is incompatible with our HV-STOP safety architecture.

### 2. Reverse-Engineering OEM CAD Is a Legal Minefield

One path to aftermarket CAD would be to scan vehicles with structured light or photogrammetry to reconstruct 3D models. This is technically feasible — several companies do this for video game asset creation. For automotive service:
- The reconstructed model may not accurately reflect the as-manufactured HV zone boundaries (vehicle-to-vehicle variation, manufacturing tolerances)
- OEMs have claimed trade dress and trade secret protection over vehicle geometry in some jurisdictions
- Using reconstructed CAD for safety-critical HV guidance without OEM authorization creates uncapped liability if the reconstruction is inaccurate

Legal counsel has advised against pursuing reconstructed CAD for HV-adjacent features without OEM sign-off.

### 3. Independent Shop Market Has Lower FTFR Urgency

Franchised dealers have FTFR as a KPI tied to OEM incentive payouts (CSI scores). Independent shops are not measured on FTFR by any external party. The economic case for a $3,500/bay/year investment is weaker when the primary buyer has no external FTFR accountability.

This doesn't mean independents don't value quality — they do — but the sales cycle is longer and the willingness to pay is lower.

---

## What Would Make Aftermarket Support Viable

### Path 1: OEM "Open Data" Initiative
Some OEMs are beginning to publish safety-relevant vehicle data under open licensing for repair right / right-to-repair compliance. The Massachusetts Right to Repair law (2020) established a precedent. If federal right-to-repair legislation passes and requires OEMs to provide 3D component location data to aftermarket, our existing data pipeline can ingest it without renegotiation.

**Timeline:** Federal right-to-repair legislation is active in Congress as of 2026. If passed in 2027–2028, aftermarket data could include spatial component data by 2029.

### Path 2: Non-HV Aftermarket Support First
For ICE vehicles and non-HV fault types (timing chain, brake system, ICE sensors), the HV zone constraint does not apply. Aftermarket spatial data for ICE components is more feasible to reconstruct or source from third parties.

**Year 4–5 opportunity:** Launch an "ICE Diagnostics" aftermarket tier using reconstructed or community-sourced CAD, explicitly scoped to non-HV fault types, targeted at independent shops that service ICE vehicles. This does not require OEM partnership and avoids the HV safety constraint entirely.

**Target:** 200 independent shop bays by Year 5 under a $2,400/bay/year "ICE tier" — no HV features, limited CAD coverage, but opens the market before the full aftermarket solution is available.

### Path 3: Fleet Operator Direct Partnership
Large fleet operators (rental companies, delivery fleets, government fleets) own large numbers of a specific vehicle model and have direct OEM relationships. A fleet operator with 500 Ford Mach-E vehicles can negotiate their own OEM data agreement and license our platform under an enterprise fleet tier.

This is not "aftermarket" in the traditional sense — it's a direct B2B relationship with a fleet operator rather than a dealer. No independent aftermarket data needed; the fleet operator provides the OEM authorization.

**This is actionable in Phase 5 / Year 3** as a high-value expansion segment. Target: Amazon Delivery (Ram 1500 EV fleet), Hertz (mixed EV fleet), US Government GSA fleet.

---

## Recommended Action

- **Now:** Do not pursue aftermarket; maintain OEM-exclusive positioning
- **Year 3:** Launch fleet operator tier (Path 3) — does not require aftermarket data
- **Year 4:** Evaluate ICE-only aftermarket tier (Path 2) if independent shop demand signals are strong
- **Year 5+:** Monitor right-to-repair legislation; assess Path 1 when OEM data obligations become clearer
