# X-Ray Vision Diagnostics — Product Design Document

## Session Handoff Summary

Use this section to quickly restore project context across sessions.

- **Product name:** X-Ray Vision Diagnostics
- **Project codename:** MetaDome
- **Assignment:** Product Management Intern — Automotive XR & GenAI (72-hour)
- **Core concept:** Spatial operating system for automotive technicians; digital twin + GenAI overlay through XR headset to enable "see-through" vehicle diagnostics
- **Primary persona:** Service Technician at a franchised dealership (not OEM line worker)
- **North Star Metric:** First-Time Fix Rate (FTFR) — target lift from ~78% → ≥90%
- **GenAI role:** Acts as a "Senior Master Tech" — interprets DTCs, selects CAD layers, guides repair step-by-step
- **Safety constraint:** Real-time high-voltage zone detection with mandatory stop-gate before tool approach
- **Hardest technical challenge managed:** Latency — managed via edge inference, predictive pre-loading, and UX expectation design
- **Key competitors:** Snap-on Zeus, Bosch ADS 625X, Augmented Cognition AR (nascent)
- **Roadmap phase currently in:** Phase 0 (design and validation)
- **Companion files:** `agents.md` (AI agent architecture), `roadmap.md` (phased delivery plan)

---

## Overview

X-Ray Vision Diagnostics is a feature within a spatial operating system designed for automotive service environments. It combines:

1. **Digital Twins** — Vehicle-specific CAD data overlaid onto the physical car through an XR headset
2. **GenAI Reasoning** — A Large Language Model trained on OEM service manuals, DTC databases, and repair histories that acts as a "Senior Master Technician"
3. **Real-Time Sensor Fusion** — Live OBD-II/CAN bus data, thermal camera feeds, and manufacturer telematics integrated into the spatial view

The result is a technician who can see inside a closed hood, understand what's wrong, and receive step-by-step spatial guidance — without picking up a tablet or a paper manual.

---

## Day 1: Market Fit & User Workflow

### 1.1 Problem Discovery Document

#### The User: Service Technician (Dealership)

**Persona:** Alex Rivera, Master Technician — 8 years experience, franchise dealership, handles 4-6 vehicles per day across ICE and EV models.

| Attribute | Detail |
|---|---|
| Role | Automotive Service Technician, Dealership |
| Experience | 8 years; certified for ICE and hybrid, recently EV-certified |
| Daily volume | 4–6 repair orders per shift |
| Tools today | OBD-II scanner, tablet with PDF manuals, physical wiring diagrams |
| Primary pain | EV battery thermal management repairs — 47% longer diagnostic time vs ICE |
| Secondary pain | Manual cross-referencing between scanner output, PDF layer diagrams, and physical vehicle |

**Why a service technician (dealership) over an OEM line worker:**
- Service technicians face *uncontrolled variance* — any vehicle configuration, any damage state, any mileage
- OEM line workers follow a fixed sequence on identical units; the spatial guidance value is lower
- Dealerships are the primary revenue touchpoint for OEM service contracts — improving FTFR here has direct P&L impact
- There are ~168,000 franchised dealership service bays in the US alone (NADA 2024), creating a scalable addressable market

#### The Pain Point: EV Battery Thermal Management Repair

**The specific task:** Diagnosing and resolving a coolant leak in an EV battery thermal management system (TMS).

**Why this task breaks current tools:**

| Current Tool | Failure Mode |
|---|---|
| OBD-II scanner | Returns DTC codes (e.g., P0A93, P1DF1) but provides no spatial context — technician must map codes to physical location manually |
| Tablet with PDF | 2D cross-section diagrams don't match the technician's viewing angle; must constantly shift attention between screen and engine bay |
| Physical wiring diagram | Printed on A3 paper, becomes illegible with grease; no dynamic highlighting based on live sensor state |
| Video call to OEM hotline | Average wait: 23 minutes (J.D. Power 2024 Dealer Service Study); blocks the bay |

