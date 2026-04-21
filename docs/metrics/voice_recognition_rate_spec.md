# Voice Command Recognition Rate Spec
## Step 33 — Phase 3, Metrics 3

**Roadmap reference:** Phase 3, Metrics 3
**Status:** `[ ]` Not started
**Owner:** ML Engineering + PM
**Target:** Tracked from Day 1 of beta; ≥97% by Month 11

---

## Objective

Track and improve Voice Command Recognition Rate (VCRR) — the % of voice commands where VoiceNLP correctly classified intent and the system responded appropriately. Target ≥97% in production.

**Why 97%:** At < 95%, technicians abandon voice and lose the hands-free benefit that justifies the headset. At 97%+, the occasional failure is tolerable and the system remains the primary interaction modality.

---

## VCRR Definition

```
VCRR = Commands correctly resolved
       ─────────────────────────────
       Total commands attempted
```

**"Correctly resolved"** = VoiceNLP classified the correct intent AND the system responded with the expected action (CAD layer activated, step navigated, etc.) within 2 seconds.

**"Attempted"** = any utterance after the wake word that VoiceNLP received and processed. Excludes background noise events that did not trigger the wake word.

### Failure Categories

| Category | Code | Description |
|---|---|---|
| Wrong intent | WI | VoiceNLP classified wrong intent (e.g., `navigate_step` when tech said `show_system`) |
| No intent | NI | VoiceNLP returned `intent: null` — could not classify |
| Correct intent, wrong entity | WE | Intent correct, entity extracted wrong (e.g., "coolant pump" extracted as "coolant sensor") |
| Correct classification, timeout | TO | Intent classified correctly but system response > 2,000ms |
| Wake word false negative | FN | Tech said wake word; system did not activate |
| Wake word false positive | FP | Background speech triggered wake word |

---

## Logging Schema

Every VoiceNLP event writes a log entry:

```json
{
  "event_id": "uuid",
  "session_id": "session_uuid",
  "technician_id": "anon_hash",
  "timestamp_utc": "...",
  "raw_transcript": "show me the coolant system",
  "classified_intent": "show_system",
  "classified_entity": "tms_coolant_loop",
  "confidence_score": 0.91,
  "response_action": "CADRenderer.activate_layer",
  "response_latency_ms": 380,
  "resolved": true,
  "failure_category": null,
  "tech_correction": false
}
```

**Tech correction flag:** Set to `true` if the tech repeated the same command within 5 seconds of a failed resolution (proxy for "I had to say it again").

---

## Tracking by Context

### By Environment
VCRR is expected to vary by shop conditions. Track separately:
- Quiet bay (background noise < 60 dB)
- Active bay (pneumatic tools, radio, other techs talking — 70–85 dB)
- Vehicle running (engine noise — 75–90 dB)

**Acceptance threshold:** ≥97% VCRR in quiet bay; ≥94% in active bay (active bay target for Phase 4).

### By Fault Type
Some fault vocabularies are harder (e.g., "inverter" is one syllable away from "inhibitor"). Track per fault type to identify vocabulary gaps.

### By Technician
Track per tech to identify: (a) techs who need additional coaching, (b) speech patterns that consistently fail (accent, speech speed, mumbling). This data is internal only — never surfaced to the manager dashboard at individual level.

---

## Improvement Protocol

### Weekly Review (during beta)
- ML Engineer reviews VCRR dashboard every Monday
- Any intent class with > 5% failure rate in the past 7 days is flagged
- Root cause: review raw transcripts for systematic misclassification patterns

### Short-Term Fixes
| Issue | Fix | Timeline |
|---|---|---|
| High NI rate for a phrase | Add phrase variant to intent training data; retrain | 2 weeks |
| High WE rate for entity | Add entity synonym to extraction grammar | 1 week |
| Wake word false positive > 2% | Adjust wake word confidence threshold | 3 days |
| Shop noise degrading accuracy | Enable noise-adaptive gain control in mic pre-processing | 2 weeks |

### Training Data Collection
With tech consent, raw audio snippets for failed commands can be collected, anonymized, and used to expand VoiceNLP training data. Protocol:
- Opt-in per technician (confirmed at onboarding)
- Audio retained max 7 days; transcript retained for model training
- No PII in transcripts; technician ID is anonymized hash

---

## Dashboard Display

Manager Dashboard shows VCRR as:
- 7-day rolling rate (prominent, top of dashboard)
- Trend line (30 days)
- Breakdown by failure category (pie chart — internal PM view only)
- Alert if VCRR drops below 95% for 48 consecutive hours (email to PM + ML Lead)

---

## Exit Criteria

- [ ] VCRR tracking live from Day 1 of beta (every session logged)
- [ ] VCRR ≥ 97% in quiet bay conditions by Month 11
- [ ] VCRR ≥ 94% in active bay conditions by Month 11
- [ ] Weekly review cadence established; ML Lead reviewing each Monday
- [ ] At least one model improvement deployed in response to beta data before Phase 3 exit
