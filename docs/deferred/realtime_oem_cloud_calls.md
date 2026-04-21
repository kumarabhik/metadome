# Deferred: Real-Time OEM Cloud Calls During Active Repair Step

**Status:** Deferred — latency risk unacceptable; all data must be edge-cached
**Earliest viable horizon:** Reconsidered when 5G private network infrastructure is standard in dealer bays (2028+)
**Decision date:** Phase 2 architecture decision

---

## What It Is

Instead of syncing OEM CAD data and service bulletins to the edge server in advance, query the OEM's cloud systems in real time during an active repair session — fetching the latest TSB, the most current CAD revision, or live OEM diagnostic data at the moment it's needed.

The appeal: always-current data without a sync lag. If Toyota publishes a revised TSB at 9am, a tech working at 10am gets the updated procedure without waiting for the next nightly sync.

---

## Why It's Deferred

### 1. The 40ms Latency Budget Has No Room for a Cloud Round-Trip

The real-time render path budget is 40ms end-to-end:
- VoiceNLP: 8ms
- DiagnosticCore: 12ms
- CADRenderer: 15ms
- HoloLens display: 5ms

A round-trip to an OEM cloud API — even on a fast enterprise network — adds:
- Enterprise WiFi to internet: 5–15ms
- Internet routing to OEM data center: 20–80ms (varies by OEM location and CDN)
- OEM API response time: 50–200ms (OEM enterprise APIs are not low-latency consumer APIs)
- **Total addition: 75–295ms**

Adding a real-time OEM cloud call to the DiagnosticCore path would push total latency to 115–335ms — 3–8× over the 40ms budget. The overlay would be visibly laggy; technicians would find it unusable.

### 2. OEM API Availability Cannot Be Guaranteed During a Safety-Critical Session

If the OEM API is down (planned maintenance, outage, rate limiting) during an active HV repair session, the system would either:
- Fail to return repair steps → tech is left without guidance mid-repair on an open HV system
- Fall back to cached data → which is what we do today anyway, making the real-time call redundant

Option A is a safety incident waiting to happen. Option B means the real-time call provides no reliability benefit — it only adds latency when the API is up and introduces risk when it's down.

**Principle:** Safety-critical systems should not depend on third-party API availability during the critical path.

### 3. OEM APIs Are Not Designed for Real-Time Repair Guidance

Toyota TIS (Technical Information System), Ford ETIS, and Stellantis TechCONNECT are designed for technicians to look up information on a laptop — paginated web responses, PDF downloads, session-based authentication. They are not low-latency REST APIs designed for machine-to-machine real-time queries.

Getting OEM to build and maintain a low-latency API for our use case would require 12–18 months of OEM engineering partnership, contractual SLAs for uptime and latency, and ongoing maintenance. This is a significant ask for Phase 1–2.

---

## What We Do Instead (Current Architecture)

**Nightly sync:** CAD files and TSBs are pulled from OEM systems once per night and cached on the edge server. Edge server holds 72 hours of data locally, making the platform resilient to cloud outages.

**On-demand sync trigger:** If a new TSB is published for an enrolled vehicle model, our ingestion pipeline detects it within 4 hours (polling OEM Partner Portal). An on-demand sync is triggered to edge servers at that dealer's region — not waiting for the nightly batch.

**Result:** The maximum data staleness in production is 4 hours for a critical TSB update, 24 hours for routine CAD revisions. For a product with a 40ms render path, 4-hour sync lag is operationally irrelevant.

---

## When This Could Be Revisited

### Condition 1: 5G Private Network in Dealer Bays

5G private networks (mmWave, sub-6GHz) deployed within a dealer facility could provide 1–2ms round-trip to an on-premises OEM data node. At 2ms, a real-time OEM cloud call consumes only 2ms of the 40ms budget — acceptable.

Toyota and Ford are both piloting 5G private network deployments in dealership facilities as of 2026. If this becomes standard dealer infrastructure by 2028–2029, real-time OEM calls to an on-premises cache (not a remote cloud) become viable.

### Condition 2: OEM Builds a Low-Latency Edge API

If an OEM builds and exposes a low-latency API (< 5ms response time, 99.99% uptime SLA) specifically for spatial diagnostic platforms, we can integrate it. This would likely emerge from the OEM embedded deal negotiation (Phase 5) — if Toyota is embedding our platform, they have incentive to build a better data delivery mechanism.

**Year 4–5 discussion:** Include "real-time data API" as a negotiation point in the OEM embedded deal. Not a product launch blocker, but a future capability to spec out together.

### Condition 3: Non-Safety-Critical Supplemental Data

Real-time cloud calls are appropriate for **supplemental** data that does not affect the repair step sequence:
- "Is this TSB superseded by a newer one?" (informational check, not a repair step)
- "Is this part number back-ordered?" (parts availability, not a safety step)
- "Is there an active recall on this VIN?" (can be fetched async, displayed in pre-session summary)

These queries can be made async (before the session starts, not during a repair step) with 2–5 second acceptable latency. This is implementable now, without changing the edge-first architecture.

**Recommendation:** Add async pre-session OEM data check (recall status, TSB freshness check) in Year 3 as a low-risk way to surface real-time OEM data without touching the critical render path.
