# X-Ray Vision Diagnostics — Agent Architecture

## Overview

The system is composed of five specialized AI agents that operate in a coordinated pipeline. Each agent owns a specific domain and communicates through a shared state bus on the edge server. No agent directly controls the headset render pipeline — all overlay instructions are routed through the `CADRenderer` agent, which is the single source of truth for what the technician sees.

---

## Status Legend
- `[x]` Designed and specced
- `[~]` Partially defined — needs implementation detail
- `[ ]` Placeholder — not yet specced

---

## Agent 1: DiagnosticCore

**Status:** `[x]`

**Role:** The central reasoning agent. Acts as the "Senior Master Tech." Receives all input signals and produces a structured repair guidance packet consumed by all other agents.

**Inputs:**
- Live DTC codes + PIDs from OBD-II / CAN bus gateway
- VIN → vehicle model, trim, CAD version
- Voice intent packets from VoiceNLP agent
- Thermal camera anomaly coordinates
- Technician action history (steps completed, components touched)

**Outputs:**
- `LayerSelectionManifest` — ordered list of CAD layers to activate/suppress with render priority
- `NarrativeInstruction` — step-by-step repair text for headset HUD + voice synthesis
- `SensorHighlightMap` — which live sensor values to surface spatially and in what color coding
- `ConfidenceScore` — 0.0–1.0 rating of how certain the diagnosis is; rendered in HUD corner

**Key logic:**
- Uses a retrieval-augmented generation (RAG) pipeline: LLM queries are grounded in OEM service bulletins and DTC databases — no freeform repair instruction generation
- Repair steps sourced from OEM-certified data only; LLM handles *narrative presentation*, not *repair truth*
- Confidence degrades gracefully: if confidence < 0.6, system surfaces "Recommend OEM hotline escalation" and reduces step specificity
- Context window includes: active DTCs, vehicle history (prior repairs on this VIN from dealer DMS), thermal anomaly data, and current repair step

**Failure modes handled:**
- No DTC present → falls back to visual query only (what did tech say? what does thermal show?)
- VIN not in CAD database → alerts tech; suppresses overlay; falls back to tablet PDF link
- LLM timeout (>2s) → surfaces last cached guidance packet; displays "AI reconnecting" badge

---

## Agent 2: CADRenderer

**Status:** `[x]`

**Role:** The only agent with write access to the headset render pipeline. Translates `LayerSelectionManifest` from DiagnosticCore into actual 3D overlay instructions anchored to the physical vehicle.

**Inputs:**
- `LayerSelectionManifest` from DiagnosticCore
- SLAM pose data from headset (head position, gaze direction, 6-DOF)
- ArUco marker positions (used to anchor coordinate system to specific vehicle)
- `SafetyOverride` signals from SafetyGuard agent (highest priority — always applied)

**Outputs:**
- Rendered holographic overlay (gLTF 2.0 layer streams to headset GPU)
- Spatial arrow glyphs (step guidance anchors)
- Color-coded component highlights:
  - Red pulsing: fault confirmed
  - Amber: fault suspected / sensor degraded
  - Blue: active system path (e.g., coolant routing)
  - Green: verified good / step complete
  - Red solid + full-screen dim: HV-STOP safety alert

**Key logic:**
- Maintains a **layer budget**: maximum 4 simultaneous CAD layers to prevent cognitive overload (validated by UX research across Boeing and Renault AR deployments)
- Priority stack: Safety layers > Active fault layers > Guidance arrows > Informational layers
- Pre-caches top 3 most probable layer combinations on bay entry (based on open DTCs) to reduce render latency on first voice query
- Supports **LOD (Level of Detail)** management: renders simplified geometry when latency > 35ms; full detail when latency < 20ms

**Performance targets:**
- Layer swap: < 200ms
- SLAM re-anchor: < 500ms
- Safety overlay activation: < 80ms (hard real-time)

---

## Agent 3: SensorFusion

**Status:** `[x]`

**Role:** Aggregates and normalizes all real-time vehicle data streams into a unified sensor state object consumed by DiagnosticCore and SafetyGuard.

**Data sources managed:**

| Source | Protocol | Refresh Rate | Data Type |
|---|---|---|---|
| OBD-II port | J1979 / ISO 15765-4 | 10Hz | DTCs, live PIDs (temp, flow, voltage, RPM) |
| CAN bus | CAN FD 2.0B | 1kHz | Raw frame data (filtered to relevant signals) |
| Thermal camera | USB3 / GigE Vision | 30fps | IR heat map, hotspot coordinates |
| Manufacturer telematics | REST API (OEM-specific) | On-demand | Extended fault history, OTA update state |
| Headset depth camera | Proprietary SDK | 60fps | Hand/tool 3D position, proximity calculations |

**Outputs:**
- `UnifiedSensorState` object — normalized, timestamped, tagged with confidence per reading
- `ProximityAlert` events — streamed to SafetyGuard at 60Hz
- Anomaly flags: readings outside 2-sigma of vehicle-model-normal emit an alert tag

**Key logic:**
- All data normalized against vehicle-model-specific baseline values (not generic defaults)
- Stale data handling: if any source is offline > 3 seconds, reading is flagged as stale and DiagnosticCore reduces confidence accordingly
- Writes raw sensor logs to edge server for post-repair audit (OSHA + OEM warranty compliance)

