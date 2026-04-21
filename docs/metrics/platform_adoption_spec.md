# Platform Adoption Spec — ≥70% DAU / Licensed Bays Target
## Step 47 — Phase 4, Metrics 3

**Roadmap reference:** Phase 4, Metrics 3
**Status:** `[ ]` Not started
**Owner:** Product + Customer Success
**Target:** ≥ 70% DAU / licensed bays ratio by Month 20 (Phase 4 exit gate)

---

## Objective

Demonstrate that X-Ray Vision Diagnostics is genuinely used — not shelf-ware. A ≥ 70% DAU / licensed bays ratio means that on any given working day, 7 out of every 10 licensed bays are actively using the headset system. This metric validates product-market fit at commercial scale and is a leading indicator of renewal and expansion revenue.

**Why DAU / licensed bays (not DAU alone):** Raw DAU grows with new site deployments and is not a quality signal. Normalizing by licensed bays controls for scale and measures engagement depth within the installed base.

---

## Metric Definition

### DAU (Daily Active Users)

**DAU** = count of unique headset activations with ≥ 1 completed diagnostic session on a given calendar day

**Qualified session criteria:**
- Headset powered on and connected to edge server
- At least one VoiceNLP intent processed (tech spoke at least one recognized command)
- Session duration ≥ 5 minutes
- At least one CAD layer activated by DiagnosticCore

**Why these criteria:** A headset turned on but not used (e.g., left charging with diagnostic app open) should not count as a DAU. The 5-minute + voice command + layer activation threshold ensures the system was genuinely used for a diagnostic task.

### Licensed Bays

**Licensed bays** = count of bays with an active paid subscription and functioning edge server + headset stack as of the measurement date

- Bays under installation (not yet live) are excluded
- Bays with a known hardware failure (offline > 48 hours) are excluded from denominator for the outage period
- Bays on paid subscription regardless of actual use are included in denominator (this is the engagement pressure metric)

### DAU / Licensed Bays Ratio

**Adoption rate (%) = (DAU on measurement day / active licensed bays on measurement day) × 100**

Reported as:
- **Daily snapshot:** single-day ratio
- **Rolling 30-day average:** smooths out weekends, holidays, and one-off low-usage days
- **Weekly trend:** 7-day moving average, displayed on manager dashboard

---

## Expected Usage Patterns

### Structural Constraints on DAU

Not every licensed bay will show headset usage every day. Structural reasons:

| Constraint | Expected Impact on DAU |
|---|---|
| Bay not scheduled for EV work that day | ~20% of bays on any given day may have no enrolled fault type presenting |
| Technician absent (PTO, sick) | ~8% daily absence rate typical for automotive service |
| Vehicle in-bay but fault not yet triaged | Technician may not activate headset until DTC pulled |
| Weekend service schedule (some sites) | Saturday-only service reduces 7-day rolling average |

**Realistic ceiling:** Given these structural constraints, 80–85% DAU / licensed bays is near-maximum in a normal operating week. The 70% target accounts for structural floor without allowing true non-usage to hide.

### Adoption Baseline from Phase 3

Phase 3 beta (3 bays) provided early adoption signal. If Phase 3 adoption rate was < 50%, Phase 4 target of 70% requires an active intervention plan (see below). If Phase 3 adoption rate exceeded 70%, Phase 4 target is achievable without new interventions.

---

## Data Collection Architecture

| Data Source | Metric Derived | Agent / System |
|---|---|---|
| Headset session logs (HoloLens 2 SDK) | Session start, duration, commands processed | Edge server session manager |
| VoiceNLP intent log | Intent count per session (used to validate "genuine use") | VoiceNLP agent log |
| CADRenderer activation log | Layer activation events per session | CADRenderer agent log |
| Subscription management system | Licensed bay count, activation date, hardware status | Cloud backend / billing system |

**Pipeline:** Edge server sends anonymized session metadata to cloud analytics every 15 minutes. Cloud batch job computes DAU at midnight UTC. Dashboard updated by 6:00 AM local time each morning.

---

