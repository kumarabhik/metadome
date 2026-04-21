# VoiceNLP Spanish Language Expansion Spec
## Step 41 — Phase 4, Feature 6

**Roadmap reference:** Phase 4, Feature 6
**Status:** `[ ]` Not started
**Owner:** ML Engineering + Linguistics Contractor + Product
**Target:** Spanish intent recognition live in production by Month 18

---

## Objective

Extend VoiceNLP from English-only to bilingual English/Spanish operation. Spanish is the primary language of approximately 19% of US automotive service technicians — the largest non-English workforce segment. Excluding Spanish speakers from voice-primary interaction forces them onto touch/tablet fallback, degrading the hands-free safety model and reducing adoption in high-value markets (Texas, California, Florida, Arizona).

**Success criteria:** Spanish Voice Command Recognition Rate ≥ 97% (same bar as English), measured over 50+ Spanish-language repair sessions across enrolled fault types.

---

## Language Scope

### Dialect Targeting

| Dialect | Priority | Rationale |
|---|---|---|
| Mexican Spanish | Primary | Dominant dialect among US automotive technicians per BLS demographic data |
| Caribbean Spanish (Puerto Rican, Dominican) | Secondary | Significant presence in Northeast dealer networks |
| Central American Spanish | Tertiary | Growing presence in Southeast markets |

**Approach:** Train on Mexican Spanish corpus; evaluate on multi-dialect test set. Dialect-specific acoustic models if recognition rate falls below 97% on secondary dialects in evaluation.

### What Is and Is Not Localized

| Component | Localized? | Notes |
|---|---|---|
| Wake word | Yes — "Oye Aria" | Configurable per site; "Hey Aria" remains for English mode |
| All 6 intent categories | Yes | Full parity with English intent taxonomy |
| OEM component vocabulary | Yes | Toyota, Ford, Stellantis Spanish-language service manual terms |
| DTC codes | No — language-neutral | "P0A93" is the same in all languages |
| UI overlay text | Out of scope for this spec | Headset HUD text localization is a separate work stream |
| NarrativeInstruction (repair steps) | Out of scope | OEM service bulletin translation requires OEM approval |

---

## Intent Taxonomy — Spanish Mapping

| Intent | English Example | Spanish Equivalent | VoiceNLP Output (unchanged) |
|---|---|---|---|
| `show_system` | "Show me the coolant" | "Muéstrame el refrigerante" / "Enséñame el sistema de refrigeración" | `{action: show, system: coolant}` |
| `navigate_step` | "Next step" | "Siguiente paso" / "El siguiente" / "Sigue" | `{action: navigate, direction: next}` |
| `request_explanation` | "Why is this failing?" | "¿Por qué está fallando?" / "¿Qué significa este código?" | `{action: explain, target: current_fault}` |
| `flag_observation` | "Flag this for supervisor" | "Márcalo para el supervisor" / "Avisa al gerente" | `{action: flag, severity: normal}` |
| `confirm_safety` | "Gloves on, I confirm" | "Guantes puestos, confirmo" / "Sí, confirmo" | `{action: safety_ack, context: active_stop_gate}` |
| `end_session` | "Close diagnostic" | "Cerrar diagnóstico" / "Ya terminé" | `{action: session_end}` |

**Multi-word variant coverage:** Intent classifier must handle 5+ natural phrasings per intent. Technicians do not follow scripted commands — the model must generalize across informal speech, truncated commands, and regional vocabulary variation.

---

## OEM Vocabulary — Spanish Component Names

### Toyota bZ4X (Mexican Spanish service manual terminology)

| English Term | Spanish Term | CAD Entity |
|---|---|---|
| Battery pack | Paquete de baterías / Batería de alta tensión | `toyota_bz4x.battery_pack` |
| Coolant system | Sistema de refrigeración | `toyota_bz4x.coolant_system` |
| High-voltage bus | Bus de alta tensión / Cable naranja | `toyota_bz4x.hv_bus` |
| Thermal management module | Módulo de gestión térmica | `toyota_bz4x.tms_module` |
| Inverter | Inversor | `toyota_bz4x.inverter` |
| Service disconnect | Desconector de servicio | `toyota_bz4x.service_disconnect` |

### Ford Mustang Mach-E (Ford Mexico service manual)

| English Term | Spanish Term | CAD Entity |
|---|---|---|
| GigE battery | Batería de alta voltaje GigE | `ford_mache.hv_battery` |
| Power distribution module | Módulo de distribución de energía | `ford_mache.pdm` |
| Onboard charger | Cargador a bordo | `ford_mache.obc` |
| Drive unit | Unidad motriz | `ford_mache.drive_unit` |

**Source policy:** All Spanish component names sourced from OEM-published Spanish-language service manuals. No AI-generated translation of component names permitted — mistranslation of HV component names is a safety risk.

---

## Model Architecture

### Base ASR Model
- **Model:** Whisper-large-v3 fine-tuned on Spanish automotive domain corpus
- **Running on:** Edge server Jetson AGX Orin — same inference pipeline as English
- **Language detection:** Automatic language detection on first utterance after wake word (< 200ms). Subsequent utterances in same session use detected language mode. Tech can switch languages mid-session via "Switch to English" / "Cambiar a inglés."
- **Latency target:** < 150ms intent classification (same as English); add up to 30ms for language detection on first utterance = 180ms first-utterance budget

