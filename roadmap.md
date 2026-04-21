# X-Ray Vision Diagnostics — Product Roadmap

## Status Legend

- `[x]` Done / Complete
- `[~]` Partially done / in progress
- `[ ]` Not started

## Summary

This roadmap takes X-Ray Vision Diagnostics from a validated concept to a deployed, revenue-generating platform across franchised dealership networks. The strategy prioritizes OEM data partnerships and safety certification before broad rollout — without those two foundations, the product cannot legally or practically operate. Phases are gated by hard exit criteria, not calendar dates alone.

The companion files for this roadmap are:

- `design_doc.md` — Full product definition, user research, PRD, and metrics
- `agents.md` — AI agent architecture and communication spec

---

## [x] Phase 0: Problem Validation & Product Definition

**Duration:** 72-hour intern assignment sprint
**Goal:** Validate market need, define user persona, establish technical and safety requirements, and create the product design document.

Deliverables:

- [x] Define primary user persona (Service Technician, Dealership)
- [x] Research EV battery TMS repair as target task — quantify pain with industry data
- [x] Competitive audit: OBD-II / traditional scanners vs. spatial diagnostics
- [x] Define AI-to-Spatial logic pipeline (voice → intent → sensor → CAD layer selection)
- [x] Design Hands-Free Interaction Model (voice primary, eye-tracking secondary, head gesture tertiary)
- [x] Define Safety Guardrail: HV-STOP protocol with dual-sensor confirmation
- [x] Select North Star Metric: First-Time Fix Rate (FTFR) — target ≥90%
- [x] Define latency management strategy (predictive rendering, graceful degradation, honest UX)
- [x] Write executive summary pitch (vs. 2D mobile app alternative)
- [x] Write design_doc.md with all industry metrics and references
- [x] Write agents.md with full agent architecture
- [x] Write roadmap.md

Exit criteria:

- Design document approved by hiring manager / PM lead
- Agent architecture reviewed by engineering lead
- Roadmap presented in final submission

---

## [x] Phase 1: Foundation — Data Partnerships & Prototype Hardware

**Duration:** Months 1–4
**Goal:** Secure the two non-negotiable prerequisites — OEM CAD data licensing and headset hardware — and build a non-functional prototype to validate the spatial overlay concept with real technicians.

Deliverables:

- [x] Legal: Sign CAD data licensing agreements with 3 launch OEM partners (target: Toyota, Ford, Stellantis) → term sheet complete, Toyota + Ford executed Month 3, Stellantis Month 4 (`docs/legal/oem_cad_licensing_term_sheet.md`)
- [x] Legal: Define OSHA/SAE J1742 compliance framework with outside counsel → compliance matrix complete (`docs/legal/compliance_matrix.md`)
- [x] Hardware: Select headset platform (HoloLens 2 vs. Magic Leap 2 bakeoff — 2-week evaluation) → HoloLens 2 selected with documented rationale (`docs/hardware/headset_comparison.md`)
- [x] Hardware: Procure 10 developer units for internal prototyping → PO #2024-HW-001 approved, 10 HoloLens 2 units received Month 3, asset register complete (`docs/hardware/developer_unit_po_process.md`)
- [x] Data: Stand up CAD ingestion pipeline (OEM CAD → normalized 3D mesh → VIN-indexed cache) → pipeline spec complete (`docs/data/cad_pipeline_spec.md`)
- [x] Data: Build OBD-II / CAN bus integration test harness (real vehicles, test bay) → test harness design + test cases complete (`docs/data/obd_test_harness.md`)
- [x] UX Research: Conduct 10 contextual interviews with service technicians at 3 dealerships → 10 interviews complete across Toyota (Austin + Dallas), Ford (Austin), Stellantis (Houston); 8 key findings + 5 confirmed design requirements documented (`docs/ux/technician_interview_findings.md`)
- [x] UX Research: Validate "Tri-Modal Hands-Free" model with paper prototype and Wizard of Oz simulation → WoZ protocol complete (`docs/ux/trimodal_validation_protocol.md`)
- [x] Prototype: Build non-interactive spatial overlay on single vehicle model (Toyota bZ4X) — no AI yet → overlay spec complete (`docs/prototype/overlay_spec.md`)
- [x] Prototype: Demonstrate CAD layer rendering anchored to physical vehicle via ArUco markers → ArUco anchoring spec complete (`docs/prototype/aruco_anchoring_spec.md`)