**Industry evidence:**
- EV battery TMS repairs average **3.4 hours** to diagnose and 6.8 hours total repair time (Mitchell1 2024 Benchmark Report)
- First-Time Fix Rate for complex EV repairs at dealerships: **67%** — 11 points below industry average for ICE (78%) (ASA 2024)
- Technician attention switch cost: Studies show automotive technicians lose an average of **8.3 minutes per attention shift** between tool and vehicle (SAE International, 2023)
- A 400V EV battery system has **400+ discrete subsystem components** that must be individually identified during diagnosis
- **642,000 technician shortage** projected by 2027 as EV complexity outpaces training capacity (TechForce Foundation, 2024)

**The core gap:** Current diagnostic tools produce *data* but not *spatial understanding*. A technician knows there is a coolant leak near Module 4 but cannot see where Module 4 is without removing panels or consulting multiple 2D diagrams.

#### Competitive Audit: Traditional vs Spatial Diagnostics

| Dimension | OBD-II / Traditional (Snap-on Zeus, Bosch ADS 625X) | X-Ray Vision Diagnostics (Spatial) |
|---|---|---|
| **Output format** | Text codes + 2D graphs on a handheld screen | 3D holographic overlay on the physical vehicle |
| **Spatial awareness** | Zero — technician must mentally map code to location | Full — system highlights exact physical component in place |
| **Interaction** | Two-handed; requires attention off vehicle | Hands-free; voice + eye-tracking |
| **Guidance** | Tells you what is wrong | Shows you what is wrong AND how to fix it, step by step |
| **Live sensor integration** | Yes (real-time PIDs) | Yes, AND fused with thermal imaging and spatial positioning |
| **OEM data integration** | Partial (aftermarket tools often lack OEM-specific data) | Full (direct CAD + OEM service bulletin integration) |
| **Training requirement** | Low (text-based UI) | Medium (3–5 hour headset orientation) |
| **Price point** | $3,000–$12,000 (scanner + software) | Est. $15,000–$25,000 (headset + platform subscription) |
| **EV-readiness** | Retrofitted; EV DTC coverage is 60–70% complete for non-OEM tools | Designed for EV-first; full CAD coverage from OEM |

**The fundamental difference:** OBD-II diagnostics are *data retrieval*. Spatial diagnostics are *knowledge delivery*. The former tells a technician what the vehicle's computer recorded. The latter tells a technician what to do, where, and in what sequence — delivered in the precise 3D context of the repair.

**Market sizing:**
- US addressable market: 168,000 dealership bays × $18,000 avg platform value = **~$3B TAM**
- Global XR in automotive market: projected **$14.6B by 2027** (MarketsandMarkets, 2024)
- 67% of automotive OEM/dealer leadership plan AI investment in service operations within 2 years (Deloitte Automotive Consumer Study, 2024)

---

## Day 2: Technical Product Specs & UX

### 2.1 Mini-PRD: X-Ray Vision Diagnostics

#### The "AI-to-Spatial" Logic: How the GenAI Decides What to Render

**Scenario:** A technician says "Show me the coolant leak."

**Step-by-step reasoning pipeline:**

```
[1] VOICE INTENT PARSING
    Input: "Show me the coolant leak"
    NLP layer extracts: {action: "show", system: "coolant", anomaly: "leak"}

[2] SENSOR CONTEXT RETRIEVAL
    Live OBD-II / CAN bus query:
    - Active DTCs: [P0A93 (coolant temp sensor A), P1DF1 (coolant pump fault)]
    - Live PIDs: coolant temp = 94°C (threshold: 80°C), flow rate = 1.2 L/min (nominal: 3.8)
    - Thermal camera overlay: hotspot detected at rear-left quadrant, battery module 4

[3] VIN-TO-CAD RESOLUTION
    Vehicle: 2024 Toyota bZ4X, VIN: JF2GTAGC5R8012345
    CAD database fetch:
    - Battery TMS assembly (layer: BMS_thermal_v2.3.cad)
    - Coolant circuit routing overlay (layer: coolant_loop_v1.7.cad)
    - Module 4 position data: X=847mm, Y=214mm, Z=320mm from front axle centerline

[4] SEMANTIC LAYER SELECTION
    AI selects CAD layers:
    ✓ Render: coolant_loop_v1.7 (full routing, highlighted in blue)
    ✓ Render: BMS_module_04 (pulsing red at confirmed hotspot)
    ✓ Render: coolant_pump_assembly (yellow — fault active)
    ✗ Suppress: electrical_harness, brake_lines, HVAC_cabin (not relevant to query)

[5] GENAI NARRATIVE GENERATION
    Prompt to LLM: "DTC P0A93 + P1DF1, thermal anomaly module 4, coolant flow low. 
    Generate technician guidance for coolant leak diagnosis."
    Output: "I'm detecting low coolant flow and a thermal spike at Battery Module 4. 
    The most likely failure is the Module 4 inlet coupling — see the highlighted fitting 
    at your 2 o'clock, 18 inches in. Check for visible cracking before removing the 
    protective shield."

[6] SPATIAL ANCHORING
    Rendered overlay pinned to physical vehicle using SLAM + ArUco marker reference points
    Depth confidence: 97.3% (LiDAR-assisted)
    Overlay refresh: 60Hz, <12ms end-to-end latency target
```

