# Subscription Model Spec
## Step 38 — Phase 4, Pricing 1

**Roadmap reference:** Phase 4, Pricing 1
**Status:** `[ ]` Not started
**Owner:** PM + Finance + Sales
**Target:** Pricing finalized by Month 14; first contracts signed at Phase 4 kickoff

---

## Objective

Define the commercial subscription model for X-Ray Vision Diagnostics at GA launch. Pricing must be simple enough for a service director to approve without a VP review, defensible against a 5-minute dealership ROI calculation, and structured to grow with deployment depth.

---

## Pricing Philosophy

**Guiding principle:** Anchor the price to the value created, not the cost to deliver. The system targets a 23-percentage-point FTFR lift (67% → 90%). At an average dealer with 200 EV repairs/year and a $450 average repair order value, each percentage point of FTFR improvement = $900/year recovered. The system should capture roughly 20% of that value created.

**Price to capture:** 20% of $900 × 23 pts × [bay count adjustment] ≈ $4,140/bay/year rough value anchor.

---

## Tier Structure

### Tier 1 — Core ($349/bay/month, annual contract)

**Includes:**
- Full diagnostic overlay for all enrolled fault types and vehicle models
- Voice command interface (English)
- HV-STOP safety system
- Edge server software license (hardware sold separately)
- Standard support: email + Slack channel, 48-hour response SLA
- Manager Dashboard (read-only metrics)

**Excludes:**
- Job card DMS integration
- Custom OEM vocabulary training
- Priority support

**Target buyer:** Independent service directors at single-point dealerships; first contract at a new site.

---

### Tier 2 — Professional ($499/bay/month, annual contract)

**Everything in Core, plus:**
- Job card integration (CDK + R&R)
- Technician confidence survey + analytics
- Manager Dashboard with alert rules + HV-STOP notifications
- Priority support: 8-hour response SLA, dedicated Slack channel with Engineering
- Quarterly business review (PM + data review call)

**Target buyer:** Multi-bay dealerships; beta graduates upgrading from Core.

---

### Tier 3 — Enterprise (custom pricing, multi-year contract)

**Everything in Professional, plus:**
- Multi-site deployment with centralized fleet management
- Custom OEM vocabulary training (proprietary repair procedures)
- API access (export session data to dealer's own BI tools)
- Named Customer Success Manager
- 4-hour support SLA
- Custom SLA with financial penalties for uptime violations
- ROI reporting delivered quarterly by PM team

**Target buyer:** Large dealer groups (10+ rooftops); OEM-level procurement.

---

## Pricing Summary Table

| Tier | Price | Annual Contract Value (5 bays) | Target |
|---|---|---|---|
| Core | $349/bay/month | $20,940/year | Single-point dealer |
| Professional | $499/bay/month | $29,940/year | Multi-bay; beta upgrade |
| Enterprise | Custom (~$599–799/bay/month) | $43K–57K/year | Dealer groups |

---

## Hardware Model

Hardware (Jetson AGX Orin, HoloLens 2, thermal camera) is sold separately — not bundled in subscription. Rationale: hardware CapEx is a one-time cost owned by the dealer; subscription is OpEx that maps to ongoing value. Separating them avoids the "what happens to my hardware if I cancel" objection.

**Hardware list price (per bay):**
- Jetson AGX Orin 64GB: $2,199
- 2× HoloLens 2: $6,500 ($3,250 each)
- FLIR Lepton 3.5 + mount: $499
- WiFi6E access point: $399
- Cabling + rack: $299
- **Total hardware per bay: ~$9,896**

**Dealer break-even (Core tier):** $9,896 hardware + $4,188/year software = payback in < 18 months if FTFR lift delivers even 50% of projected value.

---

## Contract Terms

| Term | Value |
|---|---|
| Minimum contract length | 12 months |
| Renewal | Auto-renews annually; 60-day cancellation notice |
| Payment | Annual upfront (5% discount) or monthly |
| Price escalation | CPI + 3% at each annual renewal |
| SLA credits | < 99.5% uptime = 5% monthly credit; < 99.0% = 10% credit |
| Data ownership | All session data owned by dealer; X-Ray Vision Diagnostics has license to use aggregate anonymized data for model improvement |

---

## Competitive Positioning

| Competitor | Price (est.) | Approach | Our Advantage |
|---|---|---|---|
| Snap-on ZEUS+ tablet | $12,000 one-time + $1,200/year | 2D scan tool; no spatial overlay | Spatial CAD overlay; voice-first; HV-STOP |
| Bosch ESI[tronic] | $3,500/year subscription | 2D diagnostics; no AR | Same as above |
| OEM factory scan tools | "Free" (bundled with dealer agreement) | Limited to one OEM; no overlay | Multi-OEM; hands-free; FTFR data |

**Pricing is positioned at a premium to Bosch but below Snap-on TCO** — and framed as paying for outcomes (FTFR), not tool access.

---

## Exit Criteria

- [ ] Pricing tiers approved by Finance and CEO
- [ ] Contract template drafted and reviewed by Legal
- [ ] Sales team trained on pricing and value narrative
- [ ] First 3 Phase 4 contracts signed at Tier 2 or above
- [ ] Revenue recognition policy confirmed with Finance (SaaS subscription = ratable over contract period)