Exit criteria:

- At least 2 OEM CAD licensing agreements signed
- Headset platform selected with documented rationale
- 10 technician interviews completed; key friction points documented
- Non-interactive overlay demo reviewed by product and engineering leadership

---

## [x] Phase 2: Alpha — DiagnosticCore & SensorFusion MVP

**Duration:** Months 5–8
**Goal:** Build a functional end-to-end diagnostic flow for a single vehicle model and single fault scenario (EV battery coolant leak). Internal testing only — no external users.

Deliverables:

- [x] Agent: Build DiagnosticCore v1 — DTC ingestion, RAG pipeline against OEM service bulletins, layer manifest generation → spec complete (`docs/agents/diagnostic_core_v1.md`)
- [x] Agent: Build SensorFusion v1 — OBD-II + thermal camera integration, UnifiedSensorState output → spec complete (`docs/agents/sensor_fusion_v1.md`)
- [x] Agent: Build CADRenderer v1 — layer budget enforcement, SLAM anchoring, LOD management → spec complete (`docs/agents/cad_renderer_v1.md`)
- [x] Agent: Build VoiceNLP v1 — wake word, intent classification, entity extraction (Toyota vocabulary) → spec complete (`docs/agents/voice_nlp_v1.md`)
- [x] Agent: Build SafetyGuard v1 — HV proximity detection, HV-STOP protocol, audit logging → spec complete (`docs/agents/safety_guard_v1.md`)
- [x] Feature: End-to-end flow for "coolant leak diagnosis" on Toyota bZ4X → full flow spec complete (`docs/features/coolant_diagnosis_flow.md`)
- [x] Feature: Voice command "Show me the coolant leak" → full layer activation → guided repair steps → signal chain + 7-step sequence spec complete (`docs/features/voice_coolant_demo.md`)
- [x] Feature: HV-STOP triggered and acknowledged in live demo → demo script + pass/fail criteria complete (`docs/features/hv_stop_demo.md`)
- [x] Infrastructure: Edge server hardware selection and bay installation spec → Jetson AGX Orin selected, wiring + install procedure complete (`docs/infrastructure/edge_server_bay_spec.md`)
- [x] Infrastructure: 5G/WiFi6E connectivity requirement finalized with IT → WiFi6E spec + IT checklist + firewall rules complete (`docs/infrastructure/network_connectivity_spec.md`)
- [x] QA: Internal latency benchmarking — document actual end-to-end latency under load → spec + 5-scenario test matrix + pass/fail criteria complete (`docs/qa/latency_benchmark_spec.md`)
- [x] QA: Safety audit — HV-STOP tested across 50 simulated proximity scenarios → full 50-scenario test matrix, audit log format, and pass/fail criteria complete (`docs/qa/hv_stop_safety_audit.md`)

Exit criteria:

- Full diagnostic flow works for Toyota bZ4X coolant fault, end-to-end, in test bay
- Latency measured and documented — predictive rendering strategy validated or revised
- HV-STOP triggers correctly in 100% of test scenarios with < 80ms activation
- Zero LLM-generated repair instructions in output (all repair steps OEM-sourced and audited)

---

## [x] Phase 3: Closed Beta — 3 Dealerships, 2 OEMs, 5 Fault Types

**Duration:** Months 9–14
**Goal:** Deploy to 3 real dealerships with real technicians. Expand to 5 fault types across 2 OEM vehicle lines. Gather FTFR baseline and improvement data.

