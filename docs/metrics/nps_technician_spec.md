# Net Promoter Score ≥50 — Technician Satisfaction

**Phase:** 5
**Roadmap Item:** Metrics: Net Promoter Score ≥ 50 from technicians (user satisfaction)

---

## Objective

Measure and maintain a Net Promoter Score (NPS) of ≥50 from technicians — the primary users of the platform. NPS ≥50 is the threshold that separates good products from great ones in B2B SaaS (industry median is ~31; scores ≥50 are considered excellent). It is also the threshold OEM embedded deal negotiators ask about: "Do technicians actually like using this?"

NPS ≥50 from technicians is a proxy for three things:
1. The product is genuinely useful and not burdensome
2. Technician adoption will sustain without management enforcement
3. Word-of-mouth referrals from techs to other dealerships are likely

---

## NPS Definition and Calculation

**Question:** "On a scale of 0–10, how likely are you to recommend X-Ray Vision Diagnostics to a fellow technician at another dealership?"

- **Promoters:** Score 9–10
- **Passives:** Score 7–8
- **Detractors:** Score 0–6

**NPS = % Promoters − % Detractors**

Range: −100 to +100. Target: ≥ 50.

This is the standard NPS question and calculation — no modification to the formula.

---

## Survey Design

### Survey Delivery
- **Channel:** In-headset survey, triggered automatically at the end of a repair session
- **Frequency:** Once per technician per 90-day period — not after every repair (survey fatigue)
- **Timing:** Session ends → HV-STOP deactivated → all steps marked complete → 10-second pause → survey card appears
- **Opt-out:** Tech can dismiss by saying "Skip survey" — counted as non-response (not as detractor)
- **Duration:** ≤ 60 seconds total (NPS question + 2 follow-up questions)

### Survey Flow

**Screen 1 — NPS Question**
"How likely are you to recommend X-Ray Vision Diagnostics to a fellow tech at another dealership?"
→ 0–10 slider, voice-selectable ("Six") or gaze-selectable

**Screen 2 — One Follow-Up (Branched)**
- If Promoter (9–10): "What's the main reason for your high score?" → 4 options: "Saves time", "Easier diagnosis", "Better accuracy", "Other" + optional freeform (voice-to-text, max 30 seconds)
- If Detractor (0–6): "What's the main thing we should improve?" → 4 options: "Voice commands miss me", "Overlay is hard to read", "Too slow", "Other" + optional freeform

**Screen 3 — Thank you + dismiss**
No third question — keep it ≤60 seconds.

### Response Schema
```json
{
  "response_id": "uuid",
  "tech_id": "string",
  "session_id": "uuid",
  "dealer_id": "string",
  "market": "US | CA | UK",
  "fault_type": "string",
  "vehicle_model": "string",
  "score": "int (0-10)",
  "category": "promoter | passive | detractor",
  "follow_up_choice": "string",
  "follow_up_freeform": "string | null",
  "submitted_at": "ISO8601",
  "session_duration_minutes": "int"
}
```

---

## Segmentation

At scale (500 bays, multiple markets), aggregate NPS hides actionable signal. Required segments:

| Dimension | Why Segment |
|---|---|
| Market (US / Canada / UK) | International launch may score lower initially; track separately |
| Technician tenure with product (0–30 days, 31–90, 90+) | New users often score lower; distinguish onboarding issues from product issues |
| Fault type | If one fault type generates disproportionate detractors, it indicates a UX or accuracy problem specific to that flow |
| Dealer tier (Core / Professional / Enterprise) | Feature access differs; Core tier may score lower if missing features they want |
| Apprentice Mode users vs. standard | Training mode techs have different UX expectations |

---

## Alerting and Response

### Thresholds

| Alert | Trigger | Response |
|---|---|---|
| Warning | Segment NPS drops below 40 (rolling 8-week window) | CS reviews detractor follow-ups; product PM review within 1 week |
| Critical | Segment NPS drops below 30 | Executive review; targeted outreach to detractor cohort |
| Platform-wide | Overall NPS drops below 45 | Product team sprint prioritization review; all detractor feedback reviewed |

### Detractor Outreach Program
For every detractor (score 0–6) who provides freeform feedback:
- CS team reaches out within 5 business days: "We saw your feedback — here's what we're doing about it"
- Goal: convert detractors to passives through follow-through, not just words
- Track: detractor re-survey rate 90 days later; target ≥ 30% improvement in score

This is standard NPS practice in B2B SaaS (Gainsight, Medallia playbooks). The follow-up matters as much as the score.

---

## Improvement Levers

### Top Detractor Categories and Responses (Based on Phase 3 Confidence Survey Patterns)

