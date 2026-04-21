# Conference Presentation Spec — NADA and SEMA
## Step 49 — Phase 4, Marketing 2

**Roadmap reference:** Phase 4, Marketing 2
**Status:** `[ ]` Not started
**Owner:** VP Product + Head of Marketing + Sales Lead
**Target:** Primary conference presentation delivered by Month 20
**Prerequisite:** Case study published (`docs/marketing/case_study_spec.md`); FTFR ≥ 90% data available; demo environment stable

---

## Objective

Present X-Ray Vision Diagnostics at a major automotive industry conference to establish market presence, generate dealer prospect pipeline, and build OEM partnership credibility. Conference presentation is the highest-leverage earned marketing channel for the B2B dealer market — dealer principals and service directors attend in decision-making capacity.

---

## Conference Selection

### Primary Target: NADA Annual Convention & Expo

| Attribute | Detail |
|---|---|
| Full name | National Automobile Dealers Association Annual Convention & Expo |
| Typical date | January (Month 20 of roadmap aligns with next NADA after Phase 4 launch) |
| Audience | Dealer principals, GMs, VP Service, VP Fixed Operations — the budget owners |
| Attendance | ~20,000 attendees; 500+ exhibitors |
| Relevance | The premier franchise dealer event; where dealer-level technology purchasing decisions are influenced |
| Format options | Exhibitor booth, Innovation Stage speaking slot, Dealer 20 Group presentation |

**Why NADA is the priority:** NADA's audience is decision-makers, not technicians. A dealer principal who sees a compelling FTFR and ROI story can authorize deployment for their entire dealer group. One NADA conversation has a higher revenue ceiling than any trade publication ad. SEMA is better for brand awareness among technicians and aftermarket audience; NADA closes deals.

### Secondary Target: SEMA Show (Las Vegas)

| Attribute | Detail |
|---|---|
| Full name | Specialty Equipment Market Association Show |
|  Typical date | October/November (Month 18–19) |
| Audience | Aftermarket suppliers, enthusiasts, shop owners, technicians, automotive press |
| Attendance | ~160,000 attendees |
| Relevance | High technician and automotive media attendance; earlier calendar than NADA |
| Format | Exhibitor booth in EV/Technology section; possible SEMA Electrified Stage speaking slot |

**Why SEMA is secondary:** SEMA audience skews aftermarket and consumer — less aligned with our B2B franchise dealer model. However, SEMA provides earlier media exposure, industry credibility, and a proving ground for live demo logistics before the higher-stakes NADA.

**Recommendation:** Present at SEMA (Month 18) for rehearsal + press exposure, then NADA (Month 20) for deal-making. If budget constraints require choosing one: NADA.

---

## NADA Presentation Plan

### Format: Innovation Stage Speaking Slot (Preferred)

NADA's Innovation Stage hosts 20-minute sessions with Q&A. This is the highest-value format — front-of-stage visibility to several hundred dealer principals who opted into the technology track.

**Submission deadline:** NADA Innovation Stage applications typically due 6 months before the event. If Month 20 = January of Year 3, submission is due Month 14.

**Application requirements:**
- 300-word abstract + presenter bio
- 3 key audience takeaways
- Evidence of product viability (case study data, named customer reference)
- No overt product pitching — educational framing required

**Session title options:**
- "From 67% to 93%: How Spatial AI Is Fixing the EV First-Time Fix Problem"
- "What Happens When Your Technician Can See Through the Car"
- "The $450 Comeback Repair and How XR Eliminates It"

### Fallback Format: Exhibitor Booth (Table Stakes)

If Innovation Stage slot is not awarded: secure a 10×20 booth in the Service/Fixed Operations section. Booth serves as demo station and sales meeting room.

- Live demo at booth: rotating 20-minute headset demo every hour during show hours
- Appointment scheduling: pre-show outreach to target dealer groups for 30-min booth meetings
- Case study handout: printed 2-page case study at booth

---

## Presentation Content (Innovation Stage — 20 Minutes)

### Slide Structure