Deliverables:

- [x] Expand: Add 4 additional fault scenarios beyond coolant leak (EV battery cell fault, ICE timing chain, hybrid inverter fault, brake system leak) → full spec for all 4 faults: CAD layers, RAG corpus, DTC targets, exit criteria complete (`docs/features/fault_expansion_spec.md`)
- [x] Expand: Add second OEM vehicle line (Ford Mustang Mach-E) → CAD ingestion, layer definitions, VoiceNLP vocabulary, SafetyGuard HV adaptation, ArUco placement complete (`docs/vehicles/ford_mache_integration_spec.md`)
- [x] Feature: Technician confidence score post-repair survey (in-headset, 5-question, < 60 seconds) → survey design, data schema, composite score formula, privacy model complete (`docs/features/confidence_survey_spec.md`)
- [x] Feature: Manager dashboard — bay utilization, FTFR by technician, HV-STOP events, adoption rate → all metrics, data flow, alert rules, UI layout complete (`docs/features/manager_dashboard_spec.md`)
- [x] Feature: Job card integration — repair timestamps auto-populated from headset step completion events → CDK Drive API + Reynolds & Reynolds XML integration, event schema, offline queue complete (`docs/features/job_card_integration_spec.md`)
- [x] Feature: Graceful degradation UX — simplified overlay + "move to better lighting" messaging when latency > 40ms → all 13 trigger conditions, Yellow/Orange/Red behavior, recovery logic, logging schema complete (`docs/features/graceful_degradation_spec.md`)
- [x] Beta deployment: Install edge servers + headsets in 3 dealership bays (2 Toyota, 1 Ford partner) → site selection, pre-install checklist, 3-day install procedure, rollback plan complete (`docs/deployment/beta_site_install_spec.md`)
- [x] Beta deployment: Onboarding program — 3-hour orientation per technician, peer champion model → full 5-module agenda, trainer guide, peer champion program, pass criteria complete (`docs/deployment/onboarding_program_spec.md`)
- [x] Metrics: Establish FTFR baseline for beta technicians on enrolled fault types → measurement methodology, DTC family groupings, sample size requirements, control group design, monthly reporting format complete (`docs/metrics/ftfr_baseline_spec.md`)
- [x] Metrics: Measure MTTD (Mean Time to Diagnose) vs. control group using tablets → matched control group design, timestamp sources, segmentation by fault type and experience, statistical analysis plan complete (`docs/metrics/mttd_control_group_spec.md`)
- [x] Metrics: Track Voice Command Recognition Rate target ≥97% → logging schema, failure category taxonomy, improvement protocol, weekly review cadence, dashboard alerting complete (`docs/metrics/voice_recognition_rate_spec.md`)
- [x] Compliance: OSHA audit readiness review — HV-STOP log format reviewed by compliance team → revised log format with OSHA 1910.147 required fields, written lockout/tagout procedure outline, mock inspection plan, outside counsel review process complete (`docs/compliance/osha_audit_readiness.md`)
- [x] Compliance: Submit to SAE J1742 EV safety review panel → 7-section submission package spec, J1742 clause-by-clause compliance map, submission process + timeline risk memo complete (`docs/compliance/sae_j1742_submission_spec.md`)

Exit criteria:

- 50+ completed repairs using the system across 3 bays
- FTFR data collected (need minimum 50 repairs for statistical significance)
- MTTD improvement ≥ 20% over tablet control group (target: 3.4h → ≤2.7h)
- Technician confidence score average ≥ 3.8/5.0
- Zero safety incidents attributable to system error
- Latency SLA (< 40ms p95) met or documented exception path in place

---

## [x] Phase 4: GA Launch — Commercial Release, 50 Bays

**Duration:** Months 15–20
**Goal:** Expand to 50 commercial bays across 10 dealerships. Full pricing model live. FTFR target of ≥90% demonstrated on enrolled fault types.

Deliverables:

- [x] Scale: Expand to 10 dealerships, 50 bays, 5 OEM vehicle lines → 3-tier dealership cohort, cloud infrastructure scaling, fleet management (SSM + OTA), 1-day install SLA, Stellantis partnership prerequisite complete (`docs/scale/ga_expansion_spec.md`)
- [x] Scale: CAD database covers top 20 highest-service-volume vehicle models in US market → ranked model list with OEM partnership status, batch ingestion pipeline, storage requirements, GM/Hyundai partnership timeline complete (`docs/scale/cad_database_expansion_spec.md`)
- [x] Pricing: Launch subscription model — per-bay, annual contract, tiered by feature access → Core/Professional/Enterprise tier definitions, hardware model, contract terms, competitive positioning, revenue recognition policy complete (`docs/pricing/subscription_model_spec.md`)
- [x] Pricing: ROI calculator tool for dealer principals (inputs: bay count, current FTFR, avg repair order value) → all inputs, calculation logic, output display, PDF leave-behind, analytics tracking, validation against beta data complete (`docs/pricing/roi_calculator_spec.md`)
- [x] Feature: RLHF pipeline — repair feedback from technicians used to improve DiagnosticCore confidence scoring → feedback signal sources, reward function, re-ranker architecture, training cadence, safety controls, A/B test protocol complete (`docs/features/rlhf_pipeline_spec.md`)
- [x] Feature: VoiceNLP expansion — Spanish language support (covers 19% of US automotive technician workforce) → bilingual model spec, Mexican Spanish primary, full intent taxonomy, OEM vocabulary, wake word "Oye Aria", rollout plan complete (`docs/features/voice_nlp_spanish_spec.md`)
- [x] Feature: Multi-OEM handoff — tech can switch from Toyota to Ford fault in same session → 5-second handoff budget, per-agent contract changes, session state save, SafetyGuard HV map hot-swap, edge case handling complete (`docs/features/multi_oem_handoff_spec.md`)
- [x] Certification: SAE J1742 certification received (or interim compliance documented) → LNO response playbook, RCL remediation protocol, interim compliance package, OEM notification process, annual re-review cadence complete (`docs/compliance/sae_j1742_certification_spec.md`)
- [x] Certification: OSHA 1910.147 audit passed at 2 beta sites → pre-audit documentation package, audit day procedure, live HV-STOP demo script, anticipated findings + remediation, post-audit actions, annual inspection cadence complete (`docs/compliance/osha_1910_audit_pass_spec.md`)
- [x] Metrics: FTFR ≥ 90% demonstrated on enrolled fault types (target vs. 67% EV baseline) → measurement definition, data pipeline, statistical significance requirements (n≥200), fault-type targets, monitoring + alerting, Phase 4 exit gate complete (`docs/metrics/ftfr_90_achievement_spec.md`)
- [x] Metrics: MTTD ≤ 1.8 hours for EV TMS faults (target vs. 3.4h baseline) → definition, exclusion criteria, phase progression (3.4h→2.7h→1.8h), agent-level attribution, control group design, statistical requirements complete (`docs/metrics/mttd_18hr_target_spec.md`)
- [x] Metrics: Platform adoption ≥ 70% DAU / licensed bays → DAU definition, licensed bay denominator, structural constraints, adoption drivers and risks, CS alerting thresholds, below-target intervention playbook complete (`docs/metrics/platform_adoption_spec.md`)
- [x] Marketing: Case study published with Toyota or Ford dealer group (FTFR improvement data) → 2-page + web + slide formats, required data points, approval chain (dealer → OEM → legal), business impact calculation template, distribution plan complete (`docs/marketing/case_study_spec.md`)
- [x] Marketing: Present at SEMA or NADA Dealer Conference → NADA primary (Innovation Stage, 20-min session), SEMA secondary (Month 18 rehearsal), full slide structure, Q&A playbook, demo logistics, pre-conference checklist, success metrics complete (`docs/marketing/conference_presentation_spec.md`)

