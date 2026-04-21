# Phase 4 GA Launch Checklist — Operational Readiness

**Target launch date:** Month 15
**Owner:** VP Product + VP Engineering
**Sign-off required from:** CEO, CTO, Head of Legal, Compliance Manager

---

## How to Use This Checklist

Every item must be ✓ before GA launch proceeds. Any item marked ✗ or ? becomes a tracked blocker. Launch date slips if a P0 blocker is not resolved. P1 blockers may be accepted with a documented mitigation plan.

| Priority | Description |
|---|---|
| P0 | Launch cannot proceed until resolved |
| P1 | Launch can proceed with documented mitigation plan |
| P2 | Should be resolved within 30 days post-launch |

---

## BLOCK 1: Safety and Compliance *(all P0)*

- [ ] SAE J1742 certification received (or interim compliance package approved by outside counsel)
- [ ] OSHA 1910.147 audit passed at ≥ 2 sites
- [ ] HV-STOP 50-scenario test matrix: 100% pass rate confirmed (Phase 2 and Phase 3 data)
- [ ] Zero safety incidents in Phase 3 beta — confirmed in Phase 3 Results Report
- [ ] SafetyGuard process isolation verified on production edge server firmware
- [ ] HV-STOP activation latency confirmed < 80ms p99 in production hardware
- [ ] Safety architecture document reviewed and signed off by outside safety counsel
- [ ] OSHA audit log format reviewed and approved by compliance manager
- [ ] HV-STOP override PIN provisioning process documented and tested

---

## BLOCK 2: Product Quality *(P0 unless noted)*

- [ ] FTFR ≥ 90% demonstrated on all 5 enrolled fault types (Phase 3 data, n ≥ 50)
- [ ] Latency p95 < 40ms confirmed on production hardware under representative load
- [ ] Voice recognition rate ≥ 97% on all production sites tested
- [ ] All 5 agents (DiagnosticCore, SensorFusion, CADRenderer, VoiceNLP, SafetyGuard) at v1 spec
- [ ] Zero LLM-generated repair instructions: code audit signed off by Engineering Lead
- [ ] Graceful degradation tested across all 13 trigger conditions
- [ ] Multi-OEM handoff (Toyota ↔ Ford) tested: < 5 second handoff budget confirmed
- [ ] Expert Mode tested with ≥ 5 sessions per enrolled fault type *(P1)*
- [ ] Spanish VoiceNLP: "Oye Aria" wake word + full Toyota vocabulary tested in noisy bay conditions *(P1)*
- [ ] RLHF pipeline: feedback ingestion tested end-to-end (not required for first 30 days post-launch) *(P2)*

---

## BLOCK 3: Infrastructure *(P0 unless noted)*

- [ ] Edge server production firmware: all 5 agents deployed, health checks passing
- [ ] Zero-touch provisioning (ZTP) tested on 5 new edge servers end-to-end
- [ ] WiFi6E network spec confirmed at all Phase 4 launch sites (pre-install site survey complete)
- [ ] Cloud infrastructure: us-east-1 all services healthy; CloudWatch dashboards configured
- [ ] Kinesis telemetry pipeline: events flowing end-to-end from bay to S3 confirmed
- [ ] CAD cache sync tested: edge server receives OEM CAD update within 7 days of bulletin publish
- [ ] OTA update pipeline tested: staged rollout (5% → 20% → 100%) with rollback confirmed *(P1)*
- [ ] us-west-2 HA failover tested: RTO < 5 minutes confirmed *(P1)*
- [ ] Multi-region infrastructure (ca-central-1, eu-west-2) NOT required for Phase 4 US-only launch *(skip)*

---

## BLOCK 4: Pricing and Commercial *(P0)*