**Slides 1–2: The Problem (4 minutes)**
- Industry data: EV service is growing — X% of new vehicles sold will be EV/hybrid by 2027 (IEA data)
- The bottleneck: technician diagnostic complexity; FTFR of 67% for EV battery faults (cited source)
- The cost: $450 per comeback repair × nationwide volume = $X billion annual rework cost for US dealership industry
- The wrong solution: more training (time-consuming, doesn't scale); more scanners (same 2D data problem)

**Slides 3–4: The Solution (5 minutes)**
- What X-Ray Vision Diagnostics does: spatial overlay + AI diagnosis + voice-primary hands-free interface
- 60-second product demo video (recorded in controlled environment; live demo optional if logistics confirmed)
- Architecture: OEM-sourced data only; edge processing; SafetyGuard as non-bypassable safety layer
- Why spatial: the information the technician needs is IN the vehicle — show it where it is, not on a screen

**Slides 5–7: The Results (7 minutes)**
- Case study: [Dealer name or "A Toyota dealer in [Region]"] — before and after
  - FTFR: 67% → 93% (specific numbers from case study)
  - MTTD: 3.4h → 1.7h
  - Zero HV safety incidents during deployment
  - Technician quote: [verbatim from case study, with consent]
- Dealer principal quote or endorsement (if available)
- Business impact: annual rework cost avoided (example calculation)
- Adoption rate: ≥ 70% DAU across enrolled bays

**Slides 8–9: The Commercial Model + What's Next (3 minutes)**
- Pricing model: per-bay subscription, tiered by feature access (no detailed pricing on slides — conversation at booth)
- Deployment: 1-day installation, 3-hour technician orientation; 50 bays live today
- Roadmap preview: Spanish language support, 500-bay expansion, OEM-embedded integration (Phase 5)
- Call to action: "If your dealer group services EVs and your service director is still relying on a 2D scanner, we should talk. See us at Booth [X] or scan this QR code."

**Slide 10: Q&A Setup**
- Moderator takes questions
- Common anticipated questions (see below)

### Demo Options

| Option | Pros | Cons | Decision |
|---|---|---|---|
| Live headset demo on stage | Most compelling; audience sees the product | Requires SLAM-calibrated vehicle or mock vehicle prop; AV logistics; risk of demo failure | Only if stage setup allows vehicle in speaking area |
| Pre-recorded video (60 seconds) | Reliable; controllable; high production quality | Less visceral than live | Default for main session |
| Live demo at booth (post-session) | Smaller audience; real interaction | Requires enrolled vehicle in booth (vehicle on show floor or AR mock-up) | Yes — regardless of stage format |

**Demo minimum viable setup for booth:** A vehicle shell or vehicle prop with ArUco markers placed, connected to edge server mock, running a pre-configured Toyota bZ4X diagnostic scenario. CAD layers display over the prop vehicle. Thermal camera optional for booth demo.

### Anticipated Q&A

| Question | Prepared Answer |
|---|---|
| "What happens if the AI is wrong?" | "The system never generates repair instructions. Every step comes from OEM-certified service bulletins. The AI decides *which* step to show and *how* to show it spatially — not *what* the repair procedure is. LLM-generated repair content is architecturally prohibited." |
| "Is this FDA/OSHA approved?" | "We passed an OSHA 1910.147 lockout/tagout compliance audit at two live sites in [Month]. We're currently in review with the SAE J1742 EV safety panel — the same body that sets HV service standards for Toyota and Ford service networks." |
| "What does it cost?" | "Subscription pricing per bay, annual contract — we tailor the package to your bay count and OEM mix. Happy to share the numbers with your service director at our booth. We also have an ROI calculator that takes your current FTFR and repair order volume and shows the payback period." |
| "Does it require cloud connectivity?" | "No cloud connectivity required during an active repair. All data is processed on an edge server in your bay. This was a non-negotiable design requirement — we cannot have latency risk on an HV system diagnosis." |
| "Which vehicles does it support?" | "Toyota bZ4X, RAV4 Hybrid, Mustang Mach-E, F-150 Lightning, and Jeep Wrangler 4xe in Phase 4. We're adding the top 20 highest-service-volume models in Phase 5 — expanding through existing OEM data partnerships." |

---

## SEMA Presentation Plan (Secondary)

**Format:** SEMA Electrified Stage — 15-minute session in EV technology programming track

**Audience difference:** SEMA audience includes automotive press and influencers. Presentation skews more toward technology story and product vision than ROI and dealer operations.

**SEMA-specific framing:**
- Lead with the technology and user experience: "What it feels like to see through an EV"
- Product demo video should be longer (90 seconds) and more visually immersive
- Mention the technician story — the human element resonates with SEMA's culture
- Dealer ROI is secondary to product innovation narrative at SEMA

**Press outreach for SEMA:** Target automotive technology press (The Autopian, Automotive News, TechCrunch Transportation) with embargo announcement + demo access day before SEMA opens.

---

## Pre-Conference Checklist

**12 weeks before conference:**
- [ ] Submit Innovation Stage application (NADA) or Electrified Stage application (SEMA)
- [ ] Reserve booth space (backup plan)
- [ ] Begin demo environment setup (edge server + vehicle prop)

**8 weeks before:**
- [ ] Case study approved and printed (2,000 copies for NADA; 500 for SEMA)
- [ ] Demo video produced and reviewed by legal + dealer for accuracy
- [ ] Speaker prep sessions begin (3 × 60-min rehearsals)

**4 weeks before:**
- [ ] Pre-show outreach to target dealer groups for booth appointments (top 30 prospects)
- [ ] Demo environment tested in conference-like conditions (ambient noise, variable lighting)
- [ ] Press briefing materials prepared (press kit, case study, exec bio)

**1 week before:**
- [ ] Full run-through of stage presentation (timing, slides, Q&A)
- [ ] Demo equipment shipped and confirmed at venue
- [ ] Booth staff briefed and role-assigned (demo runner, sales lead, product expert)

---

## Success Metrics for Conference

| Metric | Target |
|---|---|
| Booth visits | ≥ 200 (NADA) / ≥ 150 (SEMA) |
| Qualified dealer meetings at booth | ≥ 20 (NADA) / ≥ 10 (SEMA) |
| Pipeline generated (post-conference) | ≥ 5 new opportunities ≥ 5-bay deals |
| Press coverage (earned media) | ≥ 3 trade publication mentions |
| Case study PDF distributed | ≥ 150 copies |
| Innovation Stage attendance (NADA) | ≥ 200 attendees in room |

---

## Links to Related Specs

- Case study (conference centerpiece): `docs/marketing/case_study_spec.md`
- ROI calculator (booth demo): `docs/pricing/roi_calculator_spec.md`
- Subscription pricing (sales conversations): `docs/pricing/subscription_model_spec.md`
- FTFR data (presentation metrics): `docs/metrics/ftfr_90_achievement_spec.md`
- OSHA compliance (Q&A readiness): `docs/compliance/osha_1910_audit_pass_spec.md`
- SAE J1742 status (Q&A readiness): `docs/compliance/sae_j1742_certification_spec.md`