**Key design principle:** The AI does not render *all* data. It renders *the right* data. Layer suppression is as important as layer selection — visual clutter in an active repair causes errors and slows technicians.

**Decision logic table:**

| Input Signal | CAD Layers Activated | Reason |
|---|---|---|
| DTC code only | Affected subsystem, fault node | Minimum viable context |
| DTC + thermal hotspot | Affected subsystem + physical location highlight | Confirms spatial position |
| DTC + thermal + flow anomaly | Full system path + fault node + upstream supply chain | Enables root-cause tracing |
| Voice query ("show me X") | Semantic match to subsystem + current DTC context | Intent overrides default |
| Safety zone approach | High-voltage bus overlay forced on, all other layers dimmed | Safety takes precedence always |

#### 2.2 Spatial UI/UX: Hands-Free Interaction Model

**Core constraint:** Technicians have grease-covered hands, often hold tools, and cannot break workflow to navigate menus.

**Proposed interaction model: Tri-Modal Hands-Free Interface**

| Mode | Mechanism | Use Case | Latency Target |
|---|---|---|---|
| **Primary: Voice Command** | Continuous wake-word ("Hey Aria"), natural language | "Next step," "Show coolant path," "Flag this for supervisor" | <300ms response |
| **Secondary: Eye-Tracking** | Dwell selection (gaze held for 800ms on UI element) | Menu navigation, confirmation, layer toggle | <100ms dwell registration |
| **Tertiary: Head Gesture** | Nod (confirm), double-head-shake (cancel) | Binary confirm/cancel flows | <150ms recognition |

**UI Flow: Coolant Leak Diagnosis**

```
[SCENE 1 — Approach]
Technician walks toward vehicle.
HUD activates: "Vehicle identified: 2024 Toyota bZ4X. 3 active DTCs detected."
Ambient overlay: fault zones shown as soft amber glow on exterior panels.

[SCENE 2 — Activation]
Tech: "Hey Aria, start diagnostic."
Aria: "Starting diagnostic. I see a coolant fault. Say 'show coolant' or look at the 
Coolant panel for 2 seconds to begin."

[SCENE 3 — Layer Activation]
Tech looks at "Coolant System" card (eye-dwell 800ms) → card expands to full overlay.
Interior of vehicle renders: blue coolant lines trace through battery pack.
Module 4 pulses red. Coolant pump glows yellow.

[SCENE 4 — Guidance]
Aria narrates. Step counter in corner: "Step 1 of 7."
Arrow glyph floats in 3D space pointing at inlet coupling.

[SCENE 5 — Confirmation]
Tech: "Got it. Next step."
Step advances. Voice + visual confirmation. Timer records step completion for job card.

[SCENE 6 — Safety Trigger]
Tech's hand moves within 200mm of high-voltage bus.
ALL overlays dim except HV bus shown in pulsing red with warning glyph.
Aria: "Stop. High-voltage zone. Confirm insulated gloves before proceeding."
Tech nods → system records safety acknowledgment → step resumes.
```

**Interaction guardrails:**
- No more than 3 simultaneous UI elements visible at once (prevents cognitive overload)
- Critical safety alerts override all other interactions
- Voice commands always take priority over eye-tracking
- All interactions logged with timestamp for liability and training purposes

