# 500 Bays Expansion Spec — Phase 5 US Scale
## Step 50 — Phase 5, Scale 1

**Roadmap reference:** Phase 5, Scale 1
**Status:** `[ ]` Not started
**Owner:** Sales + Engineering + Customer Success + BD
**Target:** 500 bays active and billing by Month 30
**Prerequisite:** Phase 4 exit criteria met (50 bays, FTFR ≥ 90%, named case study published, OSHA + SAE compliance)

---

## Objective

Expand from 50 commercial bays (Phase 4 exit) to 500 active bays across 50+ US dealerships — a 10× scale-up. This milestone establishes X-Ray Vision Diagnostics as a material, revenue-generating platform and enables the OEM-embedded integration negotiations that define Phase 5's long-term moat.

**Revenue context at 500 bays:** At the Professional tier price point ($2,400/bay/month from `docs/pricing/subscription_model_spec.md`), 500 bays = $1.2M MRR / $14.4M ARR. This is the scale at which the company becomes self-sustaining and OEM-embedded deal negotiations become credible.

---

## Expansion Strategy

### Dealer Acquisition Model

Phase 5 growth comes from three parallel channels, not a single sales motion:

| Channel | Bays Target | Avg Deal Size | Primary Driver |
|---|---|---|---|
| Phase 4 customer expansion | 150 bays | 5–10 bays/dealer | Existing customers adding bays post-Phase 4 results |
| Referral from case study + conference | 100 bays | 5 bays/dealer | NADA case study → warm inbound; peer referrals from enrolled dealer groups |
| Direct outbound (AE-led) | 150 bays | 3–5 bays/dealer | AE-driven prospecting; target top 100 EV-certified dealer groups by service volume |
| OEM-assisted channel | 100 bays | 10+ bays/dealer | Toyota/Ford dealer network newsletters; OEM regional manager introductions |

**OEM-assisted channel:** As OEM partnerships mature (Toyota + Ford BD relationships from Phase 1–4), OEM regional service managers become a de facto channel — they recommend X-Ray Vision Diagnostics to certified dealers as part of their EV service quality push. This channel requires no outbound sales cost but depends on maintaining OEM relationships and compliance status.

### Dealer Profile for Phase 5 Expansion

| Criterion | Target |
|---|---|
| EV/hybrid service volume | ≥ 20 enrolled fault type repairs per month |
| Bay count | ≥ 5 service bays (multi-bay deployments only at Phase 5 scale) |
| OEM certification | Toyota, Ford, Stellantis, GM, or Hyundai EV-certified |
| DMS compatibility | CDK Drive or Reynolds & Reynolds (job card integration prerequisite) |
| IT infrastructure | WiFi6E capable or willing to upgrade (network spec: `docs/infrastructure/network_connectivity_spec.md`) |

---

## Geographic Expansion

### US Market Coverage — Phase 5 Priority Markets

Phase 4 concentrated in 10 dealerships across 2–3 metro markets. Phase 5 expands nationally across top EV service markets.

| Market Tier | States | Rationale |
|---|---|---|
| Tier 1 (Months 21–24) | CA, TX, FL, NY, WA | Highest EV/hybrid vehicle density; highest dealer service volume |
| Tier 2 (Months 24–27) | IL, GA, CO, AZ, NC | Fast-growing EV adoption; high Toyota and Ford franchise density |
| Tier 3 (Months 27–30) | OH, MI, PA, VA, MN | Traditional auto market; adoption following Tier 1/2 reference network |

**California priority:** CA has the highest EV penetration in the US and the most complex EV service regulatory environment (CARB-compliant repairs required). Strong FTFR improvement story resonates most with CA dealers who have the highest EV service volume and the most pressure to fix right the first time.

---

## Infrastructure Scaling — 500 Edge Servers

Managing 500 Jetson AGX Orin edge servers across 50+ sites is qualitatively different from 50 servers at 10 sites. Phase 5 requires:

### Fleet Management at Scale

| Capability | Phase 4 (50 servers) | Phase 5 (500 servers) |
|---|---|---|
| Remote management | AWS SSM per-server | SSM fleet groups by region + OEM line |
| OTA updates | Ansible playbooks, manual trigger | Automated staged rollout — canary (5%) → regional (25%) → full fleet |
| Monitoring | CloudWatch per-server | Centralized fleet health dashboard; PagerDuty on-call with auto-escalation |
| Hardware failure response | Manual dispatch | 4-hour SLA: spare unit shipped same-day; remote edge server diagnostic first |
| Configuration management | Per-site config files | Fleet config as code (Terraform + Ansible); site config derived from OEM + dealer profile |

### Multi-Region Cloud Architecture

Phase 4 ran on a single-region cloud stack (US-East). 500 bays across a national footprint require:

| Component | Phase 4 | Phase 5 |
|---|---|---|
| CAD database (primary) | RDS PostgreSQL Multi-AZ, US-East | RDS Aurora Global Database: primary US-East, read replica US-West |
| OEM CAD serving latency | < 2s (acceptable at 50 bays) | < 500ms required at 500 bays — regional CDN for CAD mesh files |
| Analytics / manager dashboard | Redshift Serverless, US-East | Redshift Multi-AZ + CloudFront for dashboard at 50-site concurrency |
| CAD pre-fetch pipeline | Single pipeline, 50 edge servers | Distributed pipeline: regional ingestion nodes pre-fetch CAD for local fleet |
| SLA | 99.5% uptime target | 99.9% uptime SLA (contractually required at commercial scale) |

### CAD Database — Top 20 Vehicle Models

Phase 5 requires expanding the CAD database to cover the top 20 highest-service-volume vehicle models in the US market (spec: `docs/scale/cad_database_expansion_spec.md`). This is a parallel dependency:

| New OEM Lines (Phase 5) | Target Models | Partnership Status |
|---|---|---|
| GM | Chevy Bolt EUV, GMC Hummer EV | BD initiation Month 18; CAD licensing Month 20 |
| Hyundai/Kia | Ioniq 5, EV6 | BD initiation Month 19; CAD licensing Month 21 |
| Rivian | R1T, R1S | BD initiation Month 20; fleet/commercial focus |
| BMW | iX, i4 | BD initiation Month 21; premium dealer segment |

---

## Deployment Operations — 1-Day Install SLA

Phase 4 used a 3-day installation procedure (from `docs/deployment/beta_site_install_spec.md`). At 500-bay scale, 3-day installs are operationally unsustainable. Phase 5 requires a 1-day install SLA.

### Changes Required to Achieve 1-Day Install

| Phase 4 Constraint | Phase 5 Solution |
|---|---|
| Edge server configured manually at site | Zero-touch provisioning: edge server ships pre-configured; plug in → connects to fleet management → self-configures |
| ArUco marker placement by engineer | Technician-assisted self-install with AR calibration guide (in-headset instructions for marker placement) |
| Network configuration requires IT visit | Standardized WiFi6E access point pre-configured for our SSID and firewall rules; IT pre-provision via remote access |
| VIN database seeded manually | VIN auto-seeded on first OBD-II connect; CAD pre-fetch triggered automatically |
| Training scheduled separately from install | Orientation delivered via onboarding module in headset; peer champion activated remotely |

**Target install timeline (1 day):**
- Morning (3 hours): hardware mount, network connect, self-provision
- Midday (2 hours): calibration + ArUco placement (tech-assisted)
- Afternoon (3 hours): technician onboarding session (headset-guided)
- End of day: first repair session completed

### Field Support Model

At 500 bays, a dedicated field engineering team is required:

| Role | Headcount | Coverage |
|---|---|---|
| Regional Field Engineers | 10 (1 per region per 50 bays) | On-site installation, hardware replacement |
| Remote Support Engineers | 5 | Remote diagnosis, OTA updates, escalation |
| Customer Success Managers | 8 (1 per ~6 accounts) | Adoption, renewal, expansion |

---

## Phase 5 — Canada and UK Preparation

Phase 5 exits with international expansion ready. Canada and UK are the first two international markets (full launch in Phase 5 deliverables).

### Canada
- **Regulatory adaptation:** Transport Canada EV safety regulations align closely with US FMVSS — minimal additional certification
- **Language:** English primary; French required for Quebec dealer network → Canadian French localization (follows Spanish language spec model: `docs/features/voice_nlp_spanish_spec.md`)
- **OEM coverage:** Same Toyota, Ford models used in US market; Stellantis RAM models added for Canadian market
- **Target:** 50 Canadian bays by Month 30

### UK
- **Regulatory adaptation:** UKCA certification required (replaces CE mark post-Brexit); DVSA EV service regulations apply
- **Language:** British English — VoiceNLP accent model requires UK automotive vocabulary training; some component names differ (e.g., "bonnet" vs. "hood")
- **OEM coverage:** Toyota, Ford, Stellantis UK models; right-hand drive ArUco placement differs from US vehicles
- **Target:** 25 UK bays by Month 30 (smaller initial footprint given longer regulatory lead time)

---

## OEM-Embedded Integration — Deal Initiation

Phase 5's strategic moat is transitioning from a dealer-sold product to an OEM-embedded product — where Toyota, Ford, or another OEM sells X-Ray Vision Diagnostics as part of their dealer tooling package (like Hunter alignment equipment).

### What OEM-Embedded Means

- OEM sells the platform to their certified dealer network as part of a service excellence package
- OEM funds part of the hardware cost (subsidized deployment)
- OEM data licensing simplified (OEM already granted; embedded relationship)
- Pricing: per-bay license sold through OEM → OEM margins; our revenue is wholesale + volume + data

### Prerequisites for Negotiation (all Phase 5 entry)

- 500 active bays demonstrating ≥ 90% FTFR
- Named case study from a dealer group the OEM knows
- OSHA + SAE compliance confirmed
- OEM data licensing relationship (Phase 1) already in place — this conversation builds on BD relationship, not starting cold

### Negotiation Initiation Target

- Toyota Motor North America: Month 22 — leverage Toyota dealer relationship + LNO status
- Ford Pro (commercial fleet division): Month 23 — F-150 Lightning fleet service is a distinct sales channel from retail dealers
- GM (secondary): Month 26 — pending GM CAD licensing completion

---

## Phase 5 Exit Criteria

- 500 bays active and billing
- OEM-embedded deal in negotiation with at least 1 major OEM
- Predictive maintenance mode in closed beta
- SOC 2 Type II audit completed
- Canada and UK deployments initiated (≥ 25 bays combined)

---

## Links to Related Specs

- Phase 4 GA expansion (50 bays): `docs/scale/ga_expansion_spec.md`
- CAD database expansion (top 20 models): `docs/scale/cad_database_expansion_spec.md`
- Edge server hardware: `docs/infrastructure/edge_server_bay_spec.md`
- Network connectivity: `docs/infrastructure/network_connectivity_spec.md`
- Beta site installation: `docs/deployment/beta_site_install_spec.md`
- Onboarding program: `docs/deployment/onboarding_program_spec.md`
- OEM partnership framework: `docs/legal/oem_partnership_framework.md`
- Spanish language support (model for French/UK localization): `docs/features/voice_nlp_spanish_spec.md`
- Subscription pricing model: `docs/pricing/subscription_model_spec.md`
