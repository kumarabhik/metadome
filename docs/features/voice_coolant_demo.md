# Voice Command Feature Spec
## Step 17 — "Show me the coolant leak" → Layer Activation → Guided Repair Steps

**Roadmap reference:** Phase 2, Feature 2
**Status:** `[x]` Spec complete
**Owner:** PM + ML Engineer
**Target:** Demonstrable in Month 7 alpha

---

## Feature Summary

A single voice command — `"Show me the coolant leak"` — triggers a complete, intelligent response chain: intent parsing, sensor cross-referencing, CAD layer selection, 3D spatial overlay activation, and the first guided repair step delivered via voice. This is the "hero demo" that proves the core product concept end-to-end.

---

## Command Variants (All Must Work)

The system must handle natural language variation. These are all equivalent and must produce the same response:

| Utterance | Notes |
|---|---|
| `"Show me the coolant leak"` | Canonical form |
| `"Where's the coolant leak?"` | Question form |
| `"Show coolant"` | Short form |
| `"Display the coolant circuit"` | Technical form |
| `"Show me what's leaking"` | Anomaly-focused, no system name |
| `"Coolant leak, where is it?"` | Inverted order |
| `"Hey Aria, show me the cooling system"` | With wake word inline |

**VoiceNLP must map all of these to:** `{intent: "show_system", entity: {system: "tms_coolant_loop", anomaly: "leak"}}`

---

## Detailed Signal Chain

```
STEP 1 — WAKE WORD
  Input: Tech says "Show me the coolant leak" (wake word omitted — auto-activated after first command)
  VoiceNLP: wake word state = ACTIVE (already awake from session start)
  Action: Audio capture begins immediately

STEP 2 — SPEECH TO TEXT
  Audio: "Show me the coolant leak" (1.8s clip)
  Whisper: transcript = "show me the coolant leak"
  Latency: 145ms

STEP 3 — INTENT CLASSIFICATION
  Input: "show me the coolant leak"
  Model: distilbert-base-uncased (automotive fine-tuned)
  Output: intent="show_system", confidence=0.97
  Latency: 12ms

STEP 4 — ENTITY EXTRACTION
  Input: "show me the coolant leak"
  NER patterns:
    "coolant" → SYSTEM → maps to layer_id: "tms_coolant_loop"
    "leak" → ANOMALY → fault filter: "leak|flow|rupture"
  Output: {system: "tms_coolant_loop", anomaly: "leak"}
  Latency: 6ms

STEP 5 — INTENT PACKET PUBLISHED
  IntentPacket → DiagnosticCore gRPC channel
  Latency: 2ms

STEP 6 — DIAGNOSTIC CORE PROCESSING
  Input: IntentPacket + current UnifiedSensorState:
    - DTCs: [P0A93, P1DF1]
    - coolant_flow_lpm: 1.2 (anomaly: low)
    - battery_temp_avg_c: 94 (anomaly: high)
    - thermal_hotspots: [{centroid: module_4_position, temp: 91.3°C}]

  RAG retrieval query:
    "Toyota bZ4X coolant leak, P0A93, P1DF1, module 4 hotspot, low flow rate"

  Retrieved chunks (top 3):
    1. TSB-EV-2024-047: "P1DF1 coolant pump fault — check inlet coupling at affected module"
    2. Workshop Manual §12.4.3: "Module 4 coolant circuit inspection procedure"
    3. TSB-EV-2024-031: "P0A93 temp sensor — caused by coolant loss near sensor harness"

  LLM output:
    diagnosis: "Low coolant flow (1.2 vs 3.8 L/min nominal) with thermal anomaly at
                Module 4 indicates inlet coupling leak. P1DF1 confirms pump performance
                degraded — likely downstream of leak site."
    confidence: 0.91
    steps: [7 steps from TSB-EV-2024-047, lightly narrated]
    layer_manifest: {
      activate: [tms_coolant_loop, bms_module_04, coolant_pump_assy],
      suppress: [hvac_chiller, bms_modules_others],
      guidance_arrow: coolant_inlet_M4 ("Module 4 Inlet — Check Here")
    }

  Latency: 520ms (Haiku, including retrieval)

STEP 7 — LAYER MANIFEST APPLIED (CADRenderer)
  tms_coolant_loop: ACTIVE PATH (blue, flow direction animation)
  bms_module_04: ACTIVE FAULT (red, 1Hz pulse)
  coolant_pump_assy: ACTIVE FAULT (amber, static)
  body_ghost: remains at 12% opacity (reference)
  hv_bus: remains at ambient orange (always-on safety layer)
  Latency: 85ms (manifest apply)

STEP 8 — ARIA VOICE + HUD
  TTS output (< 200ms to first audio byte):
  "I've highlighted the coolant circuit in blue. I'm detecting low flow and a heat spike
   at Module 4 — the red pulsing component at your 2 o'clock, about 18 inches in.
   The most likely cause is the Module 4 inlet coupling. Say 'start repair' when ready."

  HUD:
  ┌─────────────────────────────────────┐
  │ COOLANT SYSTEM — FAULT DETECTED     │
  │ Flow: 1.2 L/min ↓ (normal: 3.8)    │
  │ Module 4 temp: 94°C ↑ (normal: 45) │
  │ Confidence: 91% ●●●●○              │
  └─────────────────────────────────────┘

  Step counter not yet visible — activates after tech says "start repair" or nods.

TOTAL LATENCY (end of speech → overlay visible + Aria speaking):
  STT: 145ms
  Intent + entity: 18ms
  gRPC: 2ms
  DiagnosticCore: 520ms
  Manifest apply: 85ms
  TTS first byte: 80ms
  ─────────────────
  TOTAL: ~850ms (within 1s target)
```

