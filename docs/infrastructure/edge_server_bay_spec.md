# Edge Server Bay Installation Specification
## Step 19 — Hardware Selection and Bay Installation

**Roadmap reference:** Phase 2, Infrastructure 1
**Status:** `[x]` Spec complete
**Owner:** Engineering Lead + Facilities
**Target:** Installed in 2 test bays by Month 5 (before agent builds begin)

---

## Overview

Each service bay requires a dedicated edge server. The edge server is the compute backbone of the entire system — it runs all 5 agents, manages the CAD cache, operates the OBD-II/CAN gateway, and streams data to the headset. Cloud connectivity is used only for CAD updates and audit log archiving; it is never on the critical repair path.

**Design principle:** If the internet goes down, the bay must continue to function at full capability for ongoing sessions.

---

## Hardware Selection

### Compute: NVIDIA Jetson AGX Orin 64GB

**Why this platform:**

| Requirement | Jetson AGX Orin 64GB | Rationale |
|---|---|---|
| ML inference (Whisper, DistilBERT) | 275 TOPS (INT8) | Runs Whisper-medium at 4× real-time; DistilBERT at < 15ms |
| CAD layer streaming | 12-core ARM CPU + LPDDR5 | Sufficient for gLTF streaming + gRPC to headset |
| Thermal (automotive bay: 0–45°C) | Operating range: -25 to 60°C | Handles bay heat without active cooling concerns |
| Power | 15–60W (configurable) | Set to 30W profile for balance of performance + heat |
| Storage interface | NVMe PCIe Gen4 | Required for < 200ms layer load from SSD |
| Connectivity | 2× GigE, 2× USB 3.2 Gen2, PCIe | All required interfaces present |

**Comparable alternatives evaluated:**

| Platform | Decision | Reason |
|---|---|---|
| Intel NUC 13 Pro (i7) | Rejected | No GPU for Whisper inference; higher latency |
| NVIDIA RTX 4060 workstation | Rejected | Too large for bay enclosure; overkill cost |
| Raspberry Pi 5 | Rejected | Insufficient ML inference capability |
| Jetson Orin NX 16GB | Rejected | 16GB insufficient for CAD cache + model weights simultaneously |
| **Jetson AGX Orin 64GB** | ✓ Selected | Best fit on performance, size, thermal, and ML capability |

### Storage: 2× Samsung 990 Pro 2TB NVMe (RAID-1)

- RAID-1 mirror: if one drive fails, bay continues operating without data loss
- 2TB each: sufficient for 3 OEM brands × 5 vehicle models × 3 years × ~2.4GB = ~108GB. Plenty of headroom.
- Sequential read: 7,450 MB/s → layer load from SSD: < 30ms for a 150MB LOD-0 layer

### Enclosure: Pelican iM2950 (Modified)

- IP67-rated: dustproof and splash-resistant (appropriate for bay environment with coolant, brake fluid)
- Rack panel removed; Jetson mounted on custom aluminum bracket (fabricated in-house, $120 material cost)
- Ventilation: 2× 80mm Noctua NF-A8 fans (low noise, dust-filtered), side-mounted
- Cable management: rear panel with 4× cable glands (USB, Ethernet, power, OBD-II)
- Mounted: on wall-mounted strut channel (Unistrut) at technician shoulder height, behind lift column — out of vehicle clearance zone

### UPS: APC Back-UPS Pro 1500VA (BX1500M)

- 1500VA / 900W capacity
- Runtime at 30W Jetson load: approximately 4.5 hours
- Runtime at full load (Jetson + switch + camera): approximately 1.5 hours
- Purpose: clean shutdown on power loss; avoid filesystem corruption on Jetson during active session
- On power loss: edge server broadcasts "Power loss — save session" to headset; closes audit log cleanly; shuts down within 60 seconds

---

## Bay Layout

```
BAY TOP VIEW (12m × 6m standard dealership bay)
┌──────────────────────────────────────────────┐
│                                              │
│   VEHICLE                                    │
│   ┌──────────────────────────────────────┐  │
│   │              Toyota bZ4X             │  │
│   └──────────────────────────────────────┘  │
│                                              │
│  ┌──────────┐                               │
│  │LIFT POST │  ← Edge server mounts here    │
│  │          │    (Unistrut bracket,          │
│  │ [SERVER] │     1.4m height)               │
│  └──────────┘                               │
│                                              │
│  [WIFI6E AP]  ← Ceiling mount, above bay    │
│                                              │
│  [THERMAL]    ← Overhead arm,               │
│  [CAMERA]       above engine bay            │
│                                              │
│  [OBD-II]     ← Cable from server to        │
│  [CABLE]        vehicle OBD port             │
└──────────────────────────────────────────────┘
```

