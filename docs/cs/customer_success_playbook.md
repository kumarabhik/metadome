# Customer Success Playbook

**Covers:** All paying accounts (Phase 4 onward)
**CS team scale:** 1 CS engineer per 50 bays (ratio maintained through Phase 5)

---

## CS Philosophy

X-Ray Vision Diagnostics is a safety-critical platform used in high-voltage environments. Customer success is not just about retention — a dealer whose techs stop using the headset because of a bad experience is a safety risk (they revert to unaided diagnosis). CS must keep the product in active use.

The CS team's single most important metric is **platform adoption rate ≥ 70% DAU / licensed bays.** Everything else follows from that.

---

## Account Segmentation

| Tier | Definition | CSM Ratio | Engagement Model |
|---|---|---|---|
| Enterprise | ≥ 5 bays OR OEM embedded | Dedicated CSM | Weekly check-in; QBR every quarter |
| Professional | 2–4 bays, direct | 1 CSM per 15 accounts | Monthly check-in; semi-annual review |
| Core | 1 bay, low engagement | 1 CSM per 40 accounts | Automated health score alerts; reactive support only |

---

## Customer Journey Map

### Phase 1: Pre-Installation (Week −2 to 0)

**Owner:** CS + Field Install team

Actions:
1. Send pre-install checklist to service manager (IT readiness, network requirements, bay prep) — 2 weeks before install date
2. Confirm headcount: how many techs will be trained at launch? Names + experience levels
3. Pre-assign Apprentice Mode profile if any techs are < 2 years experience
4. Confirm service manager has access to manager dashboard before install (account provisioned)

**Handoff from Sales:** CS receives account brief: bay count, tier, primary contact (service manager), secondary contact (dealer principal), techs to be trained.

---

### Phase 2: Install and Onboarding (Days 1–3)

**Owner:** Field Install team (Day 1) → CS (Days 2–3)

Day 1: Field install (per `docs/deployment/beta_site_install_spec.md`)
Day 2–3: Technician onboarding (per `docs/deployment/onboarding_program_spec.md`)

CS actions during onboarding week:
- Attend final hour of onboarding Day 1 (remote or in-person for Enterprise accounts)
- Set up manager dashboard: confirm bay assignments, tech profiles, alert thresholds
- Send welcome kit: support contact, escalation path, feedback channel
- Schedule 7-day check-in call

---

### Phase 3: Activation (Weeks 1–4)

**Goal:** ≥ 3 completed repairs in the first 7 days. This is the activation criterion — accounts that complete 3 repairs in Week 1 have dramatically higher 90-day retention.

**CS actions:**
- Day 7 check-in: review dashboard together with service manager. Key questions:
  - How many sessions started vs. completed?
  - Any voice command failures?
  - Any tech complaints about ergonomics or step pacing?
- If < 3 repairs in Week 1: trigger activation intervention (see below)
- Day 14: review FTFR data for first completed repairs; confirm system performing as expected

**Activation intervention:**
If a new account completes < 3 repairs in Week 1, CS schedules a "shadow session" — a CS engineer joins a live repair session (remote via observer stream or in person for Enterprise) to identify the blocker. Common blockers:
- Tech not wearing headset (forgot it exists; habit not yet formed)
- Voice commands not working in the specific bay (noise issue)
- Service manager hasn't added fault types to the enrolled list

---

### Phase 4: Adoption (Months 1–3)

**Goal:** Reach ≥ 70% DAU / licensed bays within 90 days.

**CS actions:**
- Monthly health score review (see Health Score below)
- Proactive outreach if adoption drops below 60% for 2 consecutive weeks
- Quarterly Business Review (QBR) for Enterprise accounts: present FTFR improvement data, compare to platform average, discuss roadmap features coming in next quarter

**QBR Agenda Template:**
1. Your FTFR data (last 90 days vs. your baseline)
2. MTTD comparison (your bays vs. platform average)
3. Tech engagement: NPS score for your team, voice recognition rate
4. Open issues / support tickets closed
5. Upcoming features relevant to your account
6. Renewal discussion (if within 6 months)

---

### Phase 5: Expansion and Renewal

**CS actions:**
- At Month 9 (3 months before annual renewal): expansion conversation
  - Are there additional bays / dealerships in the group that could benefit?
  - Is the dealer group ready to upgrade from Core → Professional or Professional → Enterprise?
  - For dealer groups with apprentice programs: pitch Apprentice Training Mode integration with OEM LMS
- Renewal: CS owns renewal for Professional/Core; AE owns renewal for Enterprise

---

## Health Score

CS monitors each account via an automated health score updated daily.

| Metric | Weight | Green | Yellow | Red |
|---|---|---|---|---|
| DAU / licensed bays | 30% | ≥ 70% | 50–70% | < 50% |
| FTFR (rolling 4 weeks) | 25% | ≥ 87% | 80–87% | < 80% |
| Voice recognition rate | 15% | ≥ 97% | 93–97% | < 93% |
| NPS (last survey) | 15% | ≥ 50 | 30–50 | < 30 |
| Support tickets open > 7 days | 10% | 0 | 1 | ≥ 2 |
| Days since last CS contact | 5% | ≤ 30 | 31–60 | > 60 |

**Composite score:** weighted average of above → Green (≥ 75), Yellow (50–74), Red (< 50)

**Red account protocol:** CS contacts account within 24 hours; escalate to CS Manager if no improvement in 2 weeks; escalate to VP Customer Success if no improvement in 4 weeks.

---

## Escalation Path

| Issue Type | L1 Response | Escalation |
|---|---|---|
| Headset not powering on | CS → Field Install (hardware dispatch) | Engineering if recurrent |
| Voice commands not working | CS troubleshooting guide; recalibrate in-app | ML team if > 3 bays affected |
| HV-STOP false positive (firing when no HV present) | Immediate escalation to Engineering; suspend HV fault types until resolved | Safety incident protocol |
| FTFR data missing / dashboard wrong | CS → Engineering data pipeline ticket | P1 if FTFR data gap > 48 hours |
| Dealer wants to cancel | CS retention conversation; involve AE if > 5 bays; involve VP CS if Enterprise | |

---

## Churn Prevention

**Early warning signals (trigger proactive CS outreach):**
- Adoption < 60% for 2 consecutive weeks
- NPS drops below 40 for any tech cohort
- No sessions completed in 7+ days (exclude scheduled downtime)
- Service manager job change (new SM = restart relationship from scratch)

**Churn save playbook:**
1. Schedule call with service manager AND dealer principal (both must be engaged)
2. Present FTFR data: show ROI in dollars (FTFR improvement × average repair order value × repair volume)
3. Offer: complimentary half-day on-site refresher training if adoption is low
4. Escalation: if dealer principal is still dissatisfied, offer a 30-day extension and a dedicated feature request slot with the product team

**Acceptable reasons to let a customer churn:** Dealer closes or is acquired; dealer exits the OEM network; dealer moves to OEM embedded channel (not a loss — we still receive wholesale revenue).

---

## Support SLA by Tier

| Tier | First Response | Resolution Target |
|---|---|---|
| Enterprise | < 1 hour (business hours) | < 4 hours |
| Professional | < 4 hours (business hours) | < 24 hours |
| Core | < 8 hours (business hours) | < 48 hours |
| Safety-critical (HV-STOP malfunction, any tier) | < 15 minutes (24/7) | Immediate escalation |
