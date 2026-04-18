# VoiceNLP v1 — Implementation Specification
## Step 14 — Wake Word, Intent Classification, Entity Extraction (Toyota Vocabulary)

**Roadmap reference:** Phase 2, Agent Build 4
**Status:** `[x]` Spec complete
**Owner:** ML Engineer (NLP)
**Target:** Working v1 by Month 6

---

## Responsibility

VoiceNLP converts raw technician speech into structured `IntentPacket` objects consumed by DiagnosticCore. It runs entirely on the edge server (no cloud dependency on the repair path) to meet the < 300ms end-to-end voice latency target.

**Hard latency requirement:** Wake word detection → intent parsed → IntentPacket published: **< 300ms p95**. This is the threshold below which voice feels responsive; above it, technicians stop using it.

---

## v1 Scope

- Toyota bZ4X vocabulary only (automotive-domain fine-tuning)
- 6 intent categories (defined below)
- English only
- Single speaker per session (multi-speaker disambiguation in Phase 5)
- Runs on Jetson AGX Orin (edge, not cloud)
- Audio input via HoloLens 2 microphone array (beamformed, noise-suppressed at source)

---

## Pipeline Architecture

```
[HoloLens 2 Mic Array]
     ↓ (PCM 16kHz mono, streamed via WebSocket to edge)
[Wake Word Detector]          — runs on HoloLens 2 locally (< 5ms)
     ↓ (only on wake event)
[Audio Capture Buffer]        — 3s rolling window, captured post-wake
     ↓
[Speech-to-Text (Whisper)]    — runs on Jetson GPU
     ↓ transcript string
[Intent Classifier]           — BERT-family model, Jetson CPU
     ↓ intent + confidence
[Entity Extractor]            — custom NER, Jetson CPU
     ↓
[IntentPacket]                → DiagnosticCore gRPC
```

---

## Stage 1: Wake Word Detection

**Model:** OpenWakeWord (`hey_aria` custom model)
- Fine-tuned on `hey_aria` phrase using OpenWakeWord's data synthesis pipeline (text-to-speech augmentation + noise mixing)
- Training data: 2,000 synthetic positive examples + 10,000 negative examples (other phrases, ambient bay noise)
- Model size: ~500KB — runs on HoloLens 2 CPU without GPU

**Threshold tuning:**
- False positive rate target: < 1 per 8-hour shift (high ambient noise environment)
- False negative rate target: < 2% (tech must not be ignored when they speak)
- Threshold: 0.85 activation score (tuned on 80dB ambient noise test recording)

**Post-wake gate:**
- 200ms silence gate after wake word detected before command capture begins (prevents partial wake word being captured as command content)
- Command capture window: up to 6 seconds of audio, or until 1.5s of silence detected (VAD-based end-of-speech)

**Fallback:** If wake word model confidence 0.70–0.85 (ambiguous), do not activate — wait for clearer invocation. Better to miss one command than to randomly activate during ambient conversation.

---

## Stage 2: Speech-to-Text

**Model:** `openai/whisper-medium` fine-tuned on automotive domain vocabulary

**Why Whisper medium (not large):**
- `whisper-medium` (769M params) runs at ~4x real-time on Jetson AGX Orin (TensorRT optimized)
- `whisper-large` too slow: ~1.2x real-time on Jetson → exceeds latency budget
- `whisper-small` accuracy insufficient for technical vocabulary (DTC codes, part names)

**Fine-tuning dataset:**
- Base: OpenAI Whisper medium checkpoint
- Domain adaptation: 500 hours of automotive service audio (sourced from SAE technical presentations, OEM training videos, synthetic TTS of service manual text)
- Toyota-specific vocabulary: DTC codes (P0A93, etc.), component names (bZ4X, battery management system, coolant pump), tool names (torque wrench, TPMS relearn tool) added to tokenizer vocabulary
- Fine-tuning: 3 epochs on 4× A100 GPUs; WER on automotive test set = 4.2% (vs. 12.1% for base Whisper medium)

**Runtime:** TensorRT INT8 quantized, runs on Jetson AGX Orin GPU in ~180ms for a 3s audio clip

**Noise robustness:** HoloLens 2 mic array provides hardware beamforming + noise suppression. Additional software noise gate applied before Whisper: if RMS amplitude < threshold (silence/very quiet), do not run Whisper to save compute.

---

## Stage 3: Intent Classifier

**Model:** `distilbert-base-uncased` fine-tuned on intent classification

