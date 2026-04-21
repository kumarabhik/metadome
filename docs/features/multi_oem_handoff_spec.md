# Multi-OEM Handoff Spec — Toyota ↔ Ford Context Switch in Single Session
## Step 42 — Phase 4, Feature 7

**Roadmap reference:** Phase 4, Feature 7
**Status:** `[ ]` Not started
**Owner:** Engineering (all 5 agents) + Product
**Target:** Multi-OEM handoff in production by Month 17

---

## Objective

Allow a technician to switch between vehicle makes (e.g., Toyota bZ4X → Ford Mustang Mach-E) within the same work shift without restarting the headset or manually resetting the system. In a 50-bay GA dealership deployment, multi-line dealers service multiple OEM brands daily. Without handoff support, a technician finishing a Toyota repair must power-cycle the headset before starting a Ford fault — a friction point that degrades adoption and breaks the seamless workflow the product promises.

**Success criteria:** Full context switch (from old-vehicle session close to new-vehicle session ready state) completes in ≤ 5 seconds with no residual CAD layers from the prior vehicle visible in the new session.

---

## Trigger Conditions

A Multi-OEM handoff is triggered by one of two events:

| Trigger | Mechanism | Initiator |
|---|---|---|
| VIN scan on new vehicle | Tech scans ArUco marker or OBD-II VIN read on different vehicle | Automatic (system-initiated) |
| Voice command | "Switch vehicle" / "New car" / "Different vehicle" | Tech-initiated |
| Manual session end + new session start | "Close diagnostic" → new bay entry → new VIN detected | Semi-automatic |

**Priority:** VIN scan is the most reliable trigger. Voice command is secondary and requires confirmation. Manual session end is the fallback.

---

## Handoff Flow — Step by Step

### Phase A: Old Session Close (≤ 2 seconds)

1. **Trigger detected** — VIN mismatch between current session VIN and newly scanned/detected VIN
2. **DiagnosticCore** — saves session state to edge server (repair step progress, open DTCs, confidence score snapshot) for audit log
3. **CADRenderer** — executes `ClearAllLayers()`: removes all active holographic layers from headset render pipeline; renders blank state
4. **SensorFusion** — disconnects OBD-II / CAN bus stream from prior vehicle; closes thermal camera session
5. **SafetyGuard** — clears prior vehicle's HV bus location map; resets proximity alert zones; enters `SAFE_IDLE` state
6. **VoiceNLP** — clears prior vehicle OEM vocabulary context; enters neutral state

**Unfinished repair warning:** If prior session has an incomplete repair (open DTC, mid-sequence step), DiagnosticCore surfaces a confirmation prompt before handoff:
> "Repair on Toyota bZ4X (VIN: JTM…) is incomplete — Battery coolant fault open. Save and switch?"
> Options: [Save & Switch] [Cancel]

Saved state is accessible in the manager dashboard for that VIN. Tech can return to it in a future session.

### Phase B: New Vehicle Initialization (≤ 3 seconds)

1. **SensorFusion** — connects to new vehicle OBD-II / CAN bus; pulls VIN confirmation; requests DTC list
2. **DiagnosticCore** — loads new vehicle context: VIN → OEM → model → CAD version → prior repair history from DMS
3. **CADRenderer** — pre-caches top 3 most probable layer combinations for new vehicle's open DTCs (same pre-cache logic as bay entry)
4. **VoiceNLP** — hot-swaps entity extractor vocabulary to new OEM (e.g., Ford Mach-E vocabulary replaces Toyota bZ4X vocabulary)
5. **SafetyGuard** — loads new vehicle's HV bus location map from CAD; reconfigures proximity alert zones; exits `SAFE_IDLE`
6. **ArUco anchor** — tech is prompted to look at ArUco marker on new vehicle; SLAM re-anchors in ≤ 500ms once marker detected
7. **Ready state** — DiagnosticCore renders new vehicle's fault summary in HUD: "Ford Mach-E — 2 active DTCs: P0A1F, U3003"

---

## Agent Contract Changes

### DiagnosticCore
- New method: `SaveSessionState(vin, repair_step, open_dtcs, confidence_score)` — writes to edge server audit log
- New method: `LoadVehicleContext(vin)` — replaces current vehicle context; triggers downstream hot-swap events
- Session isolation: context window is fully replaced on handoff; no cross-vehicle reasoning contamination

### CADRenderer
- New method: `ClearAllLayers()` — unconditional layer purge; completes in ≤ 200ms
- Layer budget enforcement: new vehicle starts at 0 active layers; budget resets fresh
- SLAM anchor: on new vehicle, renders "Scan ArUco marker to anchor" prompt until anchor confirmed

### SensorFusion
- OBD-II connection: graceful disconnect from prior vehicle socket; reconnect to new vehicle socket
- `UnifiedSensorState` includes `session_id` and `vin` tags — all post-handoff readings tagged to new VIN
- If new vehicle OBD-II is not yet connected at handoff: SensorFusion enters `WAITING_FOR_VEHICLE` state; DiagnosticCore displays "Connect OBD-II to begin"

