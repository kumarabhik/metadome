# Technical Architecture Overview

**Version:** Phase 5 complete
**Audience:** Engineering leads, OEM technical reviewers, enterprise procurement

---

## System Summary

X-Ray Vision Diagnostics is a distributed edge-cloud system. The critical real-time path (render, voice, safety) runs entirely on hardware local to the service bay — no cloud round-trip during active repair. Cloud handles non-real-time functions: CAD updates, telemetry ingestion, dashboards, model improvements.

This architecture is a deliberate constraint. A 40ms latency requirement for the overlay render path cannot be met over cellular or enterprise WiFi to a remote server. Every design decision follows from this: edge is primary; cloud is secondary.

---

## Component Map

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  SERVICE BAY                                                                │
│                                                                             │
│  ┌──────────────┐     BLE/WiFi6E     ┌─────────────────────────────────┐  │
│  │  HoloLens 2  │ ◄────────────────► │      Edge Server                │  │
│  │  (headset)   │                    │  (NVIDIA Jetson AGX Orin)       │  │
│  │              │                    │                                  │  │
│  │  - Renders   │                    │  ┌────────────┐ ┌─────────────┐ │  │
│  │    CAD layers│                    │  │DiagnosticCore│ │SensorFusion │ │  │
│  │  - Captures  │                    │  │  (RAG+LLM) │ │(OBD+thermal)│ │  │
│  │    voice     │                    │  └────────────┘ └─────────────┘ │  │
│  │  - Gaze/     │                    │  ┌────────────┐ ┌─────────────┐ │  │
│  │    gesture   │                    │  │CADRenderer │ │SafetyGuard  │ │  │
│  └──────────────┘                    │  │(SLAM+LOD)  │ │(HV-STOP)    │ │  │
│                                      │  └────────────┘ └─────────────┘ │  │
│  ┌──────────────┐     CAN/OBD-II     │  ┌────────────┐                 │  │
│  │   Vehicle    │ ───────────────► │  │VoiceNLP    │  VIN-indexed    │  │
│  │  (OBD-II     │                    │  │(wake+intent)│  CAD cache     │  │
│  │   port)      │                    │  └────────────┘  (local SSD)   │  │
│  └──────────────┘                    └──────────────┬──────────────────┘  │
│                                                      │ WiFi6E               │
│  ┌──────────────┐    USB/HDMI                       │                      │
│  │ Thermal IR   │ ──────────────────────────────────┘                      │
│  │  Camera      │                                                           │
│  └──────────────┘                                                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                              HTTPS/TLS 1.3
                                       │
┌─────────────────────────────────────────────────────────────────────────────┐
│  CLOUD  (AWS multi-region — us-east-1 primary)                             │
│                                                                             │
│  ┌───────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  CAD Ingestion │  │  Telemetry   │  │  Dashboard   │  │  Recording   │ │
│  │  Pipeline     │  │  Pipeline    │  │  Backend     │  │  Storage     │ │
│  │  (ECS+S3)     │  │(Kinesis+S3)  │  │(ECS+Aurora)  │  │  (S3)        │ │
│  └───────────────┘  └──────────────┘  └──────────────┘  └──────────────┘ │
│  ┌───────────────┐  ┌──────────────┐                                       │
│  │  RAG Corpus   │  │  RLHF        │                                       │
│  │  (OpenSearch) │  │  Pipeline    │                                       │
│  └───────────────┘  └──────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                              HTTPS / partner APIs
                                       │
┌─────────────────────────────────────────────────────────────────────────────┐
│  EXTERNAL SYSTEMS                                                           │
│                                                                             │
│  OEM CAD Storage  │  OEM Telematics  │  CDK / R&R (job cards)             │
│  Toyota TTMS LMS  │  Ford LMS        │  OSHA audit systems                │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Real-Time Path (< 40ms budget)

This is the path that runs during an active repair session. Every component must complete its work within the 40ms total latency budget.

```
Tech voice command
      │
      ▼
VoiceNLP (edge)          ~8ms   Wake word → intent classification → entity extraction
      │
      ▼
DiagnosticCore (edge)    ~12ms  Intent + current DTC + sensor state → LayerManifest
      │
      ▼
CADRenderer (edge)       ~15ms  LayerManifest → select layers from local cache
      │                          → apply SLAM anchor → render delta (changed layers only)
      ▼
HoloLens 2 render        ~5ms   Display updated overlay
      │
Total                    ~40ms
```

**Budget allocation notes:**
- VoiceNLP 8ms: intent classification runs on Jetson GPU (TensorRT-optimized model); no cloud call
- DiagnosticCore 12ms: RAG retrieval from local OpenSearch index on edge SSD; LLM call blocked (LLM used only for pre-cached summaries, not real-time generation)
- CADRenderer 15ms: predictive rendering pre-loads likely next layers; delta render only (full render = 35ms; delta = 15ms)
- HoloLens 2 5ms: display pipeline is fixed; not controllable

**When budget is exceeded:** Graceful degradation triggers at > 40ms p95 (see `docs/features/graceful_degradation_spec.md`)

---

## Safety Path (< 80ms, independent of main render path)

SafetyGuard runs as an isolated process on the edge server. It does not share a thread pool or memory space with DiagnosticCore or CADRenderer. A deadlock or crash in the diagnostic path cannot block SafetyGuard.