| Detractor Reason | Expected Frequency | Response |
|---|---|---|
| "Voice commands miss me" | High (especially new users, noisy bays) | VoiceNLP vocabulary expansion; noise-cancellation tuning; Apprentice Mode teaches correct command phrasing |
| "Overlay is hard to read" | Medium (lighting-dependent) | Graceful degradation UX (Phase 3) addresses this; also: adaptive contrast modes for direct sunlight |
| "Too slow" | Medium | Predictive rendering improvements; if latency > 40ms p95, escalate to engineering |
| "Too many steps / too much information" | Low (mainly new users) | Onboarding UX improvements; expert mode: condenses step cards for techs with ≥ 50 sessions |

**Expert Mode (derived from NPS data):** After 50 completed sessions on a fault type, tech is offered "Expert Mode" for that fault type — step cards condensed to single-line instructions, no WHY/WATCH FOR text, faster pacing. This directly addresses the "too much information" detractor segment without removing the scaffolding that new users need.

---

## NPS in OEM and Enterprise Sales

### How OEM Teams Use NPS Data
- In OEM embedded deal presentations: show aggregate NPS score and trend chart — "Our technicians give us a 54 NPS after 90 days of use"
- Include anonymized promoter quotes from follow-up freeform field
- OEM training teams want to see NPS by fault type — proof that the training overlay is effective for their specific vehicles

### How Enterprise Dealers Use NPS Data
- Service managers want to know their technicians' NPS vs. platform average — a below-average dealer NPS is a CS intervention signal
- Add NPS score (aggregate for the dealer) to the manager dashboard under "Technician Satisfaction"

---

## Rollout Plan

| Month | Milestone |
|---|---|
| 22 | Design survey UX: in-headset flow, voice-selectable slider, follow-up branches |
| 23 | Build survey delivery engine: post-session trigger, 90-day cooldown, opt-out handling |
| 23 | Build response schema, storage (regional, no cross-border PII transfer), Athena query |
| 24 | Build NPS dashboard (manager view + internal Product/CS view) |
| 24 | Build detractor outreach workflow (CS Salesforce integration: detractor response → CS task) |
| 25 | First NPS data collection begins (rolled out to all Phase 4 bays still active) |
| 27 | First 90-day cohort complete — baseline NPS established |
| 28 | Expert Mode built based on "too much information" detractor signal |
| 30 | Phase 5 exit criterion assessment: NPS ≥ 50 across ≥ 100 respondents |

---

## Phase 5 Exit Criterion

NPS ≥ 50 from technicians is confirmed when:
- ≥ 100 unique technician respondents (not responses — unique techs, to avoid one power user driving the score)
- Rolling 12-week NPS ≥ 50
- No market segment persistently below 40 for more than 8 consecutive weeks
- Response rate ≥ 40% (eligible techs who were shown the survey and completed it — dismissals excluded from denominator)

Response rate ≥ 40% is important: a low response rate biases NPS toward either engaged promoters or vocal detractors, making the score unreliable.

---

## Safety Incident Rate Metric

**Roadmap Item:** Metrics: Safety incident rate ≤ 0.8 per 1,000 repairs (vs. OSHA baseline 2.3)

### Definition
Safety incident: any injury, near-miss, or equipment damage that occurs during a repair session in a bay where X-Ray Vision Diagnostics was active and required documentation under OSHA recordkeeping requirements (OSHA 300 log).

Rate = (incidents / completed repairs) × 1,000

### Baseline
OSHA industry baseline for automotive service technicians: 2.3 recordable incidents per 1,000 workers per year. We normalize to per-1,000 repairs (not workers) for precision — a repair is a bounded event.

### Measurement
- Source: HV-STOP audit logs (captures near-misses) + dealer OSHA 300 log entries linked to active sessions
- Dealer self-report + session cross-reference: if a dealer files an OSHA incident report on a date/bay where a session was active, the session ID is linked
- Monthly aggregation: incidents logged → rate calculated → reported to compliance team

### Alerting
- Any single HV-system injury: immediate executive alert + incident review
- Rate > 1.5/1,000 (rolling 8-week): compliance team review + engineering incident analysis
- Rate > 2.0/1,000: platform-wide safety review; consider mandatory Apprentice Mode retraining for HV fault types

### Target Achievement
≤ 0.8/1,000 is expected to be achievable because:
- HV-STOP with dual-sensor confirmation prevents the highest-severity incidents
- Step-by-step overlay reduces procedural errors that lead to incidents
- This target represents a 65% reduction from the OSHA baseline — ambitious but consistent with Phase 3 and 4 results where zero safety incidents were recorded

Phase 5 exit criterion: rate ≤ 0.8/1,000 sustained over ≥ 1,000 completed repairs with no HV-system injuries.