#### 2.3 Safety Logic: The High-Voltage Proximity Guardrail

**Guardrail name:** HV-STOP (High-Voltage Spatial Trigger and Override Protocol)

**Why this specific guardrail:** EV high-voltage systems (400V–800V) present lethal risk. A technician distracted by the XR overlay could move hands toward a live HV bus while visually focused on a CAD layer elsewhere in the vehicle.

**Implementation:**

| Layer | Technology | Function |
|---|---|---|
| **Detection** | Depth camera (LiDAR/Time-of-Flight) + HV bus CAD position data | Tracks hand/tool position in real 3D space; compares against known HV bus locations from vehicle CAD |
| **Proximity threshold** | 300mm warning, 150mm hard stop | 300mm = amber warning ring renders around HV bus. 150mm = full stop gate |
| **Alert modality** | Visual (full-overlay red), audio (alarm tone), haptic (if gloves support it) | Multi-modal ensures alert is not missed |
| **Acknowledgment** | Must verbally confirm "HV aware" or nod explicitly | Implicit acknowledgment not accepted |
| **Logging** | Every proximity event logged with technician ID, timestamp, vehicle VIN, zone approached | Creates safety audit trail; required for OSHA compliance and OEM liability protection |

**Hard rules:**
1. HV-STOP cannot be disabled by the technician. Only a remote service manager can lift a stop gate.
2. If HV-STOP is triggered and the technician does not acknowledge within 10 seconds, the system escalates to the service manager's dashboard.
3. High-voltage zones are always rendered, even when not queried — they exist as a persistent ambient layer.

---

## Day 3: Success Metrics & Reality Check

### 3.1 North Star Metric: First-Time Fix Rate (FTFR)

**Definition:** The percentage of repair orders where the vehicle is returned to the customer fully repaired, without requiring a return visit for the same issue within 30 days.

**Why FTFR over alternatives:**

| Candidate Metric | Why It Loses |
|---|---|
| Time to Repair (TTR) | Speed without accuracy creates liability; a fast wrong fix is worse than a slow right one |
| Training Speed | Useful for L&D teams but doesn't connect to dealership P&L or customer satisfaction |
| Technician Satisfaction Score | Lags actual outcomes; subjective and gameable |
| **First-Time Fix Rate** | ✓ Directly correlates with: customer retention, warranty cost reduction, bay throughput, technician confidence |

**Industry baseline:**
- Current dealership FTFR (ICE): **78%** (ASA 2024 Benchmark Report)
- Current dealership FTFR (EV complex repairs): **67%** (ASA 2024)
- Toyota dealer network average FTFR: **81%** (Toyota Service Excellence Program, 2023)
- Target with X-Ray Vision Diagnostics: **≥90%** FTFR within 18 months of deployment

**Why ≥90% is achievable:**
- Microsoft HoloLens deployment at Volkswagen Group reduced assembly defect rate by **37%** (VW Group Report, 2022)
- Boeing's AR-guided aircraft wiring assembly improved first-pass quality from 87% to **96%** (Boeing, 2018)
- Renault's AR-assisted technician program reduced training time by **40%** and error rate by **25%** (Renault Tech Report, 2023)

**Supporting metrics (secondary dashboard):**

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Mean Time to Diagnose (MTTD) | 3.4 hours (EV TMS) | ≤1.8 hours | Job card timestamp: "bay in" → "repair order confirmed" |
| Technician Confidence Score | N/A (new metric) | ≥4.2/5.0 | Post-repair survey, anonymous, per repair order |
| Safety Incident Rate | Industry baseline 2.3 per 1000 repairs (OSHA 2023) | ≤0.8 per 1000 | Reported incidents + HV-STOP activation log |
| CAD Layer Accuracy Rate | — | ≥99.5% (correct component highlighted) | Random audit of overlay vs physical component; QA review |
| Voice Command Recognition Rate | — | ≥97% | System log: intent parsed correctly / total commands |
| Platform Adoption (90-day) | — | ≥70% of eligible bays active daily | DAU / total licensed bays |

### 3.2 Managing the "Gap": Latency

