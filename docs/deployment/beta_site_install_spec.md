# Beta Site Installation Spec
## Step 29 — Phase 3, Beta Deployment 1: 3 Dealership Bays

**Roadmap reference:** Phase 3, Beta Deployment 1
**Status:** `[ ]` Not started
**Owner:** Engineering Lead + Sales/BD
**Target:** All 3 bays installed and operational by Month 9

---

## Beta Site Selection

### Target Sites

| Site | OEM | Location | Bays Selected | DMS | Rationale |
|---|---|---|---|---|---|
| Toyota of Bellevue | Toyota | Bellevue, WA | Bay 3 (EV dedicated) | CDK Global | High EV volume; existing OEM research relationship |
| Toyota of Mission Viejo | Toyota | Mission Viejo, CA | Bay 5 (EV dedicated) | CDK Global | Strong service director champion; tech demographics match persona |
| Ford of Kirkland | Ford | Kirkland, WA | Bay 2 (EV dedicated) | Reynolds & Reynolds | Geographic proximity to HQ for rapid issue response |

### Site Selection Criteria
1. ≥ 2 EV repairs per week on enrolled fault types (volume floor for FTFR statistical significance)
2. Service manager committed to weekly check-in during beta
3. At least 1 "peer champion" technician identified (early adopter willing to onboard colleagues)
4. DMS is CDK or R&R (Phase 3 integration scope)
5. Network: WiFi6E capable or willing to install access point in bay

---

## Pre-Installation Checklist (Per Bay)

### Site Survey (4 weeks before install)
- [ ] Measure bay dimensions (min 5m × 8m required for SLAM anchor coverage)
- [ ] Map existing lighting fixtures; identify lux levels at 4 corners (target > 200 lux)
- [ ] Identify network infrastructure: existing WiFi, Ethernet drops, router location
- [ ] Identify 4× ArUco marker mounting locations (floor, wall, column — no vehicle contact)
- [ ] Identify edge server mounting location (wall-mounted rack, < 3m from OBD-II port reach)
- [ ] Identify power outlet for edge server (standard 120V / 15A)
- [ ] Confirm OBD-II cable reach from server rack to center of bay (max 5m extension)
- [ ] Confirm thermal camera mounting position: ceiling or column, angled for full vehicle coverage

### Network Provisioning (2 weeks before install)
- [ ] Install WiFi6E access point per `docs/infrastructure/network_connectivity_spec.md`
- [ ] Configure SSID: `XRAY_BAY_[N]` (isolated VLAN, no internet access for edge server)
- [ ] Open firewall ports: 443 (Claude API), 5432 (cloud DB sync), 4317 (OTEL)
- [ ] Test bandwidth: ≥ 500Mbps upload from edge server to cloud
- [ ] Assign static IP to edge server on dealer VLAN

### Hardware Delivery (1 week before install)
- [ ] 1× Jetson AGX Orin (64GB) — configured per `docs/infrastructure/edge_server_bay_spec.md`
- [ ] 2× HoloLens 2 (one active, one spare)
- [ ] 1× FLIR Lepton 3.5 thermal camera + USB-C mount arm
- [ ] 4× ArUco marker boards (laminated, 200mm × 200mm, ID 0x10–0x13)
- [ ] 1× OBD-II CAN bus adapter (USB-C) + 5m cable
- [ ] 1× WiFi6E access point (if not provided by dealer IT)
- [ ] 1× wall-mount server rack (2U)
- [ ] 1× HoloLens 2 charging station (5-port)

---

## Installation Procedure

### Day 1: Infrastructure (4 hours)

**Step 1: Edge Server Rack Mount**
- Mount 2U rack on wall at 1.5m height, adjacent to power outlet
- Install Jetson AGX Orin in rack; connect power
- Connect WiFi6E access point to rack via Cat6

**Step 2: Thermal Camera**
- Mount FLIR Lepton arm on ceiling junction box or column
- Route USB-C cable to edge server (cable management clips every 30cm)
- Verify camera orientation: full vehicle footprint in frame (5m × 2m)

**Step 3: ArUco Markers**
- Apply markers at 4 anchor points per `docs/prototype/aruco_anchoring_spec.md`
- Photograph marker placement with measurements; file in site install log

**Step 4: Network Verification**
- Connect edge server to WiFi6E AP; confirm static IP assigned
- Run connectivity test: `curl https://api.anthropic.com` — expect 200
- Run OTEL connectivity test: confirm Jaeger span received

### Day 2: Software Configuration (3 hours)

**Step 1: VIN Pre-Cache**
- Load dealer's top 10 VINs into CAD pre-cache (provided by service manager)
- Verify cache via CLI: `python cache_verify.py --vin [VIN]` → expect "CACHED"

**Step 2: Headset Pairing**
- Pair both HoloLens 2 units to edge server IP
- Confirm spatial anchor initialization with ArUco markers (confidence > 0.85)
- Run "Hello World" overlay: simple colored sphere at vehicle origin

**Step 3: Agent Health Check**
- Start all 5 agents; verify health endpoint: `GET /health` → `{"status": "healthy"}`
- Run sample diagnostic query (no vehicle): confirm DiagnosticCore returns manifest

**Step 4: End-to-End Smoke Test**
- Drive Toyota bZ4X into bay (or use test fixture)
- Tech dons HoloLens 2
- Say "Show me the coolant system"
- Verify: CAD layer activates within 40ms perceived; TMS layers visible
- Log: smoke test pass/fail in site install log

### Day 3: Acceptance Test (2 hours)

- Full coolant diagnosis flow from voice command to guided repair (per `docs/features/coolant_diagnosis_flow.md`)
- HV-STOP test: simulate proximity, verify trigger in < 80ms
- Manager Dashboard: confirm session visible in dashboard within 60s of session close
- Job card integration: confirm CDK/R&R receives RO update (using test RO number)
- Sign install acceptance form: Service Manager + Engineering Lead

---

## Post-Install Monitoring (First 2 Weeks)

- Engineering checks edge server logs daily (remote SSH)
- On-call: Engineering Lead on call 8am–8pm local time for first 7 days
- Manager Dashboard: Engineering PM reviews daily for anomalies

---

## Rollback Plan

If a critical issue occurs post-install:
1. Headsets powered off and placed on charging station
2. Edge server set to `maintenance_mode` (disables all agent activity, preserves logs)
3. Tech reverts to standard scan tool workflow (zero customer impact)
4. Engineering diagnoses remotely; targeted fix deployed before next business day
5. No bay downtime expected > 1 business day for any non-hardware issue

---

## Exit Criteria

- [ ] All 3 bays pass Day 3 acceptance test
- [ ] Manager Dashboard shows all 3 bays active
- [ ] No P0 (safety) issues in first 48 hours of operation
- [ ] HV-STOP confirmed functional at all 3 sites before any live vehicle work begins
