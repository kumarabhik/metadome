# DiagnosticCore v1 — Implementation Specification
## Step 11 — DTC Ingestion, RAG Pipeline, Layer Manifest Generation

**Roadmap reference:** Phase 2, Agent Build 1
**Status:** `[x]` Spec complete
**Owner:** ML Engineering Lead
**Target:** Working v1 by Month 6

---

## Responsibility

DiagnosticCore is the central reasoning engine. It receives all sensor and intent signals, runs them through a Retrieval-Augmented Generation (RAG) pipeline grounded in OEM service data, and produces a structured `LayerSelectionManifest` telling CADRenderer exactly what to render.

**Hard constraint:** DiagnosticCore may never generate freeform repair instructions. Every repair step must be retrieved from OEM-sourced service bulletins. The LLM's role is narrative presentation only — it wraps OEM text in natural language, it does not invent repair procedures.

---

## v1 Scope (Phase 2)

**In scope:**
- Single vehicle: Toyota bZ4X (2022–2024)
- Single fault family: Battery Thermal Management System (TMS) faults
- DTCs covered: P0A93, P0A94, P1DF1, P0A9C, P0A80, P0A81
- Voice intent handling: `show_system`, `navigate_step`, `request_explanation`
- Confidence scoring: output 0.0–1.0 per diagnosis

**Out of scope for v1:**
- Multi-vehicle support (Phase 3)
- Predictive fault detection without active DTC (Phase 5)
- RLHF fine-tuning loop (Phase 4)

---

## Technology Stack

| Component | Technology | Rationale |
|---|---|---|
| LLM backbone | Claude claude-haiku-4-5 (Anthropic API) | Fast inference < 800ms; sufficient for narrative wrapping; cost-effective for per-repair calls |
| Embedding model | `text-embedding-3-small` (OpenAI) or `nomic-embed-text` (local) | Local preferred for latency; OpenAI fallback if local embedding quality insufficient |
| Vector store | Chroma DB (local, embedded) | Runs on Jetson AGX Orin; no external dependency; sufficient for Toyota TMS corpus |
| Retrieval | Hybrid: dense (embedding cosine similarity) + sparse (BM25 keyword) | Hybrid retrieval outperforms dense-only on technical documents with part numbers/DTC codes |
| Orchestration | LangChain 0.2 (Python) | Mature RAG tooling; good Chroma + Anthropic integration |
| API server | FastAPI (Python 3.11) | Async; low overhead; easy gRPC bridge via grpcio |
| Runtime | Edge server (Jetson AGX Orin) for latency-sensitive path; Anthropic cloud API for LLM calls |

---

## RAG Corpus (v1)

**Source documents (Toyota bZ4X TMS):**

| Document Type | Source | Processing |
|---|---|---|
| OEM Service Bulletins (TSBs) | Toyota TechStream (via data licensing agreement) | PDF → text extraction → chunked at 512 tokens, 50-token overlap |
| Workshop Manual sections | Toyota eTC (electronic Technical Center) | Same as above |
| DTC troubleshooting trees | Toyota Techinfo export | Structured JSON → converted to prose chunks for embedding |
| Repair time standards (flat-rate) | Mitchell1 ProDemand | Tabular → prose |

**Corpus size (v1 estimate):** ~850 chunks for Toyota bZ4X TMS subsystem

**Chunking strategy:**
- Split at section boundaries (OEM manuals are section-structured)
- Each chunk tagged with: `{document_id, section, dtc_relevance: [list], subsystem_tag}`
- DTC-tagged chunks always retrieved first when a matching DTC is active (boosted relevance score)

---

## Data Flow