### VoiceNLP
- Vocabulary hot-swap: entity extractor model re-initialized with new OEM vocabulary without restarting the Whisper ASR pipeline (ASR is OEM-agnostic; only NER layer is OEM-specific)
- Hot-swap latency target: ≤ 500ms
- Wake word: unchanged — "Hey Aria" / "Oye Aria" is OEM-neutral

### SafetyGuard
- HV bus location map is per-vehicle-model; must be replaced on every handoff
- SafetyGuard enters `SAFE_IDLE` during Phase A (old session close) — no proximity alerts possible until new HV map loaded
- `SAFE_IDLE` → `ACTIVE` transition: only after new HV bus map confirmed loaded and SensorFusion CAN bus stream confirmed for new vehicle
- Hard rule: SafetyGuard cannot enter `ACTIVE` state on new vehicle until HV bus map is confirmed for that specific VIN/model combination. If CAD data for new VIN is not available: SafetyGuard stays in `SAFE_IDLE`, renders "CAD data unavailable — operating without HV proximity alerts" warning in headset.

---

## Same-OEM Handoff (Same Brand, Different Vehicle)

Multi-OEM handoff spec covers all vehicle switches, including same-OEM (e.g., Toyota bZ4X → Toyota RAV4 Hybrid). The protocol is identical but Phase B is faster because:
- OEM CAD database doesn't change (Toyota → Toyota)
- VoiceNLP vocabulary base stays the same (only per-model entity updates)
- SensorFusion OBD-II reconnect is the same procedure

Expected same-OEM handoff time: ≤ 3 seconds total (vs. ≤ 5 seconds for cross-OEM).

---

## Edge Cases

| Scenario | Behavior |
|---|---|
| New VIN not in CAD database | CAD layers suppressed; DiagnosticCore falls back to tablet PDF link; SafetyGuard in `SAFE_IDLE`; system alerts tech with clear message |
| OBD-II not responding on new vehicle | SensorFusion enters `WAITING_FOR_VEHICLE`; DiagnosticCore in reduced-confidence mode using thermal camera only |
| Tech scans wrong ArUco marker (wrong bay) | SLAM anchor coordinates mismatch with expected vehicle footprint → system warns "Vehicle position mismatch — confirm correct vehicle" before rendering layers |
| Power loss during handoff | On restart, system enters fresh session (no partial-state recovery); audit log preserved from prior session |
| Two techs swap headsets mid-repair | Session tied to headset device ID + VIN. Swapping headset to same vehicle is allowed. Swapping to different vehicle triggers normal handoff flow. |

---

## Latency Budget

| Phase | Step | Budget |
|---|---|---|
| A | ClearAllLayers() | ≤ 200ms |
| A | SensorFusion disconnect | ≤ 500ms |
| A | SafetyGuard HV map clear | ≤ 100ms |
| A | Session state save to disk | ≤ 500ms |
| **A total** | | **≤ 2,000ms** |
| B | OBD-II connect + VIN confirm | ≤ 500ms |
| B | DiagnosticCore vehicle context load | ≤ 500ms |
| B | VoiceNLP vocabulary hot-swap | ≤ 500ms |
| B | SafetyGuard HV map load | ≤ 500ms |
| B | CADRenderer pre-cache | ≤ 1,000ms (async — doesn't block ready state) |
| **B total** | | **≤ 3,000ms** |
| **Total handoff** | | **≤ 5,000ms** |

---

## Testing Protocol

### Unit Tests
- `ClearAllLayers()` with 4 active layers → confirms 0 layers in ≤ 200ms
- `SaveSessionState()` → confirms all fields written to audit log
- VoiceNLP vocabulary hot-swap → Toyota entity "coolant system" no longer resolves after Ford swap; Ford entity "GigE battery" resolves

### Integration Tests
- Full handoff flow: Toyota bZ4X session (mid-repair) → Ford Mach-E session
- Full handoff flow: same-OEM (Toyota bZ4X → Toyota RAV4 Hybrid)
- Unfinished repair warning → [Cancel] path (stays in old session) and [Save & Switch] path

### Performance Tests
- Handoff latency measured end-to-end across 20 repetitions: mean and p95 must meet 5-second budget
- SafetyGuard `SAFE_IDLE` → `ACTIVE` transition: confirm HV map loaded correctly for both vehicles

### Regression Test
- After handoff, confirm zero Toyota CAD layer residue visible in Ford session (render pipeline screenshot comparison)
- Confirm DiagnosticCore context window contains only Ford vehicle data after handoff

---

## Open Questions

- **Multi-bay handoff (tech walks to different bay):** If tech carries headset from Bay 1 (Toyota) to Bay 2 (Ford), does SLAM re-anchor automatically? Current plan: ArUco marker scan is required for anchor in new bay. SLAM does not auto-anchor across physical bay boundaries.
- **Repair history cross-reference:** If the same VIN appears in multiple sessions, should DiagnosticCore surface prior repair history at handoff? Yes — prior open DTCs on same VIN should be surfaced. This is already in DiagnosticCore's vehicle context load logic (DMS prior repair history). No new work required.
