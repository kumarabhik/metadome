# ArUco Marker Anchoring Specification
## Step 10 — CAD Layer Rendering Anchored to Physical Vehicle

**Roadmap reference:** Phase 1, Deliverable 10
**Status:** `[x]` Spec complete
**Owner:** Engineering Lead (Spatial Computing)
**Target:** Anchoring system integrated in prototype by Month 4

---

## Objective

Define the complete technical specification for anchoring CAD geometry to a physical vehicle using ArUco fiducial markers. This is the foundational spatial registration system for Phase 1. Phase 2 will layer SLAM refinement on top of this; Phase 3 will evaluate marker-free anchoring as a deployment option.

---

## What Is an ArUco Marker?

An ArUco marker is a 2D barcode printed in a high-contrast pattern, designed to be detected by cameras in real-world conditions. Each marker encodes a unique ID and provides its 3D pose (position + rotation) relative to the camera when detected.

**Why ArUco for Phase 1:**
- Detection works on glossy, reflective, or dark surfaces where SLAM fails
- 6-DOF pose estimation from a single camera in < 5ms
- Open-source detection via OpenCV — runs on HoloLens 2 CPU without GPU cost
- Accuracy: ±2–5mm pose error at 1m distance with a 150mm marker

---

## Marker Specification

### Physical Properties

| Property | Value | Rationale |
|---|---|---|
| Dictionary | DICT_6X6_250 (OpenCV) | 6×6 bit pattern; 250 unique IDs; robust to partial occlusion |
| Marker size | 150mm × 150mm | Visible from 2–3m; accurate pose at 0.3–2m working range |
| Border width | 1 cell width (25mm) | Required for detection reliability |
| Print substrate | Matte white paper, laminated | Lamination for durability; matte prevents glare-induced detection failure |
| Adhesion | 3M Scotch Blue painter's tape (3M 2090) | Does not damage vehicle paint; repositionable; holds at 30–120°C |

### Marker IDs (Toyota bZ4X, Phase 1)

Marker IDs are globally unique per vehicle type to prevent cross-vehicle anchoring accidents:

| Marker ID | Vehicle | Placement | Notes |
|---|---|---|---|
| 001 | Toyota bZ4X | Front-left hood corner (driver side) | Primary anchor — highest visibility |
| 002 | Toyota bZ4X | Front-right hood corner (passenger side) | Secondary anchor |
| 003 | Toyota bZ4X | Hood center-rear | Provides rotation constraint |
| 004 | Toyota bZ4X | Driver door sill, mid-point | Provides Z (height) ground truth |
| 011–014 | Ford Mach-E | TBD (Phase 2) | Reserved ID range for Mach-E |
| 021–024 | Stellantis Ram 1500 REV | TBD (Phase 3) | Reserved |

---

## Vehicle Coordinate System Definition

All CAD geometry is authored in the **Vehicle Reference Frame (VRF)**:

```
Origin: Front axle centerline, at ground plane (base of front tire contact patch)

     Z (up)
     │
     │
     │────── Y (vehicle left, positive)
    ╱
   ╱
  X (vehicle forward, positive)

All measurements in millimeters.
```

**ArUco marker positions in VRF (Toyota bZ4X, measured from physical vehicle):**

| Marker | X (mm) | Y (mm) | Z (mm) | Notes |
|---|---|---|---|---|
| 001 | 850 | +920 | 1185 | Measured to marker center |
| 002 | 850 | −920 | 1185 | Mirror of 001 |
| 003 | 1450 | 0 | 1120 | Hood rises toward rear |
| 004 | 1820 | +950 | 0 | Door sill at ground level |

**Measurement protocol:**
- All positions measured from physical vehicle with measuring tape (±5mm precision)
- Verified with digital calipers at anchor attachment points
- Re-measure if vehicle variant (AWD vs FWD) has different hood/door geometry
- Store measurements in `configs/vehicle_anchors/toyota_bz4x_2024.json`

---

## Detection Pipeline

### HoloLens 2 Camera Configuration

| Parameter | Value |
|---|---|
| Camera used | Front-facing world-locking camera (grayscale, 1920×1080) |
| Frame rate for detection | 15fps (sub-sampled from 30fps to reduce CPU load) |
| Resolution for detection | 960×540 (half-resolution crop — sufficient for 150mm markers at 2m) |
| OpenCV ArUco parameters | `adaptiveThreshWinSizeMin=3`, `adaptiveThreshWinSizeMax=23`, `minMarkerPerimeterRate=0.03` |

