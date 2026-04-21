# RLHF Pipeline Spec — DiagnosticCore Confidence Improvement
## Step 40 — Phase 4, Feature 1

**Roadmap reference:** Phase 4, Feature 1
**Status:** `[ ]` Not started
**Owner:** ML Engineering + PM
**Target:** First RLHF-improved model deployed by Month 17

---

## Objective

Build a Reinforcement Learning from Human Feedback (RLHF) pipeline that uses real technician repair outcomes to improve DiagnosticCore's fault confidence scoring. Over time, DiagnosticCore learns which RAG retrievals and diagnoses consistently lead to correct first-time repairs — and which do not.

**Why this matters:** Phase 2/3 DiagnosticCore is static — its confidence scoring is based on RAG retrieval similarity, not on whether the diagnosis was actually correct. The RLHF loop closes this gap by grounding confidence in real-world outcomes.

**Hard constraint:** RLHF must never change which repair steps are shown. All repair steps remain OEM-sourced. RLHF only adjusts the confidence score and the ranking/weighting of RAG retrievals — not the content of repair guidance.

---

## Feedback Signal Sources

### Signal 1: FTFR Outcome (Primary — Strongest Signal)
- Source: DMS job card data (via CDK/R&R integration)
- Signal: Vehicle returned for same fault within 30 days = negative reward; no return = positive reward
- Delay: 30-day lag (cannot train on a repair until 30 days have passed)
- Quality: Ground truth — this is the North Star metric

### Signal 2: Technician Confidence Survey (Secondary)
- Source: Post-repair in-headset survey (`docs/features/confidence_survey_spec.md`)
- Signal: Q1 (diagnosis confidence) + Q2 (repair confidence) composite
- Delay: Immediate (available at session close)
- Quality: Subjective proxy; correlates with FTFR but is not ground truth

### Signal 3: DiagnosticCore Confidence Score vs. Outcome (Calibration)
- Source: Session logs + FTFR outcome
- Signal: If DiagnosticCore reported confidence 0.90 but vehicle came back → overconfidence; adjust calibration
- Use: Platt scaling to calibrate confidence scores over time

---

## Data Pipeline

### Collection

```
Session Close Event
  → session_log written to edge server
  → synced to cloud (PostgreSQL)

30 days later:
  → DMS pull: was VIN seen again for same fault?
  → FTFR outcome attached to session_log
  → training record created

Training Record Schema:
{
  "session_id": "uuid",
  "vehicle_vin": "...",
  "fault_type": "coolant_leak",
  "oem": "toyota",
  "dtc_codes": ["P0A93", "P1DF1"],
  "dc_confidence_score": 0.87,
  "rag_query": "coolant pump performance fault bZ4X",
  "top3_retrieved_chunks": ["chunk_id_1", "chunk_id_2", "chunk_id_3"],
  "tech_confidence_q1": 4,
  "tech_confidence_q2": 5,
  "ftfr_outcome": "fixed",  // "fixed" | "returned"
  "reward": 1.0  // 1.0 = fixed, 0.0 = returned, interpolated by survey
}
```

### Reward Function

```python
def compute_reward(ftfr_outcome, tech_confidence_composite):
    if ftfr_outcome == "fixed":
        base_reward = 1.0
    else:  # returned
        base_reward = 0.0
    
    # Blend in survey signal (weighted 20% — survey is leading, FTFR is lagging)
    survey_signal = (tech_confidence_composite - 1) / 4  # normalize 1-5 to 0-1
    reward = 0.8 * base_reward + 0.2 * survey_signal
    return reward
```

---

## What Gets Trained

### What changes: RAG retrieval re-ranking
DiagnosticCore uses a hybrid retrieval (dense + BM25). The RLHF loop trains a lightweight re-ranking model (cross-encoder) that scores retrieved chunks by their historical association with positive outcomes.

```
Re-ranker input: (query, chunk)
Re-ranker output: relevance score (0–1)
Training signal: reward from downstream FTFR outcome
```

Architecture: DistilBERT cross-encoder fine-tuned on (query, chunk, reward) triples. Runs on edge server (Jetson AGX Orin); inference latency < 20ms.

### What changes: Confidence score calibration
Platt scaling applied to raw DiagnosticCore confidence scores using historical (predicted confidence, actual FTFR outcome) pairs. Recalibrated monthly.

### What does NOT change
- Content of repair steps (OEM-sourced, never modified by RLHF)
- Intent classification in VoiceNLP
- CAD layer definitions
- SafetyGuard logic

---

## Training Cadence

| Phase | Frequency | Trigger | Data Required |
|---|---|---|---|
| Initial bootstrap | Once (Month 17) | Manual | 200+ labeled training records |
| Monthly update | Monthly | Automated pipeline | ≥ 50 new labeled records |
| Emergency retrain | As needed | PM trigger | If accuracy drops > 5pp |

### Bootstrap Data
Phase 3 beta will generate the first labeled training records (50+ repairs with FTFR outcomes). 200 records required for initial training. Phase 3 ends at Month 14; 30-day FTFR lag means bootstrap data available by Month 15. First model ready Month 17.

---

## Safety Controls

**No autonomous deployment:** Every RLHF-trained model update requires:
1. Offline evaluation on held-out test set (10% of labeled data)
2. If held-out FTFR prediction accuracy < 75%: block deployment; investigate
3. Manual sign-off from ML Lead + PM before production push
4. A/B test for first 2 weeks: 50% of sessions get updated model, 50% get previous model
5. A/B winner determined by FTFR outcome at 30-day lag; loser rolled back

**Bias audit:** Before each deployment, check: does the new model perform equally across vehicle OEM, fault type, and technician experience level? Flag any subgroup with > 10pp accuracy gap.

---

## Exit Criteria

- [ ] Training data pipeline live: session records automatically joined with FTFR outcomes at 30-day lag
- [ ] 200+ labeled training records collected from Phase 3 beta
- [ ] Re-ranking model trained and evaluated (held-out accuracy ≥ 75%)
- [ ] First RLHF-improved model deployed to 10% of Phase 4 fleet (canary)
- [ ] A/B test result shows ≥ 2pp confidence score improvement or ≥ 2pp FTFR improvement vs. baseline
- [ ] Bias audit passed (no subgroup > 10pp gap)