```
[Inputs]
  UnifiedSensorState (from SensorFusion)    ← 100ms refresh
  IntentPacket (from VoiceNLP)              ← on voice event
  VehicleContext {VIN, model, year, trim}   ← set at session start

        ↓
[1] CONTEXT BUILDER
    Constructs retrieval query string:
    "Toyota bZ4X 2024 AWD, active DTCs: P0A93 P1DF1,
     coolant_flow_lpm=1.2 (low), battery_temp_avg=94C (high),
     thermal_hotspot: module_4. Tech query: show coolant leak"

        ↓
[2] RETRIEVER (Hybrid RAG)
    Dense: top-5 chunks by cosine similarity to query embedding
    Sparse: top-5 chunks by BM25 keyword match (P0A93, coolant, pump)
    Fusion: Reciprocal Rank Fusion (RRF) → top-6 unique chunks

        ↓
[3] CONTEXT VALIDATOR
    Every retrieved chunk checked:
    ✓ Is it from an OEM-authorized source? (source tag check)
    ✓ Does it apply to this exact VIN configuration? (trim/year filter)
    ✗ Chunks from generic aftermarket sources → rejected

        ↓
[4] LLM CALL (Claude claude-haiku-4-5 or Sonnet)
    System prompt: "You are a Senior Master Technician AI. Using ONLY the provided
    service bulletin excerpts, generate repair guidance. Never invent procedures.
    If the answer is not in the excerpts, say so explicitly."

    User prompt: [query + validated context chunks + sensor state summary]

    Output schema (structured, via tool_use):
    {
      "diagnosis_summary": str,       // 1-2 sentence plain English
      "confidence": float,            // 0.0–1.0
      "steps": [{
        "step_number": int,
        "instruction": str,           // OEM text, lightly rephrased for voice
        "source_bulletin": str,       // document ID for audit
        "component_id": str,          // matches CAD layer anchor point
        "estimated_minutes": int
      }],
      "layer_manifest": LayerSelectionManifest,
      "escalate": bool                // true if confidence < 0.6
    }

        ↓
[5] MANIFEST BUILDER
    Builds LayerSelectionManifest from diagnosis:
    - Maps component_ids from steps → CAD layer IDs
    - Sets render priority (fault layers > guidance layers > reference layers)
    - Assigns colors per layer state
    - Includes SensorHighlightMap (which PIDs to surface spatially)

        ↓
[Output]
    LayerSelectionManifest → CADRenderer
    NarrativeInstruction → Headset HUD + TTS
    ConfidenceScore → Headset HUD badge
    EscalationFlag → Manager dashboard (if true)
```

---

## LayerSelectionManifest Schema

```python
@dataclass
class LayerSelectionManifest:
    session_id: str
    timestamp_ms: int
    vehicle_vin: str
    confidence: float

    layers: list[LayerDirective]  # ordered by render priority
    sensor_highlights: list[SensorHighlight]
    guidance_arrows: list[GuidanceArrow]
    suppress_layers: list[str]    # explicitly suppress these layer IDs

@dataclass
class LayerDirective:
    layer_id: str                 # e.g. "tms_coolant_loop"
    render_state: str             # "active_fault" | "active_path" | "reference" | "suppressed"
    color_override: str | None    # hex; None = use layer default
    opacity: float                # 0.0–1.0
    animation: str | None         # "pulse_1hz" | "flow_direction" | None
    lod_hint: str                 # "lod0" | "lod1" | "auto"

@dataclass
class GuidanceArrow:
    anchor_point_id: str          # maps to CAD anchor point (e.g. "coolant_inlet_M4")
    label: str                    # displayed near arrow tip
    step_number: int
    color: str
```

---

## Confidence Scoring Logic

| Condition | Confidence Effect |
|---|---|
| Active DTC matches retrieved bulletin exactly | +0.30 |
| Thermal hotspot position matches DTC-indicated module | +0.25 |
| Sensor value anomaly corroborates DTC (e.g., low flow + high temp) | +0.20 |
| Retrieved chunk confidence (cosine score) ≥ 0.85 | +0.15 |
| VIN matches exact bulletin applicability | +0.10 |
| No DTC present (voice query only) | −0.25 |
| Conflicting DTCs (multiple subsystems) | −0.15 |
| Retrieved chunk is flagged "draft" or "superseded" | −0.30 |
| No applicable bulletin found | → confidence = 0.0, escalate = true |

If `confidence < 0.6`: system narrows guidance to safest possible action ("Recommend OEM hotline — DTC pattern requires Level 3 support") and sets `escalate = true`.

---

## Latency Budget

| Stage | Target p50 | Target p95 |
|---|---|---|
| Context builder | < 5ms | < 10ms |
| Hybrid retrieval | < 30ms | < 60ms (Chroma local) |
| Context validation | < 5ms | < 10ms |
| LLM call (Haiku) | < 500ms | < 900ms |
| Manifest builder | < 10ms | < 20ms |
| **Total** | **< 550ms** | **< 1000ms** |

LLM call is the dominant cost. Mitigation: pre-warm DiagnosticCore at session start with open DTCs — by the time tech asks a question, context is pre-fetched and a speculative inference can be cached.

---

## Audit Requirements

Every DiagnosticCore output logged to edge server with:
- Session ID, VIN, timestamp
- Retrieved chunk IDs + scores (full retrieval trace)
- LLM input prompt hash (not full text — for privacy)
- LLM output JSON (verbatim)
- Confidence score
- Whether escalation was triggered

Retention: 12 months on edge server; 36 months in cloud archive. Required for OEM warranty audit and liability protection.