---

## Agent 4: SafetyGuard

**Status:** `[x]`

**Role:** A dedicated safety enforcement agent that runs on an independent process with real-time process priority. Cannot be paused, overridden by technician, or deprioritized by the main pipeline.

**Monitors:**
- Hand/tool proximity to known HV bus locations (from CAD + depth camera — dual-source confirmation required)
- HV bus live voltage state (from CAN bus — confirms bus is energized before triggering stop)
- Technician acknowledgment state (has tech confirmed safety checklist for this vehicle?)
- Environmental hazard flags (e.g., fuel vapor sensors if applicable)

**Actions:**

| Trigger | Response | Escalation |
|---|---|---|
| Hand within 300mm of HV bus | Amber ring renders around HV component; audio tone | None (warning only) |
| Hand within 150mm of HV bus | Full HV-STOP: red overlay, alarm, voice alert, render pipeline paused | Log created; manager dashboard notified |
| HV-STOP not acknowledged within 10s | Escalate to service manager app; bay locked in system | Manager must remotely clear |
| Tech skips required safety checklist item | Step blocked; cannot advance until acknowledged | Checklist item flagged in audit log |

**Hard constraints:**
- SafetyGuard process cannot be killed by technician or by any other agent
- HV-STOP is non-bypassable at headset level; only a remote manager role can clear
- All events logged with: tech ID, vehicle VIN, timestamp, GPS bay location, action taken
- Logging cannot be disabled — it is outside the main application permission scope

**Regulatory compliance targets:**
- OSHA 29 CFR 1910.147 (Lockout/Tagout) logging requirements
- SAE J1742 (EV Safety Standards) proximity alert specifications
- NFPA 70E (Electrical Safety in the Workplace) HV zone definitions

---

## Agent 5: VoiceNLP

**Status:** `[~]` — Intent taxonomy defined; entity extraction needs expansion for multi-make vocabulary

**Role:** Converts raw technician speech into structured intent packets consumed by DiagnosticCore. Handles automotive-domain vocabulary, noisy shop floor audio, and partial / interrupted utterances.

**Architecture:**
- Wake word: "Hey Aria" (configurable per dealer to avoid collision with other voice systems)
- Base model: fine-tuned Whisper-large-v3 for automotive domain vocabulary
- Intent classifier: lightweight BERT-family model running on edge (not cloud) for <150ms latency
- Entity extractor: custom NER trained on OEM service manual vocabulary, DTC codes, component names

**Supported intent categories:**

| Intent | Example Utterances | Output |
|---|---|---|
| `show_system` | "Show me the coolant," "Display the battery pack" | {action: show, system: coolant/battery} |
| `navigate_step` | "Next step," "Go back," "Skip this" | {action: navigate, direction: next/back/skip} |
| `request_explanation` | "Why is this failing?" "What does P0A93 mean?" | {action: explain, target: current_fault/DTC} |
| `flag_observation` | "Flag this for supervisor," "Add a note: hose is cracked" | {action: flag, severity: normal/urgent, note: text} |
| `confirm_safety` | "HV aware," "Gloves on," "I confirm" | {action: safety_ack, context: active_stop_gate} |
| `end_session` | "Close diagnostic," "I'm done" | {action: session_end} |

**Failure handling:**
- Confidence < 0.75 on intent: system repeats back interpreted intent — "Did you mean: Show coolant system?" — requires confirmation
- Ambient noise causing false wake: 200ms silence gate after wake word before command processing
- Unknown command: "I didn't catch that — try 'next step,' 'show [system],' or 'explain this'"

**Planned improvements (not in v1):**
- Multi-technician voice disambiguation (when two techs work same vehicle)
- Continuous passive listening for anomaly commentary ("this is really hot in here") parsed as observation input to DiagnosticCore
- Language localization (Spanish, French, German) for global dealer network

---

## Agent Communication Map

```
                      ┌─────────────────┐
                      │   VoiceNLP      │
                      │ (Intent Parser) │
                      └────────┬────────┘
                               │ IntentPacket
                               ▼
┌─────────────────┐   ┌────────────────────┐   ┌─────────────────┐
│  SensorFusion   │──▶│  DiagnosticCore    │──▶│  CADRenderer    │
│ (Data Aggregator│   │ (Reasoning Engine) │   │ (Render Pipeline│
│  + Alerts)      │   └────────────────────┘   └─────────────────┘
└────────┬────────┘                                      ▲
         │ ProximityAlert (60Hz)                         │ SafetyOverride
         ▼                                               │ (highest priority)
┌─────────────────────────────────────────────────────────────────┐
│                        SafetyGuard                               │
│               (Independent real-time process)                    │
└─────────────────────────────────────────────────────────────────┘
```

**Communication protocol:**
- All inter-agent messages are typed structs (Protobuf v3) over local gRPC on edge server
- SafetyGuard → CADRenderer channel is a dedicated out-of-band channel with real-time priority
- No agent communicates directly with the cloud during an active repair step (all cloud data pre-fetched)
- Agent health is monitored by a watchdog process; any agent crash triggers headset alert and graceful degradation
