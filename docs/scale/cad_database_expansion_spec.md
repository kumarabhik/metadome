# CAD Database Expansion Spec — Top 20 Service-Volume Vehicle Models
## Step 37 — Phase 4, Scale 2

**Roadmap reference:** Phase 4, Scale 2
**Status:** `[ ]` Not started
**Owner:** Data Engineering + BD
**Target:** Top 20 models ingested and indexed by Month 18

---

## Objective

Expand the CAD database from the 3 models in beta (Toyota bZ4X, Toyota RAV4 Hybrid, Ford Mustang Mach-E) to cover the top 20 highest-service-volume vehicle models in the US franchised dealership market. This broadens the addressable fault universe and enables Phase 5 scale.

---

## Model Selection Methodology

### Data Source
US dealership service volume data from NADA 2024 Industry Data Report + OEM dealer service network statistics (obtained through OEM partnerships).

### Ranking Criteria
1. **Annual service visits** at franchised dealerships (warranty + customer pay)
2. **EV/hybrid share** — prioritize models with complex electrified powertrains (higher diagnostic complexity = higher FTFR leverage)
3. **OEM partnership status** — models where CAD licensing is already active rank higher

### Target Top 20 List (Prioritized)

| Rank | Model | OEM | Powertrain | CAD Status | Priority |
|---|---|---|---|---|---|
| 1 | Toyota RAV4 Hybrid | Toyota | Hybrid | ✅ Phase 3 | Done |
| 2 | Ford F-150 Lightning | Ford | BEV | 🔄 Phase 4 | High |
| 3 | Toyota bZ4X | Toyota | BEV | ✅ Phase 2 | Done |
| 4 | Ford Mustang Mach-E | Ford | BEV | ✅ Phase 3 | Done |
| 5 | Jeep Wrangler 4xe | Stellantis | PHEV | 🔄 Phase 4 | High |
| 6 | Toyota Camry Hybrid | Toyota | Hybrid | 📋 Phase 4 new | High |
| 7 | Chevrolet Equinox EV | GM | BEV | 📋 Phase 4 new | High (pending GM partnership) |
| 8 | Hyundai Ioniq 6 | Hyundai | BEV | 📋 Phase 4 new | Medium (pending partnership) |
| 9 | Ford Escape PHEV | Ford | PHEV | 📋 Phase 4 new | High |
| 10 | Toyota Prius Prime | Toyota | PHEV | 📋 Phase 4 new | High |
| 11 | Kia EV6 | Kia/Hyundai | BEV | 📋 Phase 4 new | Medium |
| 12 | Toyota Highlander Hybrid | Toyota | Hybrid | 📋 Phase 4 new | High |
| 13 | Ford Explorer PHEV | Ford | PHEV | 📋 Phase 4 new | High |
| 14 | Chevrolet Bolt EUV | GM | BEV | 📋 Phase 4 new | Medium |
| 15 | Jeep Grand Cherokee 4xe | Stellantis | PHEV | 📋 Phase 4 new | High |
| 16 | BMW 5 Series PHEV | BMW | PHEV | 📋 Phase 4 new | Low (luxury, low volume) |
| 17 | Toyota Sienna Hybrid | Toyota | Hybrid | 📋 Phase 4 new | Medium |
| 18 | Ford Maverick Hybrid | Ford | Hybrid | 📋 Phase 4 new | Medium |
| 19 | Rivian R1T | Rivian | BEV | 📋 Phase 4 new | Low (limited dealer network) |
| 20 | Mercedes EQS | Mercedes | BEV | 📋 Phase 4 new | Low (luxury, low volume) |

> GM and Hyundai/Kia CAD partnerships require new OEM agreements. Initiate BD outreach by Month 14.

---

## Ingestion Pipeline at Scale

The CAD pipeline (`docs/data/cad_pipeline_spec.md`) was designed for single-model ingestion. Phase 4 requires a batch ingestion workflow:

### Batch Ingestion Process
1. OEM delivers CAD package (STEP AP214 ZIP archive) via secure SFTP
2. Ingestion job triggered automatically on file arrival
3. Pipeline: normalize → mesh → VIN-index → layer definition → cache build
4. Validation: automated QA checks (mesh bounds, layer completeness, HV zone coverage)
5. Staging review: PM + Engineering spot-check 5 random CAD views before production promotion
6. Production deployment: edge server cache update via OTA push

**Target throughput:** 1 new model ingested per 2 weeks (bottleneck: OEM CAD delivery, not pipeline speed)

### Storage Requirements

| Component | Per Model | 20 Models | Notes |
|---|---|---|---|
| Raw STEP archive | ~8 GB | ~160 GB | Retained as source of truth |
| Normalized mesh | ~2 GB | ~40 GB | GPU-optimized LOD pyramid |
| Layer definitions | ~50 MB | ~1 GB | JSON + texture maps |
| Edge server cache | ~2 GB | ~40 GB per bay | Pre-loaded at VIN match |

Phase 4 total cloud storage: ~200 GB raw + processed. AWS S3 cost: ~$5/month. Negligible.

---

## OEM Partnership Pipeline

Phase 4 requires 4 new OEM relationships (GM, Hyundai/Kia, Stellantis expansion, BMW):

| OEM | BD Contact Needed By | Agreement Target | Models in Scope |
|---|---|---|---|
| GM | Month 13 | Month 15 | Equinox EV, Bolt EUV |
| Hyundai/Kia | Month 13 | Month 15 | Ioniq 6, EV6 |
| BMW | Month 14 | Month 17 | 5 Series PHEV |

**Stellantis** — Wrangler 4xe requires Stellantis expansion of existing framework (already initiated per `docs/legal/oem_partnership_framework.md`).

---

## Exit Criteria

- [ ] Top 10 models (by service volume, excluding GM/Hyundai pending) ingested and available in production
- [ ] Top 20 models ingested by Month 18 (subject to OEM partnership timelines)
- [ ] Automated batch ingestion pipeline live (manual ingestion retired)
- [ ] QA validation automated for mesh bounds + layer completeness
- [ ] GM and Hyundai/Kia CAD partnerships in signed agreement or advanced negotiation