### Detection Flow

```
[Camera frame at 15fps]
        │
        ▼
[Grayscale conversion]
        │
        ▼
[ArUco detectMarkers()]
        │
        ├── No markers detected → continue scanning, show "Scanning for vehicle markers..." HUD
        │
        └── Markers detected (≥2 required for valid pose)
                │
                ▼
        [estimatePoseSingleMarkers()]
        Outputs: rotation vector (rvec) + translation vector (tvec) per marker
                │
                ▼
        [Multi-marker pose fusion]
        If ≥2 markers visible: average poses weighted by reprojection error
        If only 1 marker visible: use last known pose + SLAM delta (fallback)
                │
                ▼
        [Coordinate transform: Camera frame → VRF]
        Apply camera intrinsics matrix (HoloLens 2 factory calibration)
        Apply headset-to-world transform (from MRTK HoloLens pose)
                │
                ▼
        [CADRenderer anchor update]
        New anchor position sent to Unity GameObject transform
        CAD geometry repositioned in world space
```

### Reprojection Error Threshold

If reprojection error > 5px for any detected marker → marker is occluded or distorted → reject that marker's contribution to pose fusion. System alerts operator if all markers are rejected (tracking lost state).

---

## SLAM Integration (Phase 1 Hybrid Mode)

For Phase 1, SLAM is used to fill gaps when markers are occluded (technician body, vehicle hood open):

| Condition | Primary Anchor Source | Fallback |
|---|---|---|
| ≥2 markers visible | ArUco pose fusion | — |
| 1 marker visible | ArUco single-marker + SLAM delta | — |
| 0 markers visible | SLAM-only (last ArUco anchor as origin) | Drift alert if SLAM-only > 30s |
| SLAM lost | Last known anchor, frozen | "Tracking lost — please look at a vehicle marker" HUD |

**SLAM implementation:** MRTK 3.0 World Locking Tools (WLT) — provides persistent spatial anchors via Azure Spatial Anchors as the backend. WLT handles the drift correction loop automatically once the ArUco-derived origin is set.

---

## Accuracy Validation Protocol

Run before every demo and after every marker repositioning:

**Equipment:** Ruler + measurement template (3D-printed bracket with known geometry)

**Procedure:**
1. Place measurement template at marker 001 position (known 3D location in VRF)
2. Don headset; complete calibration
3. Ask system to render a 50mm red sphere at the known measurement template position
4. Photograph rendered sphere position vs. physical template with GoPro
5. Measure offset in photograph (known scale from template geometry)
6. Record: date, marker placement iteration, X/Y/Z error in mm

**Acceptance criterion:** ±15mm error in all axes at all 4 marker positions.

**Log format:**

```json
{
  "date": "2025-05-15",
  "vehicle": "Toyota bZ4X 2024 AWD",
  "bay": "Bay-01",
  "marker_errors_mm": {
    "001": {"x": 3.1, "y": 2.4, "z": 4.7},
    "002": {"x": 4.2, "y": 3.1, "z": 3.8},
    "003": {"x": 5.0, "y": 2.9, "z": 6.1},
    "004": {"x": 2.8, "y": 4.4, "z": 2.1}
  },
  "max_error_mm": 6.1,
  "pass": true,
  "notes": "Marker 003 had slight bend in laminate; re-flattened"
}
```

---

## Phase 2 Transition: Toward Marker-Free Anchoring

In Phase 2, we will evaluate reducing ArUco dependency using:

1. **Vehicle fingerprinting via LiDAR:** HoloLens 2's depth camera creates a point cloud of the vehicle. Match against a stored reference point cloud (pre-scanned per vehicle model) to derive anchor without markers.

2. **Neural object detection:** Fine-tune a lightweight YOLO model to detect OEM-specific vehicle features (headlight shape, logo position, door gap geometry) for coarse anchor, then refine with SLAM.

3. **Decision criteria for marker-free Phase 2:** If LiDAR-based fingerprinting achieves ±15mm accuracy in 90% of tests across 5 vehicle models, retire ArUco markers from production deployment. Markers retained as fallback for high-drift environments.

Marker-free is preferred for production (markers require dealer preparation and can fall off) but ArUco is the right Phase 1 choice for controlled prototype validation.
