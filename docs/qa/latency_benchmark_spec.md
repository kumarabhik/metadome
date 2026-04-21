# Internal Latency Benchmarking Spec
## Step 21 — QA: End-to-End Latency Under Load

**Roadmap reference:** Phase 2, QA 1
**Status:** `[ ]` Not started
**Owner:** Engineering Lead + QA
**Target:** Complete by end of Month 7 before Phase 2 exit gate

---

## Objective

Document actual end-to-end latency for every agent interaction in the full diagnostic pipeline under realistic concurrent load. Validate or revise the predictive rendering strategy defined in `design_doc.md`. Results gate Phase 2 exit.

**Pass threshold:** p95 end-to-end voice-to-overlay latency < 40ms under 2-concurrent-session load in test bay.

---

## Latency Segments to Measure

| Segment | Start Event | End Event | Target | Budget |
|---|---|---|---|---|
| Voice capture → VoiceNLP intent | Mic silence end | `IntentEvent` published | < 350ms | 350ms |
| IntentEvent → DiagnosticCore manifest | `IntentEvent` received | `LayerSelectionManifest` published | < 800ms | 800ms |
| SensorFusion poll → UnifiedSensorState | OBD-II poll start | State published | < 200ms | 200ms |
| DiagnosticCore RAG retrieval | Query sent to Chroma | Top-k chunks returned | < 500ms | 500ms |
| DiagnosticCore LLM call | Prompt sent to Claude API | First token received | < 600ms | 600ms |
| CADRenderer manifest → frame rendered | Manifest received | First frame on display | < 16ms (60fps) | 16ms |
| SafetyGuard HV-STOP activation | Proximity trigger | Audio kill + visual stop | < 80ms | 80ms |
| **Voice → visible overlay (full path)** | **Mic silence end** | **New CAD layer visible** | **< 40ms perceived** | **1,966ms total** |

> Note: The 40ms perceptual target applies to the visual rendering update, not the full pipeline. Predictive pre-rendering means CADRenderer starts before DiagnosticCore finishes. See `design_doc.md §Latency Management`.

---

## Test Scenarios

### Scenario A: Cold Start (Worst Case)
- No CAD pre-cached for bZ4X
- No speculative RAG pre-warm
- Single session
- Measures: full pipeline from cold

### Scenario B: Warm Start (Nominal)
- bZ4X CAD pre-cached in GPU memory
- Speculative RAG pre-warmed (DiagnosticCore pre-ran likely queries on VIN load)
- Single session
- Measures: operational steady-state

### Scenario C: Concurrent Load (Stress)
- 2 headset sessions simultaneously on same edge server
- Both warm-started
- Alternating voice commands at 10-second intervals
- Measures: resource contention impact

### Scenario D: Network Degradation
- WiFi6E signal artificially degraded to simulate 10Mbps (vs. 1.2Gbps nominal)
- OBD-II latency injected: +50ms
- Measures: graceful degradation trigger threshold

### Scenario E: LLM API Latency Spike
- Claude API throttled via mock proxy to simulate 2,000ms response
- Measures: predictive rendering fallback behavior

---

## Measurement Methodology

### Instrumentation
Every agent publishes OpenTelemetry (OTEL) spans on every inbound/outbound event. The edge server runs a local Jaeger instance (port 16686) to collect traces without external network dependency.

```
AgentSpan = {
  trace_id: uuid,
  agent: "DiagnosticCore" | "SensorFusion" | ...,
  event_in: str,
  event_out: str,
  t_start_ms: int,
  t_end_ms: int,
  session_id: str
}
```

### Sample Size
- 50 command sequences per scenario
- Commands: ["Show me the coolant system", "Highlight the hot zone", "Next step", "What is the risk level", "Show battery modules"]
- Randomized order per run to avoid warm-cache bias

### Ground Truth Timing
- Video capture at 240fps on a GoPro Hero 12 pointed at headset display
- Lip mic audio track synced via clap marker at run start
- Frame-level timestamps extracted via ffprobe to confirm perceived latency

### Reporting Format

```
| Scenario | Segment | p50 (ms) | p95 (ms) | p99 (ms) | Max (ms) | Pass? |
|---|---|---|---|---|---|---|
```

---

## Pass/Fail Criteria

| Test | Pass Condition | Fail Action |
|---|---|---|
| Voice → overlay (warm, single session) | p95 < 40ms perceived | Revise predictive rendering; re-benchmark before Phase 3 |
| HV-STOP activation | p95 < 80ms | BLOCK Phase 3 — safety critical |
| Concurrent load (2 sessions) | p95 degradation < 25% vs. single | Scale edge server spec; Jetson AGX Orin may need upgrade to 64GB RAM |
| Network degradation trigger | Graceful degradation activates at correct threshold | Fix degradation detection logic |
| LLM API spike | Stale overlay displayed without crash; user notified | Fix fallback rendering path |

---

## Deliverables

- `docs/qa/latency_benchmark_results.md` — raw measurements, scenario-by-scenario
- `docs/qa/latency_benchmark_analysis.md` — root cause for any misses; revised strategy
- Update `design_doc.md §Latency Management` if strategy requires revision
- Go/no-go recommendation to Phase 3 signed by Engineering Lead

---

## Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| Claude API p95 > 600ms on production traffic | Medium | Pre-cache OEM repair steps; use streaming tokens to begin display before full response |
| Jetson AGX Orin memory pressure under 2 sessions | Medium | Profile memory per session; reduce CAD mesh resolution for concurrent sessions |
| ArUco anchor drift causing re-render spike | Low | Anchor confidence threshold filters; SLAM anchor takes over at confidence > 0.85 |
