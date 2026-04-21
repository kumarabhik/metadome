# Series A Fundraising Narrative

**Round:** Series A
**Target raise:** $18M
**Use of proceeds:** Scale to 500 bays (Phase 5), OEM embedded deal execution, SOC 2 certification, international expansion

---

## The Pitch (60 Seconds)

Every year, 40 million EVs need service at franchised dealerships. The mechanics doing that work have a 67% chance of fixing the problem the first time — meaning one in three EV repairs requires a second visit. That costs dealers $800–1,200 per comeback in wasted labor, and it's getting worse as EV complexity grows faster than technician training.

We built X-Ray Vision Diagnostics: a spatial operating system that runs on a standard XR headset in the service bay. When a tech picks up an EV, the system shows them exactly where the fault is in 3D, reads the sensor data in real time, and walks them through OEM-prescribed repair steps — hands-free, eyes on the vehicle. We've raised FTFR from 67% to 91.8% on enrolled fault types across 73 real repairs at three Toyota and Ford dealerships.

We're raising $18M to go from 3 beta bays to 500 commercial bays, close a Toyota OEM embedded deal that puts us inside their dealer tooling contract, and expand to Canada and the UK.

---

## Market

### Total Addressable Market

**US franchised dealer service bays (automotive):** 80,000 bays across 17,000 franchised dealerships (NADA 2024)

At $3,500/bay/year: **US TAM = $280M/year**

**EV-serviceable bays (those with ≥ 20 EV repairs/month, our initial target):** ~35,000 bays

At $3,500/bay/year: **Serviceable addressable market (SAM) = $122M/year**

**Our Phase 5 target (500 US bays + 20 international):** $1.68M ARR = 1.4% of SAM

SAM penetration of 1.4% is not ambitious — it is conservative. The OEM embedded channel (Toyota alone has 1,500 US dealers × avg 4 serviceable bays = 6,000 bays) represents a 2× SAM expansion once embedded.

### Market Timing

Three converging forces make 2024–2026 the right window:

1. **EV fleet growth:** 40M EVs on US roads by 2025 (BloombergNEF); 90M by 2030. Every one of those vehicles will need dealer service.
2. **Technician talent gap:** NADA estimates 90,000 open technician positions in 2024; EVs require new skills most techs don't have yet. Dealers are hiring juniors who need support tools.
3. **OEM pressure on FTFR:** Toyota and Ford have publicly tied dealer CSI scores (and incentive payouts) to first-visit resolution rates. Service managers are now financially motivated to fix FTFR.

---

## Product

### What We've Built

- 5-agent AI architecture: DiagnosticCore (RAG + LLM), SensorFusion (OBD-II + thermal), CADRenderer (SLAM + 3D overlay), VoiceNLP (hands-free control), SafetyGuard (HV-STOP)
- OEM CAD data licensing: Toyota Motor North America, Ford Motor Company, Stellantis North America
- Safety certifications: OSHA 1910.147 audit passed; SAE J1742 certified
- 73 real-world repairs across 3 dealerships; 91.8% FTFR; 38% MTTD reduction
- End-to-end latency: 31ms p95 (well within 40ms target)

### What Makes It Defensible

**Data moat:** OEM CAD licensing takes 12–18 months to acquire. We have Toyota, Ford, and Stellantis locked up. Any competitor starts 12–18 months behind just on data.

**Safety certification moat:** SAE J1742 certification for our HV-STOP implementation took 18 months. A competitor must repeat this process. It cannot be shortcut.

**Network effect (early):** Every repair generates training data for DiagnosticCore (RLHF pipeline). More repairs = better model = higher FTFR = more dealer adoption. This flywheel starts turning at Phase 4 scale.

**OEM relationship moat:** Toyota and Ford regional training teams are actively recommending us to dealers. An OEM embedded deal converts this relationship into a structural distribution advantage.

---

## Traction

| Metric | Value |
|---|---|
| Beta bays | 3 |
| Repairs completed | 73 |
| FTFR improvement | 67% → 91.8% (all enrolled fault types) |
| MTTD improvement | 3.4h → 2.1h (38%) |
| Voice recognition rate | 97.6% |
| Technician confidence score | 4.1 / 5.0 |
| Safety incidents | 0 |
| OEM data agreements signed | 3 (Toyota, Ford, Stellantis) |
| Safety certifications | 2 (OSHA 1910.147, SAE J1742) |
| NPS from beta technicians | 67 |

