# OEM Data Partnership Framework
## Step 1 — CAD Data Licensing Agreements

**Roadmap reference:** Phase 1, Deliverable 1
**Status:** `[~]` Framework complete; agreements pending execution
**Owner:** BD Lead + General Counsel
**Target close:** Month 2

---

## Objective

Secure signed CAD data licensing agreements with 3 OEM launch partners before any prototype development begins. Without OEM-sourced CAD data, the spatial overlay cannot be accurate enough for production use. This is the single highest-priority dependency in Phase 1.

**Target OEM partners (priority order):**

| Priority | OEM | Rationale | Key Vehicle | Internal Champion Target |
|---|---|---|---|---|
| 1 | **Toyota** | Largest US EV dealer network; existing TechStream diagnostic platform relationship possible | bZ4X (primary EV) | VP Service Operations, Toyota Motor North America |
| 2 | **Ford** | Mach-E volume growing; Ford Pro fleet relationships open doors | Mustang Mach-E | Director, Connected Vehicle Services, Ford Motor Co. |
| 3 | **Stellantis** | Ram EV + Jeep 4xe hybrid complexity creates strong FTFR pain | Ram 1500 REV | VP Aftersales, Stellantis North America |

---

## What We Are Requesting from OEMs

| Data Type | Format | Sensitivity | Our Use |
|---|---|---|---|
| Vehicle assembly CAD (geometry) | STEP / JT / gLTF | High | Spatial overlay rendering |
| Subsystem layer data (battery, coolant, HV bus) | Proprietary + STEP | High | Layer-specific rendering per DTC |
| VIN-to-configuration mapping | CSV / API | Medium | Variant-correct CAD selection per vehicle |
| OEM service bulletin corpus | PDF / structured text | Medium | RAG grounding for DiagnosticCore |
| DTC-to-component mapping table | JSON / CSV | Medium | AI layer selection logic |

---

## Key License Terms to Negotiate

**Must-haves (non-negotiable):**
- Right to render CAD geometry on authorized headsets in authorized service bays
- Right to use service bulletin text as RAG corpus (read-only, not redistributed)
- VIN-level entitlement gating — a tech can only see CAD for a vehicle physically present in their bay
- OEM retains CAD ownership; we hold a non-exclusive rendering license per deployment

**Offer in exchange:**
- De-identified FTFR improvement data reported quarterly to OEM
- OEM logo/branding on headset HUD for their vehicle models ("Powered by Toyota Service Data")
- OEM service bulletin currency: we commit to refreshing data within 72h of OEM publication
- Revenue share option: OEM receives 8–12% of per-bay subscription revenue for their vehicle coverage

**Red lines (do not accept):**
- CAD that cannot be used for real-time rendering (static display-only licenses)
- Agreements requiring OEM approval for every software release
- Data that must be re-queried from OEM cloud on every repair step (latency incompatible with our architecture)

---

## Outreach Timeline

| Week | Action | Owner |
|---|---|---|
| Week 1 | Identify internal champions via NADA dealer council contacts | BD Lead |
| Week 2 | Send NDA + executive 1-pager to all 3 OEMs | BD Lead |
| Week 3–4 | Discovery calls: confirm pain (FTFR gap, tech shortage) and CAD data structure | BD + PM |
| Week 5–6 | Draft term sheet; share with OEM legal | General Counsel |
| Week 7–8 | Negotiate; target signature | BD Lead + GC |
| Month 2 close | 2 of 3 agreements signed — exit criterion met | BD Lead |

---

## Risk: OEM Refusal

**Probability:** Medium (OEMs historically protective of CAD IP)

**Mitigation path if Toyota refuses:**
- Propose a "data room" model: CAD lives in OEM-controlled cloud; our system sends rendering queries, OEM server returns gLTF geometry. We never hold raw CAD.
- Reference precedent: Volkswagen Group's HoloLens AR partnership (2022) demonstrates OEM willingness when security architecture is sound.
- Alternative for v1 beta only: Reconstruct approximate vehicle geometry from photogrammetry scans of donor vehicles. Flag clearly as "estimated geometry" in overlay. Lower accuracy but unblocks prototype phase.
