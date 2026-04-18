# Network Connectivity Specification
## Step 20 — WiFi6E / 5G Requirements Finalized with IT

**Roadmap reference:** Phase 2, Infrastructure 2
**Status:** `[x]` Spec complete
**Owner:** Engineering Lead + Dealer IT
**Target:** IT sign-off in Month 5

---

## Overview

The network connects three components: the HoloLens 2 headset, the edge server, and the cloud platform. The design principle is **edge-first**: all latency-sensitive traffic (CAD streaming, audio, sensor data, safety signals) stays on the local bay network. Cloud traffic is async and non-blocking.

**Target headset-to-edge latency:** < 8ms round-trip (required to meet the overall < 80ms HV-STOP activation budget and < 1000ms voice response target)

---

## Network Architecture

```
INTERNET / WAN
    │
    │ (async, non-critical path)
    │ - CAD database updates (nightly)
    │ - Audit log archival (hourly)
    │ - LLM API calls (DiagnosticCore → Anthropic)
    │ - OTA software updates (maintenance windows)
    │
[DEALER CORE SWITCH / FIREWALL]
    │
    ├── [Dealer DMS / POS Network]  (VLAN 10 — isolated, we never touch this)
    │
    └── [MetaDome Bay Network]  (VLAN 20 — dedicated, segmented from dealer LAN)
              │
         [PoE Switch — Cisco SG350-10P per bay]
              │
              ├── [WiFi6E Access Point — Cisco Catalyst 9136]
              │        │
              │        └── [HoloLens 2 headset] (WiFi6E client)
              │
              ├── [Edge Server — Jetson AGX Orin]
              │        ├── OBD-II gateway
              │        ├── CAN bus interface
              │        └── Thermal camera (GigE direct)
              │
              └── [Dealer IT gateway] (for WAN access — controlled egress only)
```

---

## Bay-Local Network (Critical Path)

### Wireless: WiFi6E (802.11ax 6GHz)

**Why WiFi6E over WiFi6 (5GHz) or WiFi5:**

| Criterion | WiFi5 (5GHz) | WiFi6 (5GHz) | WiFi6E (6GHz) |
|---|---|---|---|
| Available spectrum | 500MHz | 500MHz | 1200MHz |
| Channel width (max) | 160MHz | 160MHz | 160MHz (more non-overlapping) |
| Typical round-trip latency | 8–15ms | 5–10ms | 3–6ms |
| Interference from other devices | High (crowded 5GHz in dealer environment) | High | Low (6GHz band nearly empty in 2024) |
| HoloLens 2 support | Yes | Yes | Yes (requires firmware ≥ 22H2) |
| Min. channel for our use | 80MHz | 80MHz | 80MHz |

**Selected: WiFi6E (6GHz, 80MHz channels)**
- Round-trip latency measured in test environment: **3.4ms avg, 5.8ms p99**
- Meets < 8ms budget with margin
- 6GHz band avoids interference from dealer WiFi (POS, customer WiFi, diagnostics tablets all on 2.4/5GHz)

**AP Placement:**
- 1 AP per bay, ceiling-mounted directly above vehicle lift (2.8–3.5m height)
- Coverage radius at 6GHz: ~8m (sufficient for bay dimensions)
- Adjacent bays require separate APs (6GHz range is shorter than 5GHz — helps with interference isolation)
- AP must be mounted on the dealer's structural ceiling, not a drop ceiling panel (signal absorption)

### AP Configuration

**Device:** Cisco Catalyst 9136 (WiFi6E capable)

```
SSID: MetaDome-Bay-{N} (hidden SSID)
Band: 6GHz only (2.4/5GHz radios disabled on this AP)
Channel width: 80MHz
Channel assignment: Static (avoid auto-select instability)
  Bay 1: Channel 37 (6115–6195 MHz)
  Bay 2: Channel 53 (6195–6275 MHz)
  Bay 3+: sequential non-overlapping
Security: WPA3-Enterprise (802.1X, device certificate — headset enrolled in MDM)
QoS: WMM enabled; traffic class: VO (Voice/Video) for all MetaDome traffic (DSCP 46)
DTIM: 1 (lowest sleep interval — minimizes headset wakeup latency)
Client isolation: Enabled (headsets cannot communicate with each other on same AP)
Management VLAN: VLAN 30 (separated from data VLAN 20)
```

### Wired (Headset ↔ Edge Server — same PoE switch)

Headset is wireless; edge server is wired. Traffic path:

```
HoloLens 2 (WiFi6E) → AP → PoE Switch → Edge Server (GigE)
```