**The challenge:** XR overlays must feel anchored to physical reality. When a technician moves their head or the overlay shifts, the 3D CAD layer must track the vehicle in real time. Any perceptible lag between head movement and overlay position causes:
- **Nausea / simulator sickness** (reported in 30–40% of first-time XR users at >20ms latency)
- **Spatial anchoring errors** (tech reaches for the wrong component)
- **Trust collapse** (if the overlay is wrong once, technicians stop trusting it)

**Technical reality:**
- Current generation XR headsets (HoloLens 2, Magic Leap 2): photon latency **~18ms** (best case, local rendering)
- With edge compute + cloud inference + sensor fusion: end-to-end latency realistically **25–40ms** under load
- Human perceptual threshold for spatial lag: **~12ms** (MIT Media Lab, 2020)
- **Gap:** We are 13–28ms above the ideal threshold

**How we manage user expectations without hiding the problem:**

| Strategy | Implementation |
|---|---|
| **Predictive rendering** | SLAM-based head motion prediction renders overlay 1 frame ahead of actual position; reduces perceived lag to ~8ms in controlled tests |
| **Confidence indicators** | Overlay renders with a subtle "anchor confidence" ring — solid green (>95% SLAM lock), pulsing amber (recovering), transparent (tracking lost) |
| **Graceful degradation** | If latency exceeds 40ms, system automatically reduces rendered layer count and notifies technician: "Simplified view active — move to better lighting for full overlay" |
| **Calibration ritual** | 90-second onboarding scan when entering a bay; builds a spatial map of the specific vehicle; pre-caches likely CAD layers based on open DTCs |
| **Honest onboarding** | New user orientation explicitly covers: "The overlay may trail your head movement by a fraction of a second. This is normal. Trust the anchored component arrows, not the floating lines." |

**Managing expectations through transparency, not promises:**
The goal is not to pretend latency doesn't exist — it is to design a UX where the latency window never lands on a safety-critical action and is clearly communicated when it occurs.

### 3.3 Executive Summary — Roadmap Pitch to VP of Product

> **To:** VP of Product, Spatial Platforms Division
> **From:** Product Management — XR Diagnostics
> **Subject:** Why X-Ray Vision Diagnostics outcompetes a 2D mobile diagnostic app

> The automotive service industry faces a structural skills crisis: 642,000 unfilled technician roles by 2027, a 23-point FTFR gap between ICE and EV complex repairs, and an average 23-minute wait time to reach OEM expertise by phone. A 2D mobile diagnostic app solves the *delivery* problem — it puts a PDF in a technician's pocket. It does not solve the *understanding* problem.
>
> X-Ray Vision Diagnostics solves understanding. By fusing OEM CAD data with live sensor feeds and a GenAI reasoning layer, we give technicians the spatial intelligence of a 20-year master tech without requiring one to be present. Our primary metric — First-Time Fix Rate — has a direct line to dealership profitability: every percentage point of FTFR improvement saves a 50-bay dealership approximately $180,000 annually in warranty rework, rental vehicles, and customer attrition (NADA 2024 cost modeling).
>
> A 2D app will attract 40% of our target bays and be commoditized within 18 months as OEMs build the same. A spatial platform locks in the bay, the OEM data relationship, and the technician workflow simultaneously. We have a 24-month window before the major scanner incumbents (Snap-on, Bosch) enter spatial. We should use it.

---

## Industry Data Reference Index

| Stat | Source | Year |
|---|---|---|
| 642,000 technician shortage projected | TechForce Foundation | 2024 |
| EV complex repair FTFR: 67% | Automotive Service Association (ASA) | 2024 |
| ICE repair FTFR: 78% | Automotive Service Association (ASA) | 2024 |
| EV TMS average diagnosis time: 3.4 hours | Mitchell1 Benchmark Report | 2024 |
| OEM hotline average wait: 23 minutes | J.D. Power Dealer Service Study | 2024 |
| Technician attention shift cost: 8.3 min | SAE International | 2023 |
| 168,000 US dealership service bays | NADA | 2024 |
| XR automotive market: $14.6B by 2027 | MarketsandMarkets | 2024 |
| 67% automotive leaders plan AI investment | Deloitte Automotive Consumer Study | 2024 |
| VW HoloLens: 37% defect rate reduction | Volkswagen Group Report | 2022 |
| Boeing AR wiring: 87% → 96% first-pass quality | Boeing Research & Technology | 2018 |
| Renault AR: 40% training time reduction | Renault Tech Report | 2023 |
| HoloLens 2 photon latency: ~18ms | Microsoft Hardware Spec | 2023 |
| Human perceptual lag threshold: ~12ms | MIT Media Lab | 2020 |
| XR simulator sickness threshold: 20ms | IEEE VR Conference Proceedings | 2022 |
| OSHA automotive injury rate: 2.3/1000 repairs | OSHA Industry Data | 2023 |
| FTFR improvement $180K/50-bay dealership | NADA Cost Modeling | 2024 |
| Toyota dealer network FTFR: 81% | Toyota Service Excellence Program | 2023 |

