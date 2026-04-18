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

## [ ] Phase 1: Foundation — Data Partnerships & Prototype Hardware

**Duration:** Months 1–4
**Goal:** Secure the two non-negotiable prerequisites — OEM CAD data licensing and headset hardware — and build a non-functional prototype to validate the spatial overlay concept with real technicians.

Deliverables:
- [ ] Legal: Sign CAD data licensing agreements with 3 launch OEM partners (target: Toyota, Ford, Stellantis)
- [ ] Legal: Define OSHA/SAE J1742 compliance framework with outside counsel
- [ ] Hardware: Select headset platform (HoloLens 2 vs. Magic Leap 2 bakeoff — 2-week evaluation)
- [ ] Hardware: Procure 10 developer units for internal prototyping
- [ ] Data: Stand up CAD ingestion pipeline (OEM CAD → normalized 3D mesh → VIN-indexed cache)
- [ ] Data: Build OBD-II / CAN bus integration test harness (real vehicles, test bay)
- [ ] UX Research: Conduct 10 contextual interviews with service technicians at 3 dealerships
- [ ] UX Research: Validate "Tri-Modal Hands-Free" model with paper prototype and Wizard of Oz simulation
- [ ] Prototype: Build non-interactive spatial overlay on single vehicle model (Toyota bZ4X) — no AI yet
- [ ] Prototype: Demonstrate CAD layer rendering anchored to physical vehicle via ArUco markers

Exit criteria:
- At least 2 OEM CAD licensing agreements signed
- Headset platform selected with documented rationale
- 10 technician interviews completed; key friction points documented
- Non-interactive overlay demo reviewed by product and engineering leadership

---

## [ ] Phase 2: Alpha — DiagnosticCore & SensorFusion MVP

**Duration:** Months 5–8
**Goal:** Build a functional end-to-end diagnostic flow for a single vehicle model and single fault scenario (EV battery coolant leak). Internal testing only — no external users.

Deliverables:
- [ ] Agent: Build DiagnosticCore v1 — DTC ingestion, RAG pipeline against OEM service bulletins, layer manifest generation
- [ ] Agent: Build SensorFusion v1 — OBD-II + thermal camera integration, UnifiedSensorState output
- [ ] Agent: Build CADRenderer v1 — layer budget enforcement, SLAM anchoring, LOD management
- [ ] Agent: Build VoiceNLP v1 — wake word, intent classification, entity extraction (Toyota vocabulary)
- [ ] Agent: Build SafetyGuard v1 — HV proximity detection, HV-STOP protocol, audit logging
- [ ] Feature: End-to-end flow for "coolant leak diagnosis" on Toyota bZ4X
- [ ] Feature: Voice command "Show me the coolant leak" → full layer activation → guided repair steps
- [ ] Feature: HV-STOP triggered and acknowledged in live demo
- [ ] Infrastructure: Edge server hardware selection and bay installation spec
- [ ] Infrastructure: 5G/WiFi6E connectivity requirement finalized with IT
- [ ] QA: Internal latency benchmarking — document actual end-to-end latency under load
- [ ] QA: Safety audit — HV-STOP tested across 50 simulated proximity scenarios

Exit criteria:
- Full diagnostic flow works for Toyota bZ4X coolant fault, end-to-end, in test bay
- Latency measured and documented — predictive rendering strategy validated or revised
- HV-STOP triggers correctly in 100% of test scenarios with < 80ms activation
- Zero LLM-generated repair instructions in output (all repair steps OEM-sourced and audited)

---

## [ ] Phase 3: Closed Beta — 3 Dealerships, 2 OEMs, 5 Fault Types

**Duration:** Months 9–14
**Goal:** Deploy to 3 real dealerships with real technicians. Expand to 5 fault types across 2 OEM vehicle lines. Gather FTFR baseline and improvement data.

Deliverables:
- [ ] Expand: Add 4 additional fault scenarios beyond coolant leak (EV battery cell fault, ICE timing chain, hybrid inverter fault, brake system leak)
- [ ] Expand: Add second OEM vehicle line (Ford Mustang Mach-E)
- [ ] Feature: Technician confidence score post-repair survey (in-headset, 5-question, < 60 seconds)
- [ ] Feature: Manager dashboard — bay utilization, FTFR by technician, HV-STOP events, adoption rate
- [ ] Feature: Job card integration — repair timestamps auto-populated from headset step completion events
- [ ] Feature: Graceful degradation UX — simplified overlay + "move to better lighting" messaging when latency > 40ms
- [ ] Beta deployment: Install edge servers + headsets in 3 dealership bays (2 Toyota, 1 Ford partner)
- [ ] Beta deployment: Onboarding program — 3-hour orientation per technician, peer champion model
- [ ] Metrics: Establish FTFR baseline for beta technicians on enrolled fault types
- [ ] Metrics: Measure MTTD (Mean Time to Diagnose) vs. control group using tablets
- [ ] Metrics: Track Voice Command Recognition Rate target ≥97%
- [ ] Compliance: OSHA audit readiness review — HV-STOP log format reviewed by compliance team
- [ ] Compliance: Submit to SAE J1742 EV safety review panel