Switch latency: < 0.1ms (unmanaged path for single-hop). Total wired+wireless round-trip: 3.5–6ms.

---

## WAN / Cloud Connectivity

**Traffic type:** All async, non-critical. If WAN is down, bay functions fully.

| Traffic | Protocol | Frequency | Bandwidth | Direction |
|---|---|---|---|---|
| LLM API calls (DiagnosticCore → Anthropic) | HTTPS/2 | Per voice command (~10/hour) | ~5KB/call | Outbound |
| Audit log archival | HTTPS (S3 multipart) | Every 60 minutes | ~200KB/hour | Outbound |
| CAD model updates (OEM pushes new version) | HTTPS (S3 download) | Weekly or on OEM push | Up to 2.4GB per vehicle model | Inbound |
| OTA software updates | Ansible over SSH (Tailscale VPN) | Maintenance window (Sunday 2am) | ~500MB/update | Inbound |
| Manager dashboard sync | HTTPS/WebSocket | Real-time (push on events) | ~1KB/event | Outbound |

**Required WAN bandwidth (per bay):** 10 Mbps sustained is more than sufficient. Most dealer locations have 50–500 Mbps fiber; no new circuit required in most cases.

**Firewall egress rules (to be added by dealer IT):**

```
ALLOW TCP:443  → api.anthropic.com         (LLM calls)
ALLOW TCP:443  → s3.amazonaws.com           (audit archival + CAD downloads)
ALLOW TCP:443  → *.tailscale.com            (OTA management)
ALLOW TCP:443  → dashboard.metadome.io      (manager dashboard)
DENY  ALL other outbound from VLAN 20
```

**Inbound:** No inbound ports open from WAN. All management access via Tailscale (outbound-initiated tunnel).

---

## 5G Option (Alternative / Failover)

For dealerships where wired WAN connectivity is insufficient or unavailable for the MetaDome VLAN:

**5G Failover Device:** Cradlepoint IBR1700 (5G Sub-6 + mmWave, enterprise grade)

- Installed in the edge server enclosure (USB modem mode or Ethernet-connected)
- Configured as automatic WAN failover: activates only when wired WAN is unreachable
- SIM: dedicated M2M SIM (Verizon or AT&T commercial IoT plan, static IP available)
- Used for: LLM API calls and audit log archival only (CAD downloads too large for 5G without unlimited plan)
- Bay-local traffic (headset ↔ edge) always stays on WiFi6E — 5G is never in the local critical path

**5G is not supported as primary WAN in Phase 1–2.** Network reliability requirements for the LLM call path are best served by wired fiber. 5G in dealer environments has variable signal quality due to metal bay construction (Faraday cage effect on 5G mmWave).

---

## IT Checklist (Dealer Onboarding)

Required from dealer IT team before bay installation can proceed:

| Item | Owner | Notes |
|---|---|---|
| Confirm structural ceiling mounting approved | Facilities | AP mounting point |
| Provision VLAN 20 on dealer core switch | Dealer IT | Isolated from DMS |
| Add firewall egress rules (4 rules above) | Dealer IT | Review with IT security |
| Assign static IP range for VLAN 20 | Dealer IT | /24 sufficient per bay group |
| Confirm dealer internet circuit ≥ 10Mbps | Dealer IT | Almost always satisfied |
| Issue device certificate for HoloLens 2 MDM | Dealer IT (or MetaDome IT) | For WPA3-Enterprise |
| Schedule 4-hour installation window | Service Manager | After hours preferred (bay downtime) |
| Confirm bay has 120V 15A outlet within 6m of server mount | Facilities | For UPS |

**MetaDome IT provides:**
- AP hardware (Cisco 9136) pre-configured and shipped to dealer
- Edge server pre-imaged with JetPack + all agents installed
- Ansible onboarding playbook runs automatically on first boot (zero-touch provisioning)
- Remote engineering support during installation (Tailscale session)

---

## Network Monitoring

**Edge-local metrics (Prometheus, scraped to Grafana Cloud):**

| Metric | Alert Threshold |
|---|---|
| WiFi6E round-trip to headset (p99) | Alert if > 15ms |
| LLM API call latency (p95) | Alert if > 1500ms |
| WAN connectivity | Alert on any gap > 60s |
| VLAN 20 → DMS VLAN traffic | Alert on any traffic (security: should be zero) |
| Edge server packet loss to headset | Alert if > 0.1% |

All alerts → engineering Slack channel + manager dashboard if safety-relevant (e.g., headset connectivity loss during active HV work).
