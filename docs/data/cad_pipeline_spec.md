# CAD Ingestion Pipeline — Technical Specification
## Step 5 — OEM CAD → Normalized 3D Mesh → VIN-Indexed Cache

**Roadmap reference:** Phase 1, Deliverable 5
**Status:** `[x]` Spec complete
**Owner:** Engineering Lead (Data Platform)
**Target:** Pipeline operational by Month 3

---

## Overview

The CAD ingestion pipeline transforms raw OEM-provided CAD files into runtime-ready 3D geometry cached at the edge, indexed by VIN. It runs as an offline batch process — never on the critical repair path. All geometry must be pre-processed and cached before any technician uses the system.

```
OEM CAD Source  →  Ingestion  →  Normalization  →  Layer Packaging  →  VIN Cache  →  Edge Deploy
(STEP / JT)       (Parser)       (gLTF 2.0)        (layer bundles)    (PostgreSQL)  (Jetson AGX)
```

---

## Input: OEM CAD Formats

| OEM | Native Format | Conversion Tool | Notes |
|---|---|---|---|
| Toyota | JT (Jupiter Tessellation) | Siemens Parasolid SDK | JT is standard Toyota internal format; requires Siemens SDK license (~$8K/year) |
| Ford | STEP AP242 | FreeCAD (open source) | Ford uses STEP for supplier exchange; good tool coverage |
| Stellantis | CATIA V5 (.CATPart) | Datakit CrossManager | CATIA V5 is legacy but stable; CrossManager handles conversion reliably |
| Fallback | OBJ / FBX | Three.js loader | For photogrammetry-derived geometry when OEM data unavailable |

---

## Pipeline Stages

### Stage 1: Ingestion

**Input:** Raw OEM CAD file (JT / STEP / CATIA)
**Process:**
- Validate file integrity (checksum against OEM-provided hash)
- Parse file using format-specific SDK
- Extract: geometry meshes, assembly hierarchy, part metadata (part number, description, material)
- Extract: subsystem classification tags (OEM-provided or inferred from part number taxonomy)
- Flag any geometry that fails validation (missing normals, non-manifold surfaces, >500K polygon count)

**Output:** Structured geometry object tree with metadata

**Rejection criteria:**
- File checksum mismatch → quarantine, notify OEM partnership team
- Non-manifold geometry on safety-critical parts (HV bus, coolant lines) → reject, request resubmit
- Missing part-to-subsystem classification for >5% of parts → flag for manual review

---

### Stage 2: Normalization

**Input:** Structured geometry object tree
**Process:**
- Convert all geometry to **gLTF 2.0** (GL Transmission Format) — the target runtime format for WebXR and Unity XR
- Apply coordinate system normalization:
  - Origin → front axle centerline at ground plane
  - X axis → vehicle longitudinal (forward)
  - Y axis → vehicle lateral (left)
  - Z axis → vertical (up)
  - All measurements in millimeters
- Apply LOD (Level of Detail) baking:
  - LOD-0: Full detail (render at < 2m distance) — max 150K triangles per layer
  - LOD-1: Medium detail (2–5m) — max 40K triangles per layer
  - LOD-2: Low detail (> 5m) — max 8K triangles per layer
- Apply material normalization: all materials converted to PBR (Physically Based Rendering) metallic-roughness workflow
- Apply UV unwrapping for texture coordinates (required for color-coding overlay effects)

**Output:** Normalized gLTF 2.0 files, 3 LOD levels per geometry object

---

### Stage 3: Layer Packaging

**Purpose:** Group geometry objects into semantically meaningful layers that the CADRenderer agent can activate/suppress as a unit.

**Layer taxonomy (v1, Toyota bZ4X):**

