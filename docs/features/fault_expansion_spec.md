# Fault Expansion Spec — 4 Additional Fault Scenarios
## Step 23 — Phase 3, Expand 1

**Roadmap reference:** Phase 3, Expand 1
**Status:** `[ ]` Not started
**Owner:** PM + ML Engineering
**Target:** All 4 faults diagnosed end-to-end in test bay by Month 11

---

## Context

Phase 2 proved the system on a single fault: EV battery coolant leak (bZ4X). Phase 3 expands diagnostic coverage to 4 additional fault types, increasing DiagnosticCore's RAG corpus and adding new CAD layer definitions to CADRenderer. Each new fault must meet the same exit bar as the Phase 2 coolant flow: full end-to-end demo, latency validated, HV-STOP coverage updated where applicable.

---

## Fault Definitions

### Fault 1: EV Battery Cell Fault (Toyota bZ4X)

**DTC targets:** P0A80 (Replace Battery Pack), P0A81 (Battery Module Imbalance), P0A7E (Battery Pack SOH Low)

**Clinical scenario:** One or more prismatic cells in Module 3 are capacity-degraded (< 70% SOH). Pack exhibits voltage imbalance > 100mV between cell groups during charge cycle.

**Diagnostic challenge:** Cell-level faults require interpreting cell voltage spread across 96 prismatic cells. The CAD layer must highlight the specific module and cell group flagged by BMS data — not just the pack.

**New CAD layers required:**
- `bms_cell_groups` — individual cell block highlighting within module (LOD 3)
- `cell_voltage_heatmap` — color-coded overlay mapped to live BMS data (green/yellow/red)

**New RAG corpus additions:**
- Toyota bZ4X Battery Service Manual Section 12: Cell Replacement Procedure
- Toyota TSB-0123: Battery Pack SOH Assessment Protocol
- SAE J2980: EV Battery Module Safety

**HV-STOP relevance:** High — cell replacement requires HV isolation. HV-STOP zone expands to include all 4 sides of pack.

---

### Fault 2: ICE Timing Chain Fault (Ford F-150 3.5L EcoBoost)

**DTC targets:** P0016 (Crankshaft/Camshaft Correlation — Bank 1, Sensor A), P0017, P0018, P0019

**Clinical scenario:** Timing chain stretch on high-mileage unit. Variable cam timing (VCT) solenoid sluggish. Technician must visually confirm chain stretch measurement and replace tensioner and chain.

**Diagnostic challenge:** ICE fault, not EV. No HV risk. CAD layer must show timing system with crankshaft, camshaft sprockets, chain path, and VCT actuator. Layer complexity is high (many moving parts).

**New CAD layers required:**
- `timing_chain_assembly` — full chain path, sprockets, guides, tensioner
- `vct_solenoid` — highlight solenoid body + oil port
- `crankshaft_position_sensor` — sensor location on block

**New RAG corpus additions:**
- Ford F-150 3.5L EcoBoost Powertrain Service Manual Ch. 7
- Ford TSB 22-2150: Timing Chain Rattle Diagnosis
- Ford Workshop Manual: VCT System Description

**HV-STOP relevance:** None — ICE vehicle, no HV bus. SafetyGuard operates in reduced mode (no HV proximity check).

---

### Fault 3: Hybrid Inverter Fault (Toyota RAV4 Hybrid)

**DTC targets:** P0A94 (DC/DC Converter Performance), P3000 (Battery Control System), P0A78 (Motor Inverter Performance)

**Clinical scenario:** Inverter module on motor generator 2 (MG2) showing thermal runaway precursor. Coolant flow through inverter reduced. Inverter output voltage oscillating ± 8V at 10Hz.

**Diagnostic challenge:** Hybrid inverter is both high-voltage and thermally sensitive. CAD layer must overlay inverter location within the engine bay, distinct from the HV battery pack. SensorFusion must integrate CAN bus signals from the hybrid ECU in addition to standard OBD-II.

**New CAD layers required:**
- `hybrid_inverter_module` — inverter position in engine bay
- `mg2_inverter_thermal_zone` — thermal camera overlay mapped to inverter body
- `hv_cable_routing_hybrid` — orange HV cable routing from pack to inverter to motor

**New RAG corpus additions:**
- Toyota RAV4 Hybrid Service Manual: Hybrid System Section
- Toyota TSB-HV-2201: Inverter Coolant Flow Check
- OSHA 1910.147: Control of Hazardous Energy (lockout/tagout reference)

**HV-STOP relevance:** High — inverter is energized to 650V DC during drive-ready state. HV-STOP zone covers full inverter access area.

---

### Fault 4: Brake System Leak (Multi-vehicle — Toyota and Ford)

**DTC targets:** C0031 (Left Front Wheel Speed Sensor), C1201 (ABS Control System), no DTC in early leak stage

**Clinical scenario:** Hydraulic brake fluid leak at left front caliper banjo bolt. Tech notices spongy pedal; brake fluid reservoir at minimum. No DTC may be present if leak is slow. Thermal camera shows fluid drip pattern on rotor.

**Diagnostic challenge:** Early-stage brake leaks may not throw DTCs. Diagnosis is primarily visual + thermal. DiagnosticCore must handle low-DTC or zero-DTC inputs and reason from sensor anomalies. CAD layer must show brake circuit hydraulic path.

**New CAD layers required:**
- `brake_hydraulic_circuit` — master cylinder, hard lines, flex hoses, calipers
- `left_front_caliper` — highlight caliper body, banjo bolt, bleed screw
- `thermal_drip_overlay` — thermal camera highlight on fluid drip location

**New RAG corpus additions:**
- Toyota/Ford Brake System Service Manual sections
- FMVSS 135: Brake System Requirements
- SAE J2050: Brake Fluid Performance Specification

**HV-STOP relevance:** None for ICE/brake fault. Low for EV (12V system only in brake circuit).

---

## DiagnosticCore Expansion Requirements

| Fault | New DTC Families | New RAG Corpus Size | New Intent Handlers |
|---|---|---|---|
| EV cell fault | P0A80, P0A81, P0A7E | ~40 pages OEM docs | `show_cell_detail`, `highlight_imbalanced_cell` |
| ICE timing chain | P0016–P0019 | ~60 pages OEM docs | `show_timing_system`, `highlight_tensioner` |
| Hybrid inverter | P0A94, P3000, P0A78 | ~50 pages OEM docs | `show_inverter`, `thermal_overlay_on` |
| Brake leak | C0031, C1201, zero-DTC | ~30 pages OEM docs | `show_brake_circuit`, `highlight_left_front` |

---

## Exit Criteria (per fault)

- [ ] Full end-to-end demo in test bay: voice → CAD overlay → guided repair steps
- [ ] Latency p95 < 40ms perceived (warm start)
- [ ] HV-STOP coverage validated for applicable faults
- [ ] All repair steps OEM-sourced; zero LLM-generated instructions
- [ ] DiagnosticCore confidence score ≥ 0.75 for each fault on 10 test runs
