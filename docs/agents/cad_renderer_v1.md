# CADRenderer v1 — Implementation Specification
## Step 13 — Layer Budget, SLAM Anchoring, LOD Management

**Roadmap reference:** Phase 2, Agent Build 3
**Status:** `[x]` Spec complete
**Owner:** XR Engineering Lead
**Target:** Working v1 by Month 7

---

## Responsibility

CADRenderer is the sole writer to the headset render pipeline. It consumes `LayerSelectionManifest` from DiagnosticCore and translates it into Unity scene mutations — activating/deactivating GameObjects, switching materials, updating spatial anchor positions. SafetyGuard has an out-of-band override channel that bypasses all normal rendering logic.

**Key invariant:** CADRenderer never makes diagnostic decisions. It renders exactly what the manifest tells it to. If the manifest is wrong, the bug is in DiagnosticCore, not CADRenderer.

---

## v1 Scope

- Platform: HoloLens 2 (UWP, ARM64)
- Engine: Unity 2022.3 LTS + MRTK 3.0 + World Locking Tools 1.5
- Input: `LayerSelectionManifest` (gRPC stream from edge DiagnosticCore)
- Output: Holographic overlay rendered at 60fps
- Spatial anchoring: ArUco + Azure Spatial Anchors (from Phase 1 prototype)
- Layer budget: max 4 simultaneous active CAD layers

---

## Unity Scene Structure

```
[Scene Root]
 ├── [AnchorManager]                    ← MRTK World Locking Tools root
 │    └── VehicleAnchor                 ← pinned to ArUco-derived pose
 │         └── [VehicleCADRoot]         ← all CAD geometry parented here
 │              ├── layer_hv_bus        ← always-on ambient layer (SafetyGuard)
 │              ├── layer_body_ghost    ← always-on reference layer
 │              ├── layer_tms_coolant   ← toggled by manifest
 │              ├── layer_bms_modules   ← toggled by manifest
 │              ├── layer_coolant_pump  ← toggled by manifest
 │              └── layer_hvac_chiller  ← toggled by manifest
 ├── [GuidanceArrowPool]               ← pooled arrow GameObjects (max 5 active)
 ├── [HUDCanvas]                       ← world-space UI (confidence badge, step counter)
 ├── [SafetyOverlayCanvas]             ← full-screen HV-STOP overlay (highest z-order)
 └── [ManifestReceiver]               ← gRPC client; receives manifests from DiagnosticCore
```

---

## Layer Management

### Layer Budget Enforcement

```csharp
public class LayerBudgetManager : MonoBehaviour
{
    private const int MAX_ACTIVE_LAYERS = 4;
    // Priority order: Safety > ActiveFault > ActivePath > Reference
    private static readonly int[] PRIORITY_ORDER = {0, 1, 2, 3};

    public void ApplyManifest(LayerSelectionManifest manifest)
    {
        // Safety layers (hv_bus, body_ghost) are excluded from budget count
        var budgetedLayers = manifest.Layers
            .Where(l => !IsSafetyLayer(l.LayerId))
            .OrderByDescending(l => GetPriority(l.RenderState))
            .Take(MAX_ACTIVE_LAYERS)
            .ToList();

        // Deactivate all non-safety layers first
        foreach (var layer in AllBudgetedLayers)
            SetLayerActive(layer.LayerId, false);

        // Activate only the top-N by priority
        foreach (var directive in budgetedLayers)
            ApplyDirective(directive);

        // Always apply safety layers regardless of budget
        ApplySafetyLayers(manifest);
    }
}
```

### Layer State Machine

Each layer GameObject has a `LayerController` component with these states:

| State | Behavior | Material |
|---|---|---|
| `Inactive` | GameObject disabled | — |
| `ActiveFault` | Visible, pulsing animation | Emissive red, 1Hz sine pulse |
| `ActivePath` | Visible, flow animation | Emissive blue, directional flow shader |
| `ActiveReference` | Visible, static | Emissive white, 30% opacity |
| `Suppressed` | Explicitly hidden (different from Inactive — DiagnosticCore decided to hide) | — |
| `SafetyDimmed` | Visible but dimmed to 20% — HV-STOP active | Multiply blend, 0.2 alpha |

### Material Swapping (GPU-Efficient)

Pre-instantiate one material instance per (layer × state) combination at scene load. State transitions swap the material reference — no shader compile stall at runtime.

```csharp
// Pre-loaded at Start()
_materials[("tms_coolant", "ActivePath")] = Resources.Load<Material>("Mat_Coolant_ActivePath");
_materials[("tms_coolant", "ActiveFault")] = Resources.Load<Material>("Mat_Coolant_Fault");
// ... etc for all combinations

// On state change:
renderer.material = _materials[(layerId, newState)];  // O(1) dictionary lookup
```