| Layer ID | Contents | Activation Trigger |
|---|---|---|
| `tms_coolant_loop` | Coolant hoses, pump, reservoir, inlet/outlet couplings | DTC: P0A93, P1DF1, P0A94; or voice: "show coolant" |
| `bms_modules` | Individual battery modules M1–M12 with position data | DTC: P0A80, P0A81; or voice: "show battery" |
| `hv_bus` | High-voltage bus bars, connectors, orange cable routing | Always-on ambient layer; SafetyGuard control |
| `bms_thermal_sensors` | Temperature sensor positions (labeled) | When thermal anomaly detected |
| `coolant_pump_assembly` | Pump body, motor, impeller (explodable) | DTC: P1DF1 specifically |
| `hvac_chiller` | Battery chiller unit, refrigerant lines | DTC: P0A9C; or voice: "show chiller" |
| `structural_floor` | Floor pan geometry (reference only, low opacity) | Always-on reference layer, 20% opacity |
| `exterior_body` | Body panels (ghost mode, 10% opacity) | Always-on during full-overlay mode |

**Layer bundle format:**
```json
{
  "layer_id": "tms_coolant_loop",
  "version": "1.2.0",
  "vehicle_models": ["2022-bZ4X", "2023-bZ4X", "2024-bZ4X"],
  "geometry_lod": {
    "lod0": "s3://metadome-cad/toyota/bZ4X/v1.2/tms_coolant_loop_lod0.glb",
    "lod1": "s3://metadome-cad/toyota/bZ4X/v1.2/tms_coolant_loop_lod1.glb",
    "lod2": "s3://metadome-cad/toyota/bZ4X/v1.2/tms_coolant_loop_lod2.glb"
  },
  "default_color": "#2196F3",
  "highlight_color": "#F44336",
  "fault_color": "#FF9800",
  "anchor_points": [
    {"part_id": "coolant_inlet_M4", "position": {"x": 847, "y": 214, "z": 320}, "label": "Module 4 Inlet"}
  ],
  "dtc_triggers": ["P0A93", "P1DF1", "P0A94"],
  "voice_triggers": ["coolant", "cooling system", "coolant leak", "coolant pump"]
}
```

---

### Stage 4: VIN-Indexed Cache

**Purpose:** Given a VIN, return the correct layer bundle set for that exact vehicle (trim, year, options affect geometry).

**VIN-to-model mapping:**
- Pull from OEM VIN decoder API (Toyota: TSM API; Ford: Ford VIN Decoder API)
- Cache VIN → {model, year, trim, powertrain, build options} in PostgreSQL
- Map build options to CAD variants (e.g., bZ4X AWD has different cooling circuit routing than FWD)

**Cache structure (per edge server):**

```
/cad-cache/
  toyota/
    bz4x/
      2024/
        awd/
          tms_coolant_loop_lod0.glb    (142 MB)
          tms_coolant_loop_lod1.glb    (38 MB)
          tms_coolant_loop_lod2.glb    (8 MB)
          bms_modules_lod0.glb         (210 MB)
          hv_bus_lod0.glb              (95 MB)
          ... (all layers)
      2023/
        ...
  ford/
    mach-e/
      ...
```

**Estimated storage per vehicle model/year/trim:** 1.2–2.4 GB (all layers, all LODs)
**Phase 1 edge server storage required:** 3 models × 3 years × 2 trims × 2.0 GB avg = ~36 GB (well within 2TB SSD)

---

## Performance Targets

| Metric | Target | Measurement |
|---|---|---|
| Full pipeline run time per vehicle model | < 4 hours | Automated CI/CD pipeline timer |
| VIN-to-cache-hit lookup | < 50ms | Edge server query log |
| Layer bundle load to headset | < 200ms (LOD-1) | CADRenderer agent log |
| Pipeline freshness: OEM CAD update → edge refresh | < 72 hours | Scheduled pipeline trigger on OEM push |

---

## Data Security

- OEM CAD geometry is encrypted at rest (AES-256) on edge server
- CAD files are never transmitted to end-user devices — only rendered geometry streams (no raw file export possible from headset)
- VIN entitlement check on every layer load: if VIN is not in the active bay's repair order, CAD is not served
- All CAD access logged for OEM audit (required per licensing agreement)
