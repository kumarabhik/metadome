# HV-STOP Safety Audit Spec
## Step 22 — QA: 50-Scenario Proximity Test Protocol

**Roadmap reference:** Phase 2, QA 2
**Status:** `[ ]` Not started
**Owner:** Safety Engineering + QA Lead
**Target:** Complete by Month 7; results required for Phase 2 exit gate

---

## Objective

Validate that SafetyGuard v1 triggers the HV-STOP protocol correctly across 50 simulated proximity scenarios covering nominal conditions, edge cases, and adversarial inputs. HV-STOP must activate in 100% of true-positive scenarios with < 80ms activation latency. Zero false negatives are acceptable.

**This is a safety-critical gate. Phase 3 deployment to real dealerships cannot begin until 100% pass rate is achieved.**

---

## Background

The HV-STOP protocol (specified in `docs/agents/safety_guard_v1.md`) activates when SafetyGuard detects a technician's hand or body entering a zone within 300mm of a high-voltage component while the HV bus is energized (> 60V DC). Dual-sensor confirmation is required: capacitive proximity sensor AND thermal imaging must both confirm proximity before activation. False positives are acceptable; false negatives are not.

---

## Test Environment

| Parameter | Value |
|---|---|
| Vehicle | 2024 Toyota bZ4X (test bay unit — HV bus active via bench supply) |
| HV bus voltage | 397V DC (nominal) — bench supply, isolated from grid |
| Proximity sensors | 4× capacitive sensors mounted per `docs/agents/safety_guard_v1.md` placement spec |
| Thermal camera | FLIR Lepton 3.5 (mounted per bay spec) |
| Test dummy | Anthropomorphic arm with embedded heat source (36.5°C ± 0.5°C to simulate hand) |
| Timer | Hardware-latched timer triggered by proximity event and HV-STOP activation signal |
| Data logger | SafetyGuard audit log + external video capture at 240fps |

---

## 50-Scenario Test Matrix

### Category 1: True Positive — Must Trigger (25 scenarios)

| # | Description | Approach Speed | Angle | Bus State | Expected |
|---|---|---|---|---|---|
| 1 | Hand enters 300mm zone, direct vertical approach | 100mm/s | 90° | Energized | TRIGGER |
| 2 | Hand enters 300mm zone, lateral approach | 100mm/s | 0° | Energized | TRIGGER |
| 3 | Hand enters 300mm zone, slow approach | 10mm/s | 45° | Energized | TRIGGER |
| 4 | Hand enters 300mm zone, fast approach | 500mm/s | 90° | Energized | TRIGGER |
| 5 | Forearm (not hand) enters 300mm zone | 100mm/s | 90° | Energized | TRIGGER |
| 6 | Hand at 250mm (inside zone) | Static | — | Energized | TRIGGER |
| 7 | Hand at 200mm (inside zone) | Static | — | Energized | TRIGGER |
| 8 | Hand crosses zone boundary from below | 100mm/s | 270° | Energized | TRIGGER |
| 9 | Two hands simultaneously entering zone | 100mm/s each | 90°/0° | Energized | TRIGGER |
| 10 | Hand enters zone while tech is walking (full body motion) | 100mm/s | 90° | Energized | TRIGGER |
| 11 | Hand enters zone in low lighting (< 50 lux) | 100mm/s | 90° | Energized | TRIGGER |
| 12 | Hand enters zone in high ambient IR noise (heat gun nearby) | 100mm/s | 90° | Energized | TRIGGER |
| 13 | Hand enters zone while second person present in bay | 100mm/s | 90° | Energized | TRIGGER |
| 14 | Hand enters zone while headset is rendering heavy frame | 100mm/s | 90° | Energized | TRIGGER |
| 15 | Hand enters zone during voice command processing | 100mm/s | 90° | Energized | TRIGGER |
| 16 | Hand enters zone during LLM response streaming | 100mm/s | 90° | Energized | TRIGGER |
| 17 | Capacitive sensor only triggered (thermal inconclusive) | 100mm/s | 90° | Energized | TRIGGER (conservative) |
| 18 | Thermal only triggered (capacitive inconclusive) | 100mm/s | 90° | Energized | TRIGGER (conservative) |
| 19 | Hand enters zone — bus at 100V (above 60V threshold) | 100mm/s | 90° | Energized low | TRIGGER |
| 20 | Hand enters zone — bus at 61V (just above threshold) | 100mm/s | 90° | Energized low | TRIGGER |
| 21 | Hand enters zone while edge server CPU > 90% load | 100mm/s | 90° | Energized | TRIGGER |
| 22 | Hand enters zone during network packet loss event | 100mm/s | 90° | Energized | TRIGGER |
| 23 | Tool (wrench) — metallic — enters zone | 100mm/s | 90° | Energized | TRIGGER (metal detection) |
| 24 | Hand enters zone — rear battery pack zone (not front) | 100mm/s | 90° | Energized | TRIGGER |
| 25 | Hand enters zone — motor inverter zone | 100mm/s | 90° | Energized | TRIGGER |