---

## SLAM Anchoring (v1)

Anchoring pipeline (detailed spec in `docs/prototype/aruco_anchoring_spec.md`). CADRenderer's responsibility:

1. **Session start:** Call `AnchorManager.ResolveOrCreateAnchor(vehicleVin)` — returns `SpatialAnchor` from Azure Spatial Anchors if bay has been used before, else starts ArUco calibration flow
2. **Anchor resolved:** Parent `VehicleCADRoot` to the resolved anchor. All child geometry is now in vehicle-relative space.
3. **Drift correction:** World Locking Tools runs its drift correction loop at 30Hz. CADRenderer does not need to manage this — WLT handles it transparently.
4. **Anchor confidence:** Expose `AnchorConfidence` (float 0–1) from WLT in HUD. If < 0.7, show amber "Anchor Recovering" badge. If < 0.4, show red "Tracking Lost — Look at vehicle markers" prompt.

---

## LOD Management

**LOD switching is driven by camera-to-object distance, evaluated per layer at 10Hz:**

```csharp
void UpdateLOD(LayerController layer)
{
    float distance = Vector3.Distance(Camera.main.transform.position,
                                       layer.BoundingSphere.center);
    string targetLod = distance < 2.0f ? "lod0"
                     : distance < 5.0f ? "lod1"
                     : "lod2";

    if (targetLod != layer.CurrentLod)
    {
        // Hysteresis: only switch if past threshold + 0.2m
        if (ShouldSwitch(distance, layer.CurrentLod, targetLod))
            layer.SwitchToLod(targetLod);  // swap mesh, no material change needed
    }
}
```

**LOD asset loading:** LOD-0 meshes are large (~150K triangles). They are pre-loaded into memory for the top-3 likely-to-be-activated layers (based on open DTCs at session start). LOD-1 and LOD-2 are loaded for all layers at scene load (small enough to pre-load all).

---

## Guidance Arrow Pool

5 arrow GameObjects pre-instantiated at scene load. On manifest update:
- Arrows not in manifest: returned to pool (disabled)
- Arrows in manifest: positioned at `anchor_point_id` → looked up from `AnchorPointRegistry` (JSON map of anchor point IDs → 3D positions in vehicle frame)
- Arrow faces toward camera at all times (billboard constraint on Y-axis only — no flip)
- Arrow label: TextMeshPro component, always legible (white text, black outline, 0.04m world size)

---

## SafetyGuard Override Channel

SafetyGuard has a dedicated method bypassing the manifest queue:

```csharp
// Called by SafetyGuard — executes on Unity main thread via Dispatcher
public void ApplyHVStop(HVStopEvent evt)
{
    // 1. Dim all non-safety layers immediately
    foreach (var layer in AllBudgetedLayers)
        layer.SetState(LayerState.SafetyDimmed);

    // 2. Force HV bus to maximum visibility
    _hvBusLayer.SetState(LayerState.SafetyHighlight);

    // 3. Activate full-screen red overlay canvas
    _safetyOverlay.SetActive(true);
    _safetyOverlay.SetMessage($"HV ZONE — {evt.ZoneName}\nConfirm: say 'HV aware' or nod");

    // 4. Disable all manifest processing until SafetyGuard clears the stop
    _manifestProcessingEnabled = false;
}

public void ClearHVStop()
{
    _safetyOverlay.SetActive(false);
    _manifestProcessingEnabled = true;
    // Reapply last manifest
    ApplyManifest(_lastManifest);
}
```

**Timing contract:** `ApplyHVStop` must complete within **80ms** of receiving the SafetyGuard signal. Profiled on HoloLens 2 hardware — measured at 12–18ms in worst case (all layers active, LOD-0).

---

## Performance Targets & Profiling Gates

| Metric | Target | Measurement |
|---|---|---|
| Sustained frame rate | ≥ 58fps | Unity Profiler, 10-min session |
| Frame time (GPU) | < 12ms | Unity GPU Profiler |
| Manifest apply time | < 200ms (layer swap) | Stopwatch in `ApplyManifest()` |
| LOD switch time | < 50ms | Stopwatch in `SwitchToLod()` |
| HV-STOP activation | < 80ms | Dedicated timer in `ApplyHVStop()` |
| Memory (heap) | < 512MB | Unity Memory Profiler |
| Anchor re-acquisition | < 10s after headset removal | Manual test, 10 repetitions |

**Profiling gate:** Before Phase 2 demo, run a 30-minute session with all 4 budget layers active. If any frame time exceeds 16ms (= 60fps miss), identify and fix the bottleneck before proceeding.
