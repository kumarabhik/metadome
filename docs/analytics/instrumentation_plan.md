# Product Analytics Instrumentation Plan

**Covers:** All user-facing events across headset, manager dashboard, and edge server
**Event tracking library:** Segment (server-side) → destinations: Amplitude (product analytics), S3 (data warehouse), Salesforce (CS health scores)

---

## Instrumentation Philosophy

Track events that answer product questions we know we'll ask. Don't track everything — every extra event is noise in the data and a privacy surface to manage. The questions that matter most:

1. Are techs using the product? (adoption / activation)
2. Is it working correctly? (FTFR, latency, voice recognition)
3. Where do techs drop off? (repair flow completion funnel)
4. Is the product safe? (HV-STOP events)
5. Are techs getting better over time? (Apprentice Mode progression)

---

## Event Taxonomy

### Session Events (Edge Server → Segment)

| Event Name | Trigger | Key Properties |
|---|---|---|
| `session_started` | Tech puts on headset + VIN confirmed | `session_id`, `tech_id`, `vin`, `vehicle_model`, `dealer_id`, `mode` (standard/apprentice/collaborative) |
| `session_ended` | Session completed or abandoned | `session_id`, `duration_seconds`, `steps_completed`, `steps_total`, `completed` (bool) |
| `fault_type_selected` | Tech initiates fault flow | `session_id`, `fault_type`, `dtc_codes[]`, `confidence_tier` (high/medium/low) |
| `step_started` | Step card displayed | `session_id`, `step_number`, `step_type` (standard/safety_critical) |
| `step_completed` | Tech says "Done" and confirmed | `session_id`, `step_number`, `dwell_seconds` |
| `step_skipped` | Tech skips a step | `session_id`, `step_number` — alert: skipping safety steps triggers separate alert event |
| `session_abandoned` | Session closed without completing all steps | `session_id`, `last_step_completed`, `abandonment_reason` (voice/manual/unknown) |

### Voice Events (VoiceNLP → Segment)

| Event Name | Trigger | Key Properties |
|---|---|---|
| `voice_command_received` | Wake word detected + command captured | `session_id`, `raw_transcript`, `intent_classified`, `confidence_score` |
| `voice_command_executed` | Intent successfully acted on | `session_id`, `intent`, `action_taken` |
| `voice_command_failed` | Intent not recognized or ambiguous | `session_id`, `raw_transcript`, `failure_reason` (no_intent/low_confidence/ambient_noise) |
| `voice_command_corrected` | Tech repeats command after failure | `session_id`, `original_transcript`, `corrected_transcript` |

### Safety Events (SafetyGuard → Segment) — highest priority, never sampled

| Event Name | Trigger | Key Properties |
|---|---|---|
| `hv_zone_proximity_warning` | Tech within 1.5m of HV system | `session_id`, `distance_meters`, `hv_system` (battery/inverter/charger) |
| `hv_stop_triggered` | SafetyGuard fires HV-STOP | `session_id`, `trigger_source` (thermal/obd2/both), `activation_latency_ms` |
| `hv_stop_acknowledged` | Tech confirms isolation complete | `session_id`, `acknowledgment_latency_seconds`, `acknowledged_by` (tech_id) |
| `hv_stop_overridden` | Supervisor overrides HV-STOP (emergency only) | `session_id`, `override_by`, `reason` — **this event triggers immediate alert to compliance team** |

### Diagnostic Events (DiagnosticCore → Segment)

| Event Name | Trigger | Key Properties |
|---|---|---|
| `layer_manifest_generated` | DiagnosticCore returns layer manifest | `session_id`, `fault_type`, `layers_selected[]`, `oem_bulletin_references[]`, `generation_latency_ms` |
| `predictive_alert_shown` | Predictive alert surfaced to tech | `session_id`, `alert_id`, `confidence_tier`, `sensor_pid`, `drift_score` |
| `predictive_alert_accepted` | Tech adds alert to repair order | `session_id`, `alert_id` |
| `predictive_alert_dismissed` | Tech dismisses alert | `session_id`, `alert_id` |

### Survey / Feedback Events