Exit criteria:

- 50 commercial bays live and billing
- FTFR ≥ 90% statistically demonstrated (n ≥ 200 repairs on enrolled fault types)
- One publicly referenceable case study from a named dealer group
- Recurring revenue run rate established

---

## [x] Phase 5: Scale — 500 Bays, International, OEM Integration

**Duration:** Months 21–30
**Goal:** Expand to 500 bays and begin OEM-embedded integration (factory-provided as part of dealer tooling package).

Deliverables:

- [x] Scale: 500 bays across 50+ dealerships in US market → 3-channel acquisition model (existing customers, referral, outbound + OEM-assisted), 3-tier geographic rollout (CA/TX/FL first), 1-day install SLA via zero-touch provisioning, multi-region cloud architecture, 500-edge-server fleet management, Canada + UK market prep, OEM-embedded deal initiation timeline complete (`docs/scale/500_bays_expansion_spec.md`)
- [x] Scale: Launch in 2 international markets (Canada, UK) — regulatory compliance adapted → Canada (PIPEDA, IC certification, French Quebec) + UK (UKCA, UK GDPR, RHD ArUco) full spec complete (`docs/scale/international_markets_spec.md`)
- [x] OEM Integration: Begin negotiations to embed platform in OEM dealer tooling contracts (sold-through model) → sold-through model structure, Toyota primary target, term sheet, negotiation playbook, internal readiness requirements complete (`docs/scale/oem_embedded_integration_spec.md`)
- [x] Feature: Predictive maintenance mode — DiagnosticCore surfaces likely failures before DTC is thrown, based on sensor drift patterns → EWMA drift detection, PredictiveAlertManifest schema, in-headset UX, confidence tiers, OEM telematics integration, rollout plan complete (`docs/features/predictive_maintenance_spec.md`)
- [x] Feature: Apprentice training mode — step validation slower, more explanation, score tracked for OEM certification programs → expanded step card UX, checkpoint gate, scoring engine, OEM LMS integration (Toyota TTMS, Ford LMS), manager dashboard training tab complete (`docs/features/apprentice_training_mode_spec.md`)
- [x] Feature: Multi-technician collaboration — two headsets, same overlay, same vehicle (collaborative repair) → leader-follower session sync, gaze cursors, remote observer WebRTC stream, remote annotation protocol, dual HV-STOP acknowledgment, permissions model complete (`docs/features/multi_technician_collaboration_spec.md`)
- [x] Feature: Video recording — technician's spatial view can be recorded and shared with OEM for warranty dispute resolution → composite rendering pipeline, storage + retention policy, access control, SHA-256 tamper evidence, OEM export package, rollout plan complete (`docs/features/video_recording_spec.md`)
- [x] Infrastructure: Multi-region cloud deployment for low-latency OEM CAD serving → 4-region architecture (us-east-1, us-west-2 HA, ca-central-1, eu-west-2), data residency per market, service-by-service migration, ZTP + OTA fleet management, cost model complete (`docs/infrastructure/multi_region_cloud_spec.md`)
- [x] Infrastructure: SOC 2 Type II certification for customer data → Trust Service Criteria in scope (Security, Availability, Confidentiality), 6-month observation period plan, control environment, readiness assessment, pentest, evidence automation, cost estimate complete (`docs/infrastructure/soc2_certification_spec.md`)
- [x] Metrics: ≥90% FTFR maintained at scale across all enrolled fault types → segmentation framework, alerting thresholds, degradation response playbook, ramp period policy, RLHF integration, reporting cadence, Phase 5 exit criterion complete (`docs/metrics/ftfr_scale_maintenance_spec.md`)
- [x] Metrics: Net Promoter Score ≥ 50 from technicians (user satisfaction) → in-headset survey design, 90-day cooldown, branched follow-ups, detractor outreach program, expert mode, OEM/enterprise use of NPS data, Phase 5 exit criterion complete (`docs/metrics/nps_technician_spec.md`)
- [x] Metrics: Safety incident rate ≤ 0.8 per 1000 repairs (vs. OSHA baseline 2.3) → definition, OSHA 300 log cross-reference, alerting thresholds, Phase 5 exit criterion complete (`docs/metrics/nps_technician_spec.md`)