### Intent Classifier
- Lightweight BERT-family model (same architecture as English); trained on Spanish intent corpus
- Operates on edge — no cloud call
- Shared confidence threshold: < 0.75 triggers confirmation prompt in Spanish: "¿Quisiste decir: mostrar sistema de refrigeración?"

### Entity Extractor
- Custom NER for Spanish automotive vocabulary, trained on:
  - OEM Spanish-language service manuals (Toyota, Ford, Stellantis)
  - DTC code lookup tables (language-neutral)
  - Annotated technician session transcripts from beta (translated and labeled)

---

## Training Data Requirements

| Data Source | Volume Required | Status |
|---|---|---|
| Spanish automotive service manual corpus (Toyota, Ford) | ~500K tokens | Requires OEM data access approval |
| Technician speech recordings — Mexican Spanish | 20 hours minimum | Requires consent from beta sites (Month 16) |
| Synthetic augmentation (TTS-generated Spanish audio) | 100 hours | Generate from OEM text corpus using Azure Neural TTS (es-MX-JorgeNeural) |
| DTC code pronunciation corpus | 5,000 code utterances | Record in-house; codes are language-neutral |
| Intent annotation dataset | 3,000 labeled utterances per intent | Linguistics contractor + ML annotation pipeline |

**Data privacy:** All technician recordings anonymized. Consent form updated for Spanish-speaking participants. Recording retention policy: 90 days for training, then purged per privacy model (`docs/features/confidence_survey_spec.md` privacy section applies).

---

## Wake Word Behavior

| Mode | Wake Word | Configured By |
|---|---|---|
| English | "Hey Aria" | Default |
| Spanish | "Oye Aria" | Set in bay configuration at install time |
| Bilingual (auto-detect) | Either wake word triggers language detection | Optional; increases false-positive risk |

**Recommendation:** Set language mode at bay level, not per-technician, for Phase 4. Per-technician profiles (language tied to headset login) is a Phase 5 improvement.

**False wake rate:** Same 200ms silence gate as English. Spanish wake word evaluated for phonetic collision with shop-floor vocabulary — "oye" is common in casual Spanish speech. Solution: require full two-word wake phrase "Oye Aria" and apply confidence threshold ≥ 0.85 on wake detection.

---

## Quality Assurance

### Evaluation Protocol

1. **Offline evaluation** — labeled Spanish utterance test set (500 utterances per intent, 3 dialects)
   - Pass bar: ≥ 97% intent classification accuracy per dialect
   - Failure mode analysis: which intents or components have highest error rate

2. **Shadow mode pilot** — 2-week period at one beta site with Spanish-speaking technicians
   - Voice commands logged alongside English-mode intent outputs for comparison
   - Recognition errors surfaced to ML team daily

3. **Live pilot** — Spanish mode enabled for 5 Spanish-speaking technicians at one Toyota beta site (Month 17)
   - Voice Command Recognition Rate measured per session
   - Technician feedback via confidence survey (Spanish-language survey form)

### Pass Criteria for Production Release
- ≥ 97% Voice Command Recognition Rate over ≥ 50 Spanish-language sessions
- Zero safety-critical misinterpretations (e.g., "confirm" intent triggered by unrelated speech during HV-STOP)
- Latency p95 ≤ 180ms for first utterance, ≤ 150ms for subsequent utterances

---

## Agent Integration Notes

**VoiceNLP → DiagnosticCore:** `IntentPacket` schema is language-neutral — DiagnosticCore receives the same struct regardless of whether the technician spoke English or Spanish. No downstream agent changes required.

**VoiceNLP → Headset (confirmation prompt):** Confirmation prompts rendered in detected language. Headset TTS must support `es-MX` voice (HoloLens 2 supports Azure Cognitive Services TTS — `es-MX-JorgeNeural` available out of the box).

**SafetyGuard:** `confirm_safety` intent confirmation prompt localized — safety acknowledgment must be unambiguous in both languages. Spanish confirmation voice alert ("ALTO — Sistema de alta tensión detectado") added to SafetyGuard audio alert library.

---

## Rollout Plan

| Month | Milestone |
|---|---|
| 15 | OEM Spanish manual data access approved; training data collection begins |
| 16 | Technician recording sessions at beta sites (consent obtained) |
| 16–17 | Model training + offline evaluation |
| 17 | Shadow mode pilot at one Toyota beta site |
| 17–18 | Live pilot (5 technicians, 50 sessions) |
| 18 | Spanish mode GA — enabled at all Phase 4 bays with Spanish-speaking workforce |

---

## Open Questions

- **OEM approval for Spanish manual corpus:** Toyota and Ford must approve use of Spanish service manuals for model training. Legal review under existing CAD/data licensing framework — same channel as OEM partnership (`docs/legal/oem_partnership_framework.md`).
- **Per-technician language profiles:** Deferred to Phase 5. Phase 4 uses bay-level language configuration.
- **French support:** Not in this spec. Canadian market (Phase 5) will require Canadian French. Architectural decisions here should not foreclose that path.