| Event Name | Trigger | Key Properties |
|---|---|---|
| `nps_survey_shown` | Post-session NPS survey displayed | `session_id`, `tech_id` |
| `nps_survey_completed` | Tech submits NPS response | `session_id`, `tech_id`, `score`, `category`, `follow_up_choice` |
| `nps_survey_dismissed` | Tech says "Skip survey" | `session_id`, `tech_id` |
| `confidence_survey_completed` | Phase 3 confidence survey submitted | `session_id`, `tech_id`, `composite_score`, `component_scores{}` |

### Manager Dashboard Events (Web → Segment)

| Event Name | Trigger | Key Properties |
|---|---|---|
| `dashboard_viewed` | Manager opens dashboard | `user_id`, `dealer_id`, `view` (overview/ftfr/training/recordings) |
| `recording_viewed` | Manager plays a session recording | `user_id`, `session_id` |
| `recording_exported` | Manager exports recording for OEM | `user_id`, `session_id`, `oem_claim_reference` |
| `tech_profile_updated` | Manager changes tech mode (standard/apprentice) | `user_id`, `tech_id`, `mode_change` |

---

## Key Metrics Derived from Events

| Metric | Derivation |
|---|---|
| **DAU / licensed bays** | Unique `session_started.dealer_id` per day / licensed bays |
| **Session completion rate** | `session_ended.completed = true` / total `session_started` |
| **Step completion funnel** | `step_completed` counts by `step_number` — drop-off shows where techs abandon |
| **Voice recognition rate** | `voice_command_executed` / (`voice_command_executed` + `voice_command_failed`) |
| **Voice command failure taxonomy** | Group `voice_command_failed.failure_reason` |
| **HV-STOP activation latency** | `hv_stop_triggered.activation_latency_ms` — must be < 80ms p99 |
| **Average session duration** | Mean of `session_ended.duration_seconds` by fault type |
| **NPS** | Calculated from `nps_survey_completed.score` — % promoters minus % detractors |
| **Predictive alert acceptance rate** | `predictive_alert_accepted` / `predictive_alert_shown` by confidence tier |

---

## Sampling Policy

- Safety events (`hv_*`): **never sampled** — 100% capture, all environments
- Session events: **100% capture** — core product funnel
- Voice events: **100% capture** — needed for recognition rate calculation
- Diagnostic events: **100% capture** — needed for accuracy measurement
- Dashboard events: **10% sample** in production at scale (reduces noise; not safety-critical)
- Survey events: **100% capture** — low volume, high signal

---

## Privacy Controls

- `tech_id` is an internal UUID — never exposed in external dashboards without explicit mapping
- `vin` is included in server-side events only — stripped before events are forwarded to Amplitude (product analytics); present in S3 warehouse only (for return-visit FTFR matching)
- Segment events are routed to regional instances (us, ca, uk) matching the dealer's data residency requirement — no cross-region forwarding of events containing `vin` or `tech_id`
- Voice transcripts (`voice_command_received.raw_transcript`): stored in S3 for model improvement; retained 90 days; never forwarded to Amplitude

---

## Alerting on Key Events

Real-time alerts (PagerDuty, on-call rotation):

| Condition | Alert Level | Recipient |
|---|---|---|
| `hv_stop_overridden` fired | P0 (immediate) | VP Engineering + Compliance Manager |
| `hv_stop_triggered.activation_latency_ms` > 80ms | P1 (< 1 hour) | Engineering on-call |
| Voice recognition rate < 93% (rolling 1 hour) | P2 (< 4 hours) | ML on-call |
| Session completion rate < 50% (rolling 24 hours, single dealer) | P3 (next business day) | CS team |
| Dashboard not loading for > 5 minutes | P2 (< 4 hours) | Engineering on-call |

---

## Instrumentation Rollout

| Phase | Events Instrumented |
|---|---|
| Phase 2 (internal) | All session events, all safety events, basic voice events |
| Phase 3 (beta) | Add survey events, diagnostic events, predictive alert events |
| Phase 4 (GA) | Add dashboard events; full sampling policy active |
| Phase 5 | Add collaborative session events, recording events, international routing |