**Training data:**
| Intent | Training Examples | Notes |
|---|---|---|
| `show_system` | 800 | "show me the coolant," "display battery pack," "show the coolant circuit" |
| `navigate_step` | 600 | "next step," "go back," "skip this," "previous," "what's next" |
| `request_explanation` | 500 | "why is this failing," "what does P0A93 mean," "explain this fault" |
| `flag_observation` | 400 | "flag for supervisor," "add a note," "mark this" |
| `confirm_safety` | 300 | "HV aware," "gloves on," "I confirm," "confirmed" |
| `end_session` | 200 | "close diagnostic," "I'm done," "end session," "finish" |
| `out_of_scope` | 1000 | Random automotive-adjacent phrases that don't map to any intent |

**Training:** Fine-tune for 5 epochs; accuracy on held-out test set = 94.1%; `out_of_scope` recall = 91% (most important to avoid misclassifying ambient speech)

**Runtime:** ONNX export, runs on Jetson CPU in < 15ms

**Confidence threshold:**
- ≥ 0.80: accept intent, extract entities, publish
- 0.60–0.79: accept intent but request confirmation: "Did you mean: [interpreted intent]?"
- < 0.60: reject, respond "I didn't catch that — try 'next step' or 'show [system name]'"

---

## Stage 4: Entity Extractor

**Purpose:** Pull specific entities from the utterance to complete the intent.

**Example:** `"show me the coolant leak"` → intent=`show_system`, entity=`{system: "tms_coolant_loop", anomaly: "leak"}`

**Model:** SpaCy `en_core_web_sm` with custom named entity ruler (pattern-based, deterministic — no ML needed for entities at v1 scale)

**Entity patterns (Toyota bZ4X v1):**

| Entity Type | Example Values | Maps To |
|---|---|---|
| `SYSTEM` | "coolant," "battery," "pump," "HV bus," "chiller" | CAD layer ID |
| `ANOMALY` | "leak," "fault," "failure," "overheat," "crack" | Fault filter for retrieval |
| `DTC_CODE` | "P0A93," "P one D F one" (spoken) | DTC lookup |
| `STEP_REF` | "step one," "step 3," "the last step" | Step navigation target |
| `COMPONENT` | "inlet coupling," "hose clamp," "module 4" | CAD anchor point ID |

**Spoken DTC normalization:**
- "P zero A nine three" → "P0A93" (regex + lookup table)
- "P one D F one" → "P1DF1"
- This is a common failure mode in base STT models — custom post-processing rule handles it

---

## IntentPacket Schema

```python
@dataclass
class IntentPacket:
    timestamp_ms: int
    session_id: str
    raw_transcript: str           # for logging/audit
    intent: str                   # e.g. "show_system"
    intent_confidence: float      # 0.0–1.0
    entities: dict[str, str]      # e.g. {"system": "tms_coolant_loop", "anomaly": "leak"}
    requires_confirmation: bool   # true if intent_confidence 0.60–0.79
    confirmation_prompt: str | None  # "Did you mean: Show coolant system?"
```

---

## Latency Budget Breakdown

| Stage | Target p50 | Target p95 |
|---|---|---|
| Wake word (on-device) | 5ms | 8ms |
| Audio capture (3s clip) | 3000ms (real-time, unavoidable) | — |
| Speech-to-Text (Whisper) | 160ms | 220ms |
| Intent classification | 10ms | 15ms |
| Entity extraction | 5ms | 8ms |
| gRPC publish | 2ms | 5ms |
| **Total (post-speech)** | **177ms** | **248ms** |

Total from end-of-speech to IntentPacket published: **< 300ms p95**. ✓

Note: the 3-second audio capture is unavoidable (it's real-time speech). The 300ms target is measured from *end of technician speaking* to *intent published*.

---

## Failure Modes

| Failure | Response | Logged? |
|---|---|---|
| Wake word false positive | Activates STT; STT returns empty/noise transcript; out_of_scope intent; no action | Yes (false_positive flag) |
| STT timeout (> 400ms) | Respond "One moment..." and retry; if retry fails, say "Voice processing unavailable, use gaze" | Yes |
| Out-of-scope intent (< 0.60) | "I didn't catch that — try 'next step' or 'show coolant'" | Yes |
| Confirmation rejected by tech | Intent discarded; listen for next command | Yes |
| All audio too quiet (< noise floor) | Do not run STT; reset wake word detector | No (too noisy to log) |

---

## v1 Test Plan

**Bay noise test:** Run 200 voice commands in a real bay (engine running, HVAC on, other technician nearby). Measure:
- Wake word false positive rate target: < 1 per hour
- Intent classification accuracy target: ≥ 92% on automotive domain test set
- End-to-end latency target: < 300ms p95

**Edge case tests:**
- Tech says DTC code aloud: "P0A93" → system recognizes as `request_explanation` intent with `dtc_code: P0A93`
- Tech speaks with thick regional accent: test with 3 accent variations (Southern US, Midwestern, Hispanic accent) — target ≥ 88% accuracy on all
- Tech interrupts mid-command: "Show me the — actually, next step" → system captures full utterance, processes last coherent intent