### Category 2: True Negative — Must Not Trigger (15 scenarios)

| # | Description | Bus State | Expected |
|---|---|---|---|
| 26 | Hand at 350mm (outside zone) — bus energized | Energized | NO TRIGGER |
| 27 | Hand at 400mm static | Energized | NO TRIGGER |
| 28 | Technician standing 500mm from battery | Energized | NO TRIGGER |
| 29 | Hand inside zone — bus de-energized (< 60V) | De-energized | NO TRIGGER |
| 30 | Hand inside zone — bus fully discharged (0V) | Off | NO TRIGGER |
| 31 | Hand inside zone — HV-STOP already acknowledged, bus isolated | Isolated | NO TRIGGER |
| 32 | Hand waving in air 600mm from vehicle | Energized | NO TRIGGER |
| 33 | Thermal blooming from exhaust vent (no hand) | Energized | NO TRIGGER |
| 34 | Cold metal tool (< 20°C) near sensor (no human) | Energized | NO TRIGGER |
| 35 | Capacitive noise from shop radio interference | Energized | NO TRIGGER |
| 36 | Second technician's body at 600mm from different side | Energized | NO TRIGGER |
| 37 | ArUco marker arm in zone (non-biological) | Energized | NO TRIGGER |
| 38 | Reflective vest material near sensor | Energized | NO TRIGGER |
| 39 | Hand at exactly 301mm (1mm outside zone boundary) | Energized | NO TRIGGER |
| 40 | Air hose movement near sensor (pneumatic shop tool) | Energized | NO TRIGGER |

### Category 3: Edge Cases & Recovery (10 scenarios)

| # | Description | Expected |
|---|---|---|
| 41 | HV-STOP triggered; tech acknowledges; immediately re-enters zone | RE-TRIGGER within 5s |
| 42 | HV-STOP triggered; tech does not acknowledge for 30s | Audio reminder at 15s; escalation alert at 30s |
| 43 | HV-STOP triggered; headset battery critically low (< 5%) | TRIGGER; audio only (display may dim) |
| 44 | SafetyGuard agent crashes mid-session | Failsafe: audio alarm activates; CAD overlays freeze |
| 45 | Proximity sensor 1 of 4 offline | Remaining 3 sufficient; system warns operator; continues |
| 46 | Thermal camera disconnected | Capacitive alone triggers; warning logged |
| 47 | HV bus state unknown (OBD-II disconnected) | Assume energized; full HV-STOP protocol active |
| 48 | Two simultaneous HV-STOP triggers from different zones | Both logged; single consolidated audio alert |
| 49 | Tech hand in zone — bus transitions from de-energized to energized | TRIGGER within 80ms of bus energize event |
| 50 | Full system reboot during HV-STOP active state | After reboot: check zone occupancy before clearing alert |

---

## Audit Log Format

Each scenario generates a structured log entry:

```json
{
  "scenario_id": 1,
  "timestamp_utc": "2025-09-12T14:23:01.452Z",
  "session_id": "qa-audit-run-003",
  "bus_voltage_v": 397.2,
  "proximity_mm": 248,
  "capacitive_triggered": true,
  "thermal_triggered": true,
  "hv_stop_activated": true,
  "activation_latency_ms": 62,
  "pass": true,
  "notes": ""
}
```

All 50 entries written to `hv_stop_audit_run_<date>.jsonl`. Retained for 7 years per OSHA 1910.147 recordkeeping requirements.

---

## Pass/Fail Criteria

| Metric | Requirement | Disposition on Fail |
|---|---|---|
| True positive trigger rate | 100% (25/25 scenarios 1–25) | BLOCK Phase 3 |
| True negative non-trigger rate | 100% (15/15 scenarios 26–40) | BLOCK Phase 3 |
| Edge case handling | All 10 behave per expected column | BLOCK if safety-critical; fix-and-retest for others |
| Activation latency (all triggers) | p95 < 80ms | BLOCK Phase 3 |
| Activation latency (max observed) | < 120ms | Investigate root cause |

---

## Deliverables

- `docs/qa/hv_stop_audit_results_<date>.jsonl` — raw per-scenario log
- `docs/qa/hv_stop_audit_summary.md` — aggregate results, latency histogram, pass/fail disposition
- Safety sign-off memo from Safety Engineering Lead — required before Phase 3 kickoff