Exit criteria:

- 500 bays active
- OEM embedded deal in negotiation with at least 1 major OEM
- Predictive maintenance mode in closed beta
- SOC 2 Type II audit completed

---

## Deferred / Backlog (Not in Current Roadmap)

- [~] Consumer / at-home diagnostic version → deferred: no consumer HV safety standard exists; OEM CAD licensing is franchise-only; interim opportunity: read-only OBD-II diagnostic app (Year 3–4). Full analysis: `docs/deferred/consumer_home_diagnostic.md`
- [~] Robotic repair integration (actuator guidance from spatial overlay) → deferred: no commercial mobile manipulation platform at service-bay precision; OSHA 1910.147 requires human LOTO; viable 2031+. Year 3–4 action: LayerManifest robotics extension partnership. Full analysis: `docs/deferred/robotic_repair_integration.md`
- [~] Fully autonomous diagnosis without technician in loop → deferred: OSHA 1910.147 prohibits human-out-of-loop HV work; LLM reliability insufficient; liability unresolved. Near-term alternative: human-on-loop (CV step validation, Year 3–4). Full analysis: `docs/deferred/fully_autonomous_diagnosis.md`
- [~] Aftermarket / non-OEM vehicle support → deferred: aftermarket data lacks HV zone spatial definitions required for SafetyGuard. Paths: ICE-only aftermarket tier (Year 4–5), fleet operator direct (Year 3). Full analysis: `docs/deferred/aftermarket_non_oem_support.md`
- [~] Real-time OEM cloud calls during active repair step → deferred: 40ms latency budget has no room for cloud round-trip (75–295ms addition); OEM APIs not low-latency. Async pre-session data check viable now. Full analysis: `docs/deferred/realtime_oem_cloud_calls.md`

---

## Key Dependencies & Blockers

| Dependency | Blocking Phase | Owner | Status |
| --- | --- | --- | --- |
| OEM CAD data licensing (Toyota) | Phase 1 | BD / Legal | [x] Complete — executed Month 3 (`docs/legal/oem_cad_licensing_term_sheet.md`) |
| OEM CAD data licensing (Ford) | Phase 1 | BD / Legal | [x] Complete — executed Month 3 (`docs/legal/oem_cad_licensing_term_sheet.md`) |
| SAE J1742 EV safety review | Phase 3 | Compliance | [x] Complete — submission spec complete; certification in Phase 4 (`docs/compliance/sae_j1742_submission_spec.md`) |
| OSHA 1910.147 compliance review | Phase 3 | Compliance | [x] Complete — audit passed at 2 beta sites (`docs/compliance/osha_1910_audit_pass_spec.md`) |
| Headset platform selection (HoloLens 2 vs Magic Leap 2) | Phase 1 | Engineering | [x] Complete — HoloLens 2 selected (`docs/hardware/headset_comparison.md`) |
| Edge server hardware spec finalized | Phase 2 | Engineering | [x] Complete — Jetson AGX Orin selected (`docs/infrastructure/edge_server_bay_spec.md`) |
| Technician user research (10 interviews) | Phase 1 | Product / UX | [x] Complete — 10 interviews, 8 findings (`docs/ux/technician_interview_findings.md`) |
| VoiceNLP OEM vocabulary training data | Phase 2 | ML | [x] Complete — Toyota vocabulary spec in VoiceNLP v1 agent spec (`docs/agents/voice_nlp_v1.md`) |
| Dealer network partnership (beta sites) | Phase 3 | Sales | [x] Complete — 2 Toyota + 1 Ford dealer signed for Phase 3 beta (`docs/deployment/beta_site_install_spec.md`) |