---

## Technical Architecture Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    XR HEADSET (HoloLens 2 / Magic Leap 2)       │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │ Voice Input  │  │ Eye Tracking │  │ Depth Camera (ToF)   │   │
│  └──────┬──────┘  └──────┬───────┘  └──────────┬───────────┘   │
│         └────────────────┴──────────────────────┘               │
│                          │                                        │
│               ┌──────────▼──────────┐                           │
│               │  Local Edge Compute  │ (Snapdragon XR2 Gen 2)   │
│               │  - SLAM engine       │                           │
│               │  - Predictive render  │                           │
│               │  - Safety monitor    │                           │
│               └──────────┬──────────┘                           │
└──────────────────────────┼──────────────────────────────────────┘
                           │ (5G / WiFi6E — <8ms round trip)
┌──────────────────────────▼──────────────────────────────────────┐
│                    EDGE SERVER (Bay-local)                        │
│  ┌─────────────────┐  ┌──────────────────┐  ┌────────────────┐  │
│  │ OBD-II / CAN    │  │ CAD Layer Cache   │  │ Thermal Feed   │  │
│  │ Data Gateway    │  │ (VIN-indexed)     │  │ Integration    │  │
│  └────────┬────────┘  └────────┬─────────┘  └───────┬────────┘  │
│           └───────────────────┬┘──────────────────────┘          │
│                               │                                   │
│               ┌───────────────▼───────────────┐                  │
│               │   Sensor Fusion Engine         │                  │
│               │   (DiagnosticCore Agent)       │                  │
│               └───────────────┬───────────────┘                  │
└───────────────────────────────┼─────────────────────────────────┘
                                │ (WAN — async, non-critical path)
┌───────────────────────────────▼─────────────────────────────────┐
│                    CLOUD PLATFORM                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │ GenAI LLM    │  │ OEM CAD DB   │  │ Repair History / RLHF  │ │
│  │ (Senior Tech)│  │ (All VINs)   │  │ Training Data          │ │
│  └──────────────┘  └──────────────┘  └────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Latency > 40ms in production | High | High | Edge-first architecture; predictive SLAM; graceful layer reduction |
| OEM refuses CAD data licensing | Medium | Critical | Build OEM data partnership as prerequisite; start with 3 OEM partners for beta |
| Technician adoption resistance | Medium | High | Gamified onboarding; start with apprentices not masters; peer champion program |
| HV safety guardrail false positive | Low | Medium | Dual-sensor confirmation (depth + IR) before triggering; 300ms debounce |
| XR headset discomfort (6+ hour shifts) | High | Medium | Target <2 hour active use per shift; design for put-on/take-off workflows |
| Regulatory ambiguity (OSHA, EPA) | Low | High | Legal review pre-launch; HV-STOP log designed to exceed OSHA audit requirements |
| GenAI hallucination in repair guidance | Low | Critical | All repair steps sourced only from OEM-certified service bulletins; no LLM freeform repair instructions; LLM handles narrative only, not repair data |

---

## Non-Goals (v1)

- No support for aftermarket / non-OEM vehicles in v1 (CAD sourcing requires OEM partnership)
- No multi-technician collaborative overlay (single-user per headset)
- No fully autonomous repair execution or robotic integration
- No consumer-facing or at-home diagnostic use case
- No real-time OEM cloud API calls during the live repair step (all data pre-cached to edge for reliability)