- [ ] Subscription pricing tiers (Core / Professional / Enterprise) approved by CEO and Finance
- [ ] ROI calculator: web version live; PDF export working; tested with 5 representative dealer inputs
- [ ] Hardware pricing ($5,200 bundle) approved; purchase order process documented
- [ ] Master Services Agreement (MSA) template approved by legal
- [ ] Data Processing Agreement (DPA) template approved by legal (CCPA compliance for CA dealers)
- [ ] Annual contract template with auto-renew and 90-day cancellation notice approved
- [ ] Revenue recognition policy documented (ASC 606 — SaaS subscription treatment confirmed with auditors)
- [ ] Billing system configured: Stripe or equivalent; invoice generation tested

---

## BLOCK 5: Customer-Facing *(P0 unless noted)*

- [ ] Manager dashboard: all Phase 3 features live (FTFR, adoption, HV-STOP, tech confidence)
- [ ] Manager dashboard daily digest email: tested with 3 test accounts
- [ ] Job card integration: CDK Drive API tested with CDK sandbox environment
- [ ] Job card integration: Reynolds & Reynolds XML tested with R&R test environment
- [ ] Onboarding program: 3-hour agenda finalized; trainer guide printed and distributed
- [ ] Peer champion program: selection criteria and recognition program documented
- [ ] Support portal live: ticket submission, SLA tracking, escalation path configured
- [ ] Support SLA by tier (Enterprise 1h, Professional 4h, Core 8h) documented in MSA *(P0)*
- [ ] Help center: top 20 FAQs published *(P1)*
- [ ] In-app feedback widget *(P2)*

---

## BLOCK 6: Sales and Marketing *(P1 unless noted)*

- [ ] Sales team: ≥ 2 AEs hired and onboarded (Hires 13 and 14 from hiring plan)
- [ ] AE onboarding complete: product demo certified, objection handling trained
- [ ] ROI calculator: AE-facing version with editable assumptions live
- [ ] Phase 3 case study: published (or approved for publication at GA launch)
- [ ] Website: pricing page live; case study linked; demo request form configured
- [ ] OEM training manager briefing: Toyota and Ford regional training managers briefed on GA and pricing
- [ ] NADA Dealer Conference submission: Innovation Stage application submitted (Month 15 deadline)
- [ ] CRM: Salesforce (or equivalent) configured for dealer pipeline tracking
- [ ] CS health score: automated scoring pipeline live in Salesforce

---

## BLOCK 7: Legal and Data *(P0)*

- [ ] OEM CAD licensing agreements: Toyota and Ford executed (2 of 3 minimum for Phase 1 exit)
- [ ] Privacy policy: published at xvd.com/privacy (covers CCPA for CA; baseline for all US states)
- [ ] Terms of Service: published and linked in MSA
- [ ] Data governance policy: approved by Legal and documented
- [ ] All tech IDs and VINs stored with encryption confirmed (AES-256)
- [ ] HV-STOP audit logs: retention policy (3 years) configured in S3 lifecycle policy
- [ ] Recording storage: access control matrix confirmed; dealer data not accessible to other dealers

---

## BLOCK 8: Go / No-Go Decision

**GA launch approved when:**
- All P0 items: ✓
- All P1 items: ✓ or documented mitigation plan accepted by CEO

**Launch date confirmed by:** CEO, CTO, Head of Legal, Compliance Manager (all must sign)

**Rollback plan:** If a critical issue is discovered within 30 days of GA:
1. Affected fault types suspended (not full platform shutdown) — techs revert to tablet procedure for that fault type
2. Safety-related issue: full platform suspension at affected site; Compliance Manager notified within 1 hour
3. Data-related issue: DPA clause invoked; affected dealers notified within 72 hours (CCPA requirement)

---

## Post-Launch Monitoring (First 30 Days)

Daily review by Product + Engineering:
- FTFR on first 20 commercial repairs (must be ≥ 87% — warning threshold)
- Latency p95 (must be < 40ms)
- Voice recognition rate (must be ≥ 97%)
- HV-STOP events (any false positive triggers immediate investigation)
- Support ticket volume (> 5 P1 tickets/day = incident declared)

Week 4 retrospective: review all metrics; decide if Phase 4 ramp continues or pause is needed.