---

## Business Model

**SaaS subscription, per bay, annual contract**

| Tier | Price | Features |
|---|---|---|
| Core | $2,400/bay/year | 3 fault types, 1 OEM |
| Professional | $3,500/bay/year | 5 fault types, 2 OEMs, dashboard, job card integration |
| Enterprise | $5,200/bay/year | All fault types, all OEMs, SLA, Apprentice Mode, recording |
| OEM Embedded (wholesale) | $2,400/bay/year | Sold through OEM dealer tooling contract |

Hardware sold separately: $5,200 (HoloLens 2 + edge server). After Year 1 warranty, dealer owns and insures hardware.

**Unit economics at steady state (hardware-sold model):**
- Gross margin per bay: 31% ($992/bay/year)
- CAC direct channel: $6,400 (6.5-year payback — acceptable only as a transition to embedded channel)
- CAC OEM embedded: $750 (2.4-year payback — the target state)
- LTV per bay: $12,400 (8% annual churn, 12.5-year lifetime)
- LTV:CAC (embedded): 16.5x

---

## Financial Projections

| Period | Bays | ARR | Gross Margin |
|---|---|---|---|
| Phase 4 end (Month 20) | 50 | $160K | Negative (investment phase) |
| Phase 5 mid (Month 25) | 275 | $880K | 12% (ramping) |
| Phase 5 end (Month 30) | 520 | $1.66M | 24% |
| Year 3 (OEM embedded active) | 1,500 | $4.8M | 35% |
| Year 4 | 3,000 | $9.6M | 42% |
| Year 5 | 5,000 | $16M | 48% |

Cash flow positive: projected Month 36–40 (dependent on OEM embedded deal timing).

---

## Use of Proceeds ($18M)

| Category | Amount | Purpose |
|---|---|---|
| Engineering + Product (18 months) | $7.2M | 8 additional engineers, 2 PMs — Phase 5 feature development and scale |
| Sales + CS | $3.6M | 6 AEs (geographic expansion), 5 CSMs (scale with bay count) |
| OEM BD and legal | $2.0M | Head of BD hire, OEM embedded negotiation legal fees, international legal entity setup |
| Hardware inventory | $1.8M | 300 HoloLens 2 + edge server units at cost (dealer-sold at $5,200 each) |
| SOC 2 + Security | $0.6M | SOC 2 audit, pentest, security engineer hire |
| International expansion | $0.8M | Canada and UK legal/compliance, localization, first sites |
| G&A + Operations | $2.0M | Finance, HR, office, infrastructure |
| **Total** | **$18M** | **~20 months runway at Phase 5 burn rate** |

---

## Team

**CEO:** [Name] — 10 years automotive software, former product lead at CDK Global; led $40M ARR product line
**CTO:** [Name] — 8 years spatial computing, former HoloLens SDK team at Microsoft; 3 patents in spatial rendering
**VP Engineering:** [Name] — 12 years engineering management, former Scale AI and autonomous vehicle startup
**Head of BD:** [Name] — 15 years OEM automotive partnerships; former Toyota Connected Services, Ford FordDirect
**Compliance Manager:** [Name] — Former OSHA compliance officer, 10 years automotive safety

**Advisory Board:**
- Former VP Service Operations, Toyota North America
- Former Chief Safety Officer, Ford Motor Company
- Partner, Foley & Lardner (automotive IP licensing)

---

## Ask

**Raising:** $18M Series A
**Pre-money valuation:** $42M (3× ARR forward; standard for SaaS with this growth profile and defensibility)
**Lead investor profile sought:** Industrial tech / automotive focused; has portfolio companies in dealer tech or OEM supply chain; can open doors to Toyota or Ford corporate venture arms
**Timeline:** Close by Month 16 (Phase 4 start); use proceeds to execute Phase 5

**Why now:** Phase 3 data is clean. FTFR improvement is statistically significant. OEM relationships are warm. The window to close an OEM embedded deal before a competitor replicates our data moat is 18–24 months. This raise funds that window.