## Adoption Drivers — What Moves the Metric

### Positive Drivers
- **Peer champion program** — trained peer champions at each site model consistent headset use; effect strongest in Months 1–3 post-deployment (`docs/deployment/onboarding_program_spec.md`)
- **Manager dashboard visibility** — service managers can see per-technician adoption; creates social accountability
- **Job card time savings** — as technicians experience MTTD reduction personally, headset becomes the preferred tool
- **FTFR improvement** — fewer comeback repairs makes value case tangible to individual technicians
- **Spanish language support** — removes language barrier for ~19% of workforce; directly lifts DAU in Spanish-dominant service teams

### Adoption Risk Factors

| Risk | Indicator | Mitigation |
|---|---|---|
| Latency SLA violations | Graceful degradation events > 3 per session average | Engineering — network + edge server performance review |
| Voice recognition failures | VCRR dropping below 97% | ML — model retrain trigger |
| Headset comfort issues | Session duration declining week-over-week | UX — headset fit training; explore Magic Leap 2 as alternative |
| Manager not engaged | Manager dashboard login < 1/week | Customer Success — manager re-onboarding call |
| Peer champion turnover | Champion left company or moved to different role | Customer Success — nominate and train replacement within 30 days |

---

## Monitoring & Alerting

### Manager Dashboard (Site Level)
- Adoption rate (DAU / licensed bays) — rolling 30-day, displayed as primary engagement KPI
- Per-technician session count for current week — not shown as a performance metric but available for manager coaching
- Site adoption vs. network average — "Your site: 72% / Network average: 68%"

### Customer Success (CS) Alerting

| Trigger | Alert | Response |
|---|---|---|
| Site adoption drops below 50% for 7 consecutive days | CS team notified immediately | CS manager calls service manager within 24 hours |
| Site adoption below 60% for 30 consecutive days | Escalation to VP Customer Success | Onsite visit + root cause review |
| Individual technician 0 sessions in 14 days | CS team notified | Manager coaching prompt; CS may offer 1:1 refresher |
| New site drops below 60% in first 30 days post-install | Onboarding failure signal | CS offers additional peer champion sessions |

### Network-Level Reporting (Product Team)
- Weekly: network-wide DAU / licensed bays ratio reported to product team
- Monthly: adoption cohort analysis — bays by months-since-install; identifies whether adoption improves or declines over time
- Quarterly: churn risk analysis — sites below 60% adoption are flagged as renewal risk

---

## Interventions for Below-Target Sites

If a site's 30-day rolling adoption falls below 70% for 30 consecutive days:

1. **Diagnosis call** — CS manager + product team member joins 30-min call with service manager
   - Questions: Is adoption driven by EV volume decline? Headset issues? Technician resistance? Manager not reinforcing usage?
2. **Root cause playbook:**
   - Low EV volume at that site → add ICE fault types to enrolled scope
   - Hardware issues → engineering support dispatched within 5 business days
   - Technician resistance → onboarding refresher + peer champion re-activation
   - No manager reinforcement → manager re-onboarding session
3. **Recovery timeline:** 60 days to return to ≥ 70%
4. **If not recovered:** escalate to VP Sales for contract renewal risk assessment

---

## Phase 4 Exit Gate Criteria

- Network-wide rolling 30-day adoption rate ≥ 70% DAU / licensed bays, sustained for ≥ 60 consecutive days
- No individual site below 55% for the final 30 days of measurement period (floor requirement prevents average being held up by a few high-adoption sites while others languish)
- Adoption data presented to board at Month 20 business review

---

## Links to Related Specs

- Onboarding program (peer champion model): `docs/deployment/onboarding_program_spec.md`
- Manager dashboard (adoption display): `docs/features/manager_dashboard_spec.md`
- Voice Command Recognition Rate (VCRR): `docs/metrics/voice_recognition_rate_spec.md`
- Graceful degradation (latency-driven session interruption): `docs/features/graceful_degradation_spec.md`
- Spanish language support (adoption barrier removal): `docs/features/voice_nlp_spanish_spec.md`