```
Dual-sensor polling (10Hz continuous)
      │
      ├── OBD-II: HV system status (isolation monitor, HV relay state)
      └── Thermal camera: proximity heat signature in HV zone
            │
            ▼
      Either sensor crosses threshold
            │
            ▼
      SafetyGuard: generate HV-STOP event                  < 15ms
            │
            ▼
      Send HV-STOP to HoloLens display (highest priority)  < 20ms
      Send HV-STOP to all collaborative session headsets   < 30ms
      Write to audit log (append-only, local SSD)          < 40ms
      Send alert to cloud telemetry (best-effort)          < 80ms
            │
Total worst-case to tech seeing HV-STOP on headset:       < 80ms
```

**Isolation guarantee:** If the edge server crashes (power failure, OS fault), the HoloLens 2 detects loss of connection within 500ms and displays a full-screen safety warning: "Connection lost — do not touch high-voltage systems."

---

## Agent Communication Protocol

All 5 agents communicate via a message bus on the edge server (Redis Pub/Sub, local only). No network hops between agents.

```
Inbound events:
  VehicleConnected { vin, oem, model_year }
  DTCReceived { codes[], severity }
  SensorUpdate { unified_sensor_state }
  VoiceIntent { intent, entities[], confidence }
  GestureEvent { type, coordinates }

Outbound events:
  LayerManifest { layers[], anchor_points[], step_sequence[] }
  HVStopTriggered { source, activation_ms, zones_affected[] }
  SessionMetrics { step_timings[], voice_accuracy, latency_p95 }

Agent subscriptions:
  DiagnosticCore  ← DTCReceived, VoiceIntent
  SensorFusion    ← VehicleConnected, publishes → SensorUpdate
  CADRenderer     ← LayerManifest, SensorUpdate
  SafetyGuard     ← SensorUpdate (independent polling), publishes → HVStopTriggered
  VoiceNLP        ← raw audio stream, publishes → VoiceIntent
```

---

## Data Flows

### CAD Data Flow (non-real-time, cloud → edge)

```
OEM CAD files (STEP/JT/NX format)
      │
      ▼
CAD Ingestion Pipeline (AWS ECS)
  → Normalize to glTF 2.0 (open format, hardware-agnostic)
  → Generate LOD levels (L0=full detail, L1=medium, L2=low — for graceful degradation)
  → Index by VIN range + component ID
  → Store in S3: s3://xvd-cad/{market}/{oem}/{model}/{vin_range}/{component}.glb
      │
      ▼
Edge server sync (nightly, or on-demand for new OEM bulletins)
  → rsync from S3 to local NVMe SSD (/data/cad/)
  → 72-hour local cache guarantees availability during cloud outage
```

### Telemetry Flow (edge → cloud)

```
Edge server session events
  → Local buffer (SQLite, append-only)
  → Batch upload every 5 minutes to regional Kinesis Data Stream
  → Kinesis → S3 (raw events, partitioned by date/dealer)
  → Athena (ad-hoc query for metrics dashboards)
  → QuickSight / internal dashboard (for product and CS teams)
```

### OEM Service Bulletin Flow (cloud → RAG corpus)

```
OEM publishes new TSB (via Partner Portal SFTP)
  → Ingestion Lambda triggered on new file
  → Parse PDF/XML → extract structured repair steps
  → Embed with text-embedding-3-small (OpenAI) → vectors
  → Upsert to OpenSearch Serverless (cloud) + nightly sync to edge OpenSearch
  → DiagnosticCore RAG picks up new bulletin on next edge sync
```

---

## Hardware Specs (Reference)

### HoloLens 2
- Display: 2K 3:2 light engine per eye, 52° diagonal FOV
- Processor: Qualcomm Snapdragon 850 (on-device SLAM, depth processing)
- Memory: 4GB LPDDR4
- Comms: WiFi 5 (802.11ac), Bluetooth 5.0
- Battery: ~2-3 hours active use
- Weight: 566g

### Edge Server (NVIDIA Jetson AGX Orin)
- CPU: 12-core Arm Cortex-A78AE
- GPU: 2048-core Ampere NVIDIA GPU + 64 Tensor Cores
- Memory: 32GB LPDDR5
- Storage: 2TB NVMe SSD (CAD cache + session data)
- Comms: 2.5GbE (bay network), USB 3.2 (OBD-II dongle + thermal camera)
- Power: 15–60W configurable TDP; 24V DC input

### Network
- Bay network: WiFi6E (802.11ax, 6GHz band) — dedicated AP per 2 bays
- WAN: 1Gbps ethernet (enterprise dealership network) or LTE backup
- Firewall: outbound HTTPS only (443); all inbound blocked except management VPN

---

## Failure Modes and Mitigations

| Component | Failure Mode | Detection | Mitigation |
|---|---|---|---|
| HoloLens 2 battery dies mid-session | Session drops | HoloLens sends low-battery warning at 15% | Session state persists on edge server; resume on reconnect |
| Edge server loses power | All bay functions offline | Watchdog heartbeat to cloud; no heartbeat = alert | Tech sees "Connection lost" safety warning; manual fallback procedure |
| WiFi6E AP failure | Headset can't reach edge server | HoloLens detects WiFi disconnect | Cached last-known overlay displayed for 10 seconds; then safety warning |
| OBD-II dongle disconnects | SensorFusion loses vehicle data | SensorFusion detects data gap | SafetyGuard switches to thermal-only mode; warning shown to tech |
| Thermal camera blocked/obscured | SafetyGuard loses thermal feed | SafetyGuard health check every 5s | Switch to OBD-II only mode; warning shown; flag in session log |
| Cloud unreachable | No CAD updates, no telemetry sync | Edge server cloud health check every 60s | Edge cache covers 72h; telemetry buffered locally; session continues |
| DiagnosticCore RAG returns no results | No repair steps surfaced | DiagnosticCore returns empty manifest | Graceful degradation: "Fault type not enrolled — use manual procedure" |