Exit criteria:
- 50+ completed repairs using the system across 3 bays
- FTFR data collected (need minimum 50 repairs for statistical significance)
- MTTD improvement ≥ 20% over tablet control group (target: 3.4h → ≤2.7h)
- Technician confidence score average ≥ 3.8/5.0
- Zero safety incidents attributable to system error
- Latency SLA (< 40ms p95) met or documented exception path in place

---

## [ ] Phase 4: GA Launch — Commercial Release, 50 Bays

**Duration:** Months 15–20
**Goal:** Expand to 50 commercial bays across 10 dealerships. Full pricing model live. FTFR target of ≥90% demonstrated on enrolled fault types.

Deliverables:
- [ ] Scale: Expand to 10 dealerships, 50 bays, 5 OEM vehicle lines
- [ ] Scale: CAD database covers top 20 highest-service-volume vehicle models in US market
- [ ] Pricing: Launch subscription model — per-bay, annual contract, tiered by feature access
- [ ] Pricing: ROI calculator tool for dealer principals (inputs: bay count, current FTFR, avg repair order value)
- [ ] Feature: RLHF pipeline — repair feedback from technicians used to improve DiagnosticCore confidence scoring
- [ ] Feature: VoiceNLP expansion — Spanish language support (covers 19% of US automotive technician workforce)
- [ ] Feature: Multi-OEM handoff — tech can switch from Toyota to Ford fault in same session
- [ ] Certification: SAE J1742 certification received (or interim compliance documented)
- [ ] Certification: OSHA 1910.147 audit passed at 2 beta sites
- [ ] Metrics: FTFR ≥ 90% demonstrated on enrolled fault types (target vs. 67% EV baseline)
- [ ] Metrics: MTTD ≤ 1.8 hours for EV TMS faults (target vs. 3.4h baseline)
- [ ] Metrics: Platform adoption ≥ 70% DAU / licensed bays
- [ ] Marketing: Case study published with Toyota or Ford dealer group (FTFR improvement data)
- [ ] Marketing: Present at SEMA or NADA Dealer Conference

Exit criteria:
- 50 commercial bays live and billing
- FTFR ≥ 90% statistically demonstrated (n ≥ 200 repairs on enrolled fault types)
- One publicly referenceable case study from a named dealer group
- Recurring revenue run rate established

---

## [ ] Phase 5: Scale — 500 Bays, International, OEM Integration

**Duration:** Months 21–30
**Goal:** Expand to 500 bays and begin OEM-embedded integration (factory-provided as part of dealer tooling package).

Deliverables:
- [ ] Scale: 500 bays across 50+ dealerships in US market
- [ ] Scale: Launch in 2 international markets (Canada, UK) — regulatory compliance adapted
- [ ] OEM Integration: Begin negotiations to embed platform in OEM dealer tooling contracts (sold-through model)
- [ ] Feature: Predictive maintenance mode — DiagnosticCore surfaces likely failures before DTC is thrown, based on sensor drift patterns
- [ ] Feature: Apprentice training mode — step validation slower, more explanation, score tracked for OEM certification programs
- [ ] Feature: Multi-technician collaboration — two headsets, same overlay, same vehicle (collaborative repair)
- [ ] Feature: Video recording — technician's spatial view can be recorded and shared with OEM for warranty dispute resolution
- [ ] Infrastructure: Multi-region cloud deployment for low-latency OEM CAD serving
- [ ] Infrastructure: SOC 2 Type II certification for customer data
- [ ] Metrics: ≥90% FTFR maintained at scale across all enrolled fault types
- [ ] Metrics: Net Promoter Score ≥ 50 from technicians (user satisfaction)
- [ ] Metrics: Safety incident rate ≤ 0.8 per 1000 repairs (vs. OSHA baseline 2.3)

Exit criteria:
- 500 bays active
- OEM embedded deal in negotiation with at least 1 major OEM
- Predictive maintenance mode in closed beta
- SOC 2 Type II audit completed

---

## Deferred / Backlog (Not in Current Roadmap)

- [ ] Consumer / at-home diagnostic version (out of scope — requires different safety framework and licensing model)
- [ ] Robotic repair integration (actuator guidance from spatial overlay — 5+ year horizon)
- [ ] Fully autonomous diagnosis without technician in loop (regulatory and liability barriers — not viable pre-2030)
- [ ] Aftermarket / non-OEM vehicle support (CAD data cannot be sourced reliably without OEM partnership)
- [ ] Real-time OEM cloud calls during active repair step (latency risk unacceptable; all data must be edge-cached)

---

## Key Dependencies & Blockers

| Dependency | Blocking Phase | Owner | Status |
|---|---|---|---|
| OEM CAD data licensing (Toyota) | Phase 1 | BD / Legal | [ ] Not started |
| OEM CAD data licensing (Ford) | Phase 1 | BD / Legal | [ ] Not started |
| SAE J1742 EV safety review | Phase 3 | Compliance | [ ] Not started |
| OSHA 1910.147 compliance review | Phase 3 | Compliance | [ ] Not started |
| Headset platform selection (HoloLens 2 vs Magic Leap 2) | Phase 1 | Engineering | [ ] Not started |
| Edge server hardware spec finalized | Phase 2 | Engineering | [ ] Not started |
| Technician user research (10 interviews) | Phase 1 | Product / UX | [ ] Not started |
| VoiceNLP OEM vocabulary training data | Phase 2 | ML | [ ] Not started |
| Dealer network partnership (beta sites) | Phase 3 | Sales | [ ] Not started |
