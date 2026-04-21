# Ford Mustang Mach-E Integration Spec
## Step 24 — Phase 3, Expand 2: Second OEM Vehicle Line

**Roadmap reference:** Phase 3, Expand 2
**Status:** `[ ]` Not started
**Owner:** Engineering Lead + BD (Ford Partnership)
**Target:** Mach-E end-to-end demo complete by Month 12

---

## Objective

Expand X-Ray Vision Diagnostics from Toyota-only (bZ4X) to a second OEM vehicle: the Ford Mustang Mach-E. This is the first cross-OEM validation and proves the platform's architecture is vehicle-agnostic. It requires new CAD data ingestion, a new VoiceNLP vocabulary, and validation of DiagnosticCore's OEM-document-switching logic.

---

## Vehicle Scope for Phase 3

| Parameter | Value |
|---|---|
| Model | 2022–2024 Ford Mustang Mach-E (Extended Range AWD) |
| Powertrain | Dual-motor BEV; 88kWh usable; 400V nominal HV bus |
| Target fault types | Hybrid inverter fault (P0A94-ford), brake system leak (C0031-ford) |
| OEM data partner | Ford Motor Company (licensing via `docs/legal/oem_partnership_framework.md`) |
| VIN sample for test | 3FMTK3SU9NMA12345 |

---

## CAD Data Ingestion

Ford delivers CAD in STEP AP214 format (same standard as Toyota). Pipeline is defined in `docs/data/cad_pipeline_spec.md`. Mach-E–specific steps:

### New VIN Indexing
- VIN decoder: Ford WMI = `3FM`; model code `TK3SU` = Mach-E Extended AWD
- CAD cache key format: `FORD_MACHE_EXTAWD_2022-2024`
- Pre-cache at edge server startup; cache TTL = 72 hours

### New Layer Definitions (Mach-E)

| Layer Name | Description | LOD Range |
|---|---|---|
| `mache_body_ghost` | Full body wireframe ghost | LOD 1 |
| `mache_hv_bus` | 400V HV cable routing (orange) | LOD 1 |
| `mache_front_motor` | Front motor unit | LOD 2 |
| `mache_rear_motor` | Rear motor unit | LOD 2 |
| `mache_inverter` | Inverter module (front trunk) | LOD 2 |
| `mache_hv_battery` | 88kWh pack (floor) | LOD 2 |
| `mache_bms_modules` | 10 battery modules with cell groups | LOD 3 |
| `mache_brake_circuit` | Full hydraulic brake circuit | LOD 2 |
| `mache_thermal_zone` | FLIR overlay anchor points | LOD 2 |

### Coordinate System
Ford CAD uses SAE vehicle coordinate system (X forward, Y left, Z up). Toyota uses ISO 8855. Normalization transform:
```
ford_to_platform: rotate 180° around Z-axis
offset: apply VIN-specific wheelbase offset
```

---

## VoiceNLP Vocabulary Extension

DiagnosticCore vocabulary is Toyota-specific in Phase 2. Mach-E requires a Ford-specific vocabulary layer.

### New Intent Vocabulary

| User Phrase | Intent | Entity |
|---|---|---|
| "Show me the Mach-E inverter" | `show_system` | `mache_inverter` |
| "Where is the high voltage battery" | `show_system` | `mache_hv_battery` |
| "Highlight the front motor" | `show_system` | `mache_front_motor` |
| "What's the brake issue" | `diagnose_fault` | `mache_brake_circuit` |
| "Ford service bulletin for P0A94" | `request_bulletin` | `ford_tsb_p0a94` |

### Model Disambiguation
When VoiceNLP receives a vehicle-specific command, it must route to the correct OEM namespace. VIN loaded at session start sets the active OEM context. Ford context enables Ford vocabulary; Toyota context enables Toyota vocabulary.

```
ActiveContext = {
  oem: "ford" | "toyota",
  model: "mache" | "bz4x" | ...,
  vin: str
}
```

---

## DiagnosticCore RAG Corpus — Ford

| Document | Source | Pages |
|---|---|---|
| Mach-E Service Manual: HV System | Ford Motor Company (licensed) | ~80 |
| Mach-E Service Manual: Brake System | Ford Motor Company (licensed) | ~40 |
| Ford TSB 22-1892: Inverter Thermal Check | Ford TSB Database | ~6 |
| Ford TSB 22-0421: Brake Caliper Inspection | Ford TSB Database | ~4 |
| FMVSS 135: Brake System Requirements | NHTSA | ~20 |
| SAE J2950: EV Service Technician Safety | SAE | ~30 |

Corpus ingested into Chroma DB under namespace `ford_mache`. DiagnosticCore routes RAG queries by OEM namespace based on ActiveContext.

---

## SafetyGuard HV-STOP — Mach-E Adaptation

| Parameter | Toyota bZ4X | Ford Mach-E |
|---|---|---|
| HV bus voltage | 397V | 400V |
| HV-STOP zone radius | 300mm from HV components | 300mm from HV components |
| HV components | Battery pack, inverter, motor terminals | Battery pack, front inverter, front/rear motor terminals |
| Discharge verification | OBD-II: HV bus voltage < 5V | Ford FDRS protocol: HV bus voltage < 5V |

---

## ArUco Marker Placement (Mach-E)

Phase 3 field install requires placing ArUco markers on the Mach-E at 4 anchor points:
- Front left wheel well (ID: 0x10)
- Rear left wheel well (ID: 0x11)
- Hood hinge (ID: 0x12)
- Rear hatch sill (ID: 0x13)

Marker placement documented in `docs/prototype/aruco_anchoring_spec.md` (extension required for Mach-E geometry).

---

## Exit Criteria

- [ ] Mach-E CAD data licensed and ingested into pipeline
- [ ] All Mach-E CAD layers defined and rendering at correct positions in test bay
- [ ] End-to-end demo: voice → Mach-E CAD overlay → guided repair steps (inverter fault scenario)
- [ ] DiagnosticCore routes correctly between Ford and Toyota contexts in same session
- [ ] HV-STOP protocol validated for Mach-E HV layout
- [ ] No Toyota-vocabulary leakage into Mach-E session (VoiceNLP isolation test)