---

## 7-Step Guided Repair Sequence (OEM-Sourced, TSB-EV-2024-047)

Each step: Aria narrates → guidance arrow repositions → tech completes → "next step" → advance.

| Step | Aria Narration | Arrow Target | Live Sensor Shown |
|---|---|---|---|
| 1 | "Inspect the Module 4 inlet coupling. Look for cracking or residue at the fitting at your 2 o'clock." | `coolant_inlet_M4` | Module 4 temp (°C) |
| 2 | "Check the hose clamp on the inlet hose. It should be tight with no corrosion." | `hose_clamp_M4_inlet` | None |
| 3 | "Remove the protective shield over Module 4 — two M6 bolts, ratchet 10mm. Set it aside." | `shield_M4_bolts` | None |
| 4 | "With the shield removed, inspect the coupling body. If you see a crack or weeping coolant, this is your failure point. Take a photo for the warranty claim." | `coolant_inlet_M4` | None |
| 5 | "Replace the inlet coupling — part number 16271-77010. Torque the fitting to 8 N·m. Do not overtighten." | `coolant_inlet_M4` | Torque reminder in HUD |
| 6 | "Reinstall the shield. Torque M6 bolts to 10 N·m." | `shield_M4_bolts` | None |
| 7 | "Run the coolant pump test via TechStream. Watch flow rate here — it should return above 3.5 L/min." | None | Coolant flow (L/min, live) |

---

## HUD During Guided Repair

```
┌─────────────────────────────────────────────┐
│  COOLANT REPAIR — MODULE 4 INLET COUPLING   │
│                                              │
│  Step 3 of 7                          [91%] │
│  ────────────────────────────────────────── │
│  Remove Module 4 shield                     │
│  2× M6 bolts, 10mm ratchet                  │
│                                              │
│  [← Back]  [Skip →]  [Flag for Supervisor] │
│                                              │
│  Live: Module 4: 89°C  Flow: 1.2 L/min     │
└─────────────────────────────────────────────┘
```

Navigation buttons are eye-trackable (gaze + 800ms dwell). Voice commands also work ("next step," "go back," "skip").

---

## TTS Voice Design (Aria)

**Voice selection:** Neural TTS — target a calm, measured voice that reads as authoritative without being robotic. Recommended: AWS Polly "Matthew" (Neural) or ElevenLabs "Adam" voice. To be finalized after user testing.

**Speech rate:** 145 words per minute (slightly below normal conversational pace — easier to follow while hands are busy)

**Prosody rules:**
- Component names and part numbers spoken slower (e.g., "Module — Four — inlet — coupling")
- Safety warnings always spoken at higher volume (+3dB) and slower rate (120 WPM)
- Step completion acknowledged with a short tone (440Hz, 80ms) before Aria begins next narration

---

## Acceptance Criteria

| Criterion | Pass Threshold |
|---|---|
| All 7 command variants recognized correctly | 100% on 3 runs per variant |
| End-of-speech to overlay visible | < 1000ms p95 |
| End-of-speech to Aria first audio byte | < 800ms p95 |
| Correct layers activated (no spurious layers) | 100% |
| Guidance arrow at correct anchor point | 100% |
| All 7 repair steps navigable by voice alone | 100% (no touch required) |
| HUD live sensor data visible and updating | 10Hz confirmed by QA |
