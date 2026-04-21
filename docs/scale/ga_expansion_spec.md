# GA Expansion Spec — 10 Dealerships, 50 Bays, 5 OEM Vehicle Lines
## Step 36 — Phase 4, Scale 1

**Roadmap reference:** Phase 4, Scale 1
**Status:** `[ ]` Not started
**Owner:** Sales + BD + Engineering
**Target:** 50 bays live and billing by Month 20

---

## Objective

Execute commercial expansion from 3 beta bays to 50 revenue-generating bays across 10 franchised dealerships and 5 OEM vehicle lines. This is the GA launch milestone.

---

## Expansion Targets

### Dealership Cohort (10 dealerships, 50 bays)

| Tier | Count | Bay Target | OEM Mix | Rationale |
|---|---|---|---|---|
| Tier 1 — Beta graduates | 3 | 15 bays (avg 5/site) | Toyota, Ford | Already live; expand from 1 bay to 5 bays each |
| Tier 2 — Warm leads | 4 | 20 bays (avg 5/site) | Toyota, Ford, Stellantis | Dealer principals engaged during beta; referrals from beta sites |
| Tier 3 — New outbound | 3 | 15 bays (avg 5/site) | Toyota, Ford | Sales-sourced; signed after case study published |

### OEM Vehicle Lines (5 total by Phase 4 exit)

| OEM | Models Enrolled | Phase Added |
|---|---|---|
| Toyota | bZ4X, RAV4 Hybrid | Phase 2 / Phase 3 |
| Ford | Mustang Mach-E, F-150 Lightning | Phase 3 / Phase 4 |
| Stellantis | Jeep Wrangler 4xe | Phase 4 (new) |

**Stellantis partnership prerequisite:** CAD data licensing agreement (same framework as Toyota/Ford per `docs/legal/oem_partnership_framework.md`). BD must initiate by Month 13 for Phase 4 readiness.

---

## Scaling the Edge Infrastructure

### Per-Bay Hardware (unchanged from Phase 3)
- 1× Jetson AGX Orin 64GB per bay
- 2× HoloLens 2 (active + spare)
- 1× FLIR Lepton 3.5 thermal camera
- WiFi6E access point per bay

### Cloud Infrastructure Scaling (new for Phase 4)
Phase 3 used a single-AZ RDS instance adequate for 3 bays. Phase 4 requires:

| Component | Phase 3 | Phase 4 |
|---|---|---|
| Database | RDS PostgreSQL single-AZ | RDS PostgreSQL Multi-AZ, db.m6g.xlarge |
| Analytics | Local edge DB | Cloud DW: Redshift Serverless |
| Dashboard | S3 + CloudFront static | Same + CDN caching for 50-site concurrency |
| Monitoring | Manual log review | CloudWatch dashboards + PagerDuty on-call |
| Deployment | Manual SSH per edge server | Ansible playbooks for fleet-wide updates |

### Fleet Management
50 edge servers across 10 sites cannot be managed manually. Phase 4 requires:
- Remote management: AWS Systems Manager (SSM) for all Jetson AGX units
- OTA updates: edge server software deployed via S3 artifact + systemd service reload; no manual SSH required
- Health monitoring: every edge server publishes heartbeat to CloudWatch every 60 seconds; alert if silent > 3 minutes

---

## Site Installation at Scale

Phase 3 used a 3-day manual install per site. Phase 4 must bring this to 1 day via:
- Pre-staged rack units shipped from warehouse (Jetson configured, software installed, VINs pre-cached per dealer's top-20 list)
- Installation crew of 2 (not 4): one on network, one on hardware
- Remote software validation by Engineering (no need to travel)
- Acceptance test via Manager Dashboard (Engineering PM confirms all 5 agents healthy remotely)

**Install SLA:** 1 business day per bay, 2 bays per crew per day → 5 crews needed for 50-bay rollout over 5 weeks.

---

## CAD Coverage for 5 OEM Lines

Each new OEM vehicle model requires:
1. CAD data licensing (OEM partnership framework)
2. Ingestion through CAD pipeline (`docs/data/cad_pipeline_spec.md`)
3. Layer definition file (per `docs/vehicles/ford_mache_integration_spec.md` pattern)
4. VoiceNLP vocabulary extension
5. DiagnosticCore RAG corpus addition
6. SafetyGuard HV zone update

**Timeline per new model:** 6–8 weeks from CAD data receipt to production deployment.

**Phase 4 new models (Ford F-150 Lightning, Jeep Wrangler 4xe):** initiate CAD licensing by Month 13 to have both ready for Month 17 deployment.

---

## Exit Criteria

- [ ] 50 bays live (edge server online, at least 1 headset session completed per bay)
- [ ] 50 bays on monthly subscription billing (revenue confirmed in finance system)
- [ ] All 5 OEM vehicle lines enrolled and available at ≥ 1 deployment site each
- [ ] Fleet management (SSM + OTA updates) tested on all 50 units
- [ ] FTFR ≥ 90% demonstrated on enrolled fault types (n ≥ 200 repairs)