---

## Wiring Diagram

```
[Shore Power 120V AC]
        │
    [UPS 1500VA]
        │
  ┌─────┴─────────────────────┐
  │ Jetson AGX Orin (19V DC)  │
  │ WiFi6E AP (PoE)           │
  └───────────────────────────┘

[Jetson USB 3.2 Gen2]
  ├── Kvaser Leaf Light HS v2  ─────→ OBD-II cable (3m, locking J1962)
  ├── Peak PCAN-USB Pro        ─────→ CAN breakout cable (2m)
  └── Reserved (USB-A)

[Jetson GigE Port 1]  ─────────→ FLIR A50 Thermal Camera (static IP 192.168.1.100)

[Jetson GigE Port 2]  ─────────→ PoE Switch (Cisco SG350-10P)
                                         │
                                    [WiFi6E AP] ─── HoloLens 2 (WiFi6E client)

[Jetson NVMe PCIe]
  ├── Samsung 990 Pro 2TB (drive 0, RAID-1)
  └── Samsung 990 Pro 2TB (drive 1, RAID-1)
```

---

## Software Stack (Edge Server OS)

| Layer | Component |
|---|---|
| OS | JetPack 6.0 (Ubuntu 22.04 LTS + NVIDIA L4T) |
| Container runtime | Docker 24 + NVIDIA Container Toolkit |
| Service management | systemd (one service per agent: diagnostic-core, sensor-fusion, cad-renderer-proxy, voice-nlp, safety-guard) |
| Watchdog | systemd watchdog (WatchdogSec=5) for each agent service |
| Log management | Journald + logrotate (50MB per service log, 7-day retention on edge) |
| Remote management | Tailscale (zero-config VPN) for remote SSH by engineering team |
| OTA updates | Ansible playbook pushed by engineering; updates applied during non-business hours |

---

## Installation Procedure (Per Bay)

**Estimated installation time:** 4 hours per bay, 2-person crew (1 electrician, 1 AV/IT tech)

| Step | Duration | Crew |
|---|---|---|
| 1. Mount Unistrut bracket on lift column | 30 min | Electrician |
| 2. Mount server enclosure on bracket | 20 min | Both |
| 3. Run power from bay circuit to UPS | 45 min | Electrician |
| 4. Mount WiFi6E AP on ceiling (above vehicle) | 30 min | IT |
| 5. Run Cat6A from switch to AP | 20 min | IT |
| 6. Mount thermal camera arm; run GigE cable to server | 30 min | Both |
| 7. Route OBD-II and CAN cables; test plug into vehicle | 20 min | IT |
| 8. Power-on Jetson; confirm JetPack boot | 10 min | IT |
| 9. Deploy agent Docker containers via Ansible | 30 min | IT |
| 10. Run calibration and bay self-test | 25 min | IT |

**Bay self-test checklist (run after installation):**
- [ ] All 5 agent services running: `systemctl status diagnostic-core sensor-fusion cad-renderer-proxy voice-nlp safety-guard`
- [ ] OBD-II connected: `python -c "import obd; c = obd.OBD(); print(c.status())"`
- [ ] CAN bus active: `cansend can0 5D0#0000000000000000; candump can0 -n 1`
- [ ] Thermal camera: `python -c "from flir import A50; cam = A50(); print(cam.get_frame().shape)"`
- [ ] WiFi6E: HoloLens 2 connects, pings edge server < 5ms
- [ ] SafetyGuard: run `tests/safety/smoke_test.py` — 5 proximity scenarios pass
- [ ] Audit log: write test entry, verify hash chain integrity via `audit_verify.py`
- [ ] UPS: simulate power loss (unplug shore power), confirm graceful shutdown initiated

---

## Phase 3 Scale: 50 Bays

For Phase 3 deployment (50 commercial bays):
- Same Jetson AGX Orin hardware (no redesign)
- Fleet management via Ansible Tower (centralized OTA updates to all 50 bays)
- CAD cache pre-staging: new vehicle models pushed to all edges before going live (no first-request cache miss at dealer)
- Monitoring: Prometheus on each Jetson → Grafana Cloud dashboard (agent health, latency percentiles, HV-STOP event counts per bay)
- Bay network: coordinate with dealer IT to ensure WiFi6E AP on isolated SSID (security segmentation from dealer POS and DMS systems)
