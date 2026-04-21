# Go-To-Market Strategy

**Product:** X-Ray Vision Diagnostics
**Covers:** Phase 3 beta through Phase 5 scale

---

## GTM Thesis

X-Ray Vision Diagnostics is sold to service managers and dealer principals at franchised automotive dealerships, and enabled by OEM technical training teams. It is not sold to technicians — they are users, not buyers. The decision chain is:

**Dealer principal** (approves budget) ← influenced by → **service manager** (daily user of FTFR/revenue data) ← validated by → **lead technician** (early adopter within the shop)

The GTM strategy works backwards: win the lead technician first (they become peer champions), then the service manager sees the FTFR data, then the dealer principal approves the renewal and expansion.

---

## Channel Strategy by Phase

### Phase 3: Closed Beta (Months 9–14) — Relationship Channel

**Channel:** Direct, curated. 3 dealerships hand-selected based on:
1. Existing OEM relationship depth (already cooperating on data access)
2. Service manager with ≥ 5 years tenure (stable relationship, won't churn mid-beta)
3. High EV repair volume (Toyota bZ4X or Ford Mach-E in top 10 repairs by volume)
4. Geographically accessible for field team (Austin TX and Dallas TX cluster)

**Objective:** Generate FTFR data and a referenceable customer. Not revenue.

**No sales motion at this stage** — beta sites are selected, not sold. They receive the product free in exchange for participation, data sharing, and reference availability.

---

### Phase 4: GA Launch (Months 15–20) — Direct Sales + OEM-Assisted

**Primary channel: OEM-Assisted Direct**
The fastest path to 50 bays is through the OEM regional training teams. Each OEM has a regional service training manager who visits dealers quarterly and recommends tools. Getting X-Ray Vision Diagnostics on their recommended list is worth more than any outbound sales effort.

Mechanism:
1. Present FTFR data from Phase 3 beta to Toyota and Ford regional training managers (Month 13)
2. They include X-Ray Vision Diagnostics in their "EV readiness toolkit" recommendation for dealers in their region
3. Interested dealers contact us directly → direct sales close

**Secondary channel: Conference + Case Study**
- NADA Dealer Conference (Month 18): 20-minute Innovation Stage session with FTFR case study
- Published case study from Phase 3 beta dealer (Toyota or Ford group) — one page, data-driven
- Website: ROI calculator tool (from `docs/pricing/roi_calculator_spec.md`)

**Sales team:** 2 Account Executives at Phase 4 start. Each AE quota: 25 bays/year. Combined: 50 bays by Month 20.

**AE target profile:**
- 5+ years B2B SaaS sales in automotive dealer tech (CDK, Reynolds & Reynolds, DealerSocket)
- Existing relationships with service managers or dealer principals at Toyota/Ford groups
- Comfortable with 90-day sales cycles and multi-stakeholder deals

**Sales cycle:**
- First meeting: service manager (demo + FTFR data)
- Second meeting: dealer principal (ROI calculator presentation)
- Pilot proposal: 1-bay, 30-day pilot with success criteria pre-defined
- Pilot → expansion: if FTFR improves ≥ 15% in pilot, dealer expands to all qualifying bays

---

### Phase 5: Scale (Months 21–30) — OEM Embedded + Referral + Geographic Expansion

**Primary channel (Phase 5): OEM Embedded**
See `docs/scale/oem_embedded_integration_spec.md`. Once one OEM embeds X-Ray Vision Diagnostics in their dealer tooling contract, customer acquisition through that OEM's dealer network is automatic. Target: Toyota embedded deal active by Month 28.

**Secondary channel: Referral / Dealer Group Expansion**
At 500 bays, word-of-mouth within dealer groups is meaningful. Large dealer groups (Penske, AutoNation, Sonic) operate 50–200 dealerships. Winning one site in a large group → pathway to the entire group.

Referral mechanics:
- "Dealer group expansion discount": 15% discount per bay for adding a second (or third) location in the same dealer group
- Peer champion network: top-performing techs (Apprentice Mode graduates with A-grade scores) are invited to be "X-Ray Vision Diagnostics Certified Trainers" — creates internal advocacy

**Tertiary channel: Geographic Expansion Sales**
Phase 4 is concentrated in Texas. Phase 5 expands geographically:
- Wave 1 (Months 21–24): California (Los Angeles, Bay Area) — highest EV density in US
- Wave 2 (Months 24–27): Florida + Southeast — large Toyota and Ford dealer populations
- Wave 3 (Months 27–30): Northeast + Midwest — lower EV density but high repair volume

Geographic hiring: add 2 AEs per wave (total 8 AEs by Month 30).

---

## Messaging Framework

### For Dealer Principals (Economic Buyer)
**Problem:** EV repairs have a 67% First-Time Fix Rate. Every comeback costs $800–1,200 in re-diagnosis labor and threatens CSI (Customer Satisfaction Index) scores that affect OEM incentive payouts.
**Solution:** X-Ray Vision Diagnostics raises FTFR to ≥ 90% on enrolled EV fault types.
**Proof:** [Dealer group name] saw FTFR improve from 64% to 92% on battery TMS repairs in 60 days. That's $[X] in recovered revenue per year.
**Ask:** 30-day pilot. One bay. Pre-defined success criteria. No risk.

### For Service Managers (Champion / Influencer)
**Problem:** Techs spend 45 minutes on average diagnosing an EV fault. The scanner gives a starting point, not an answer. Glancing at the tablet while the vehicle is live creates mistakes.
**Solution:** Hands-free, eyes-on-vehicle diagnosis. The overlay shows exactly what to look at, with OEM-sourced steps, in real time.
**Proof:** Average diagnosis time drops from 45 minutes to 22 minutes on enrolled fault types (Phase 3 data).
**Ask:** Let your best tech try it on the next bZ4X or Mach-E that comes in.

### For Technicians (User / Peer Champion)
**Problem:** EV diagnostics is different from ICE. The failure modes are less visible. The risk of getting it wrong is higher (HV exposure).
**Solution:** The overlay shows the fault location in 3D, the HV exclusion zone in real space, and the step-by-step procedure — all without looking away from the vehicle.
**Proof:** "I used to triple-check the procedure before touching the HV system. Now I trust what I see." — T01, Toyota Master Tech.

---

## Launch Sequence

| Month | GTM Action |
|---|---|
| 13 | Brief Toyota and Ford regional training managers on Phase 3 FTFR data |
| 14 | Beta dealer case study approved for publication |
| 15 | GA launch: pricing live, sales team hired |
| 16 | OEM training managers begin recommending to their dealer networks |
| 17 | ROI calculator on website |
| 18 | NADA Dealer Conference — 20-minute session |
| 19 | First 25 bays live and billing |
| 20 | 50 bays live; begin Phase 5 OEM embedded approach |
| 22 | California launch: 2 new AEs, Toyota/Ford dealer outreach begins |
| 24 | Florida launch |
| 26 | Toyota OEM embedded: Letter of Intent target |
| 28 | Toyota OEM embedded: executed agreement target |
| 30 | 500+ bays live; OEM embedded deal in billing |

---

## Competitive Positioning

| Competitor | Their Approach | Our Differentiation |
|---|---|---|
| Snap-on / Solus | Handheld scanner, 2D display, OBD-II only | Spatial overlay; hands-free; HV proximity visualization |
| Mitchell1 / ALLDATA | Desktop software, repair manual database | In-headset, in-bay; no desktop/tablet lookup required |
| Bosch ESI[tronic] | Tablet-based diagnostics | Same as Mitchell1 — 2D tablet vs. 3D spatial |
| OEM proprietary tools (Toyota TechStream, Ford FDRS) | OEM-specific, desktop-only | Multi-OEM; spatial; AI-assisted disambiguation |
| No competitor has spatial overlay + HV safety integration | | This is the moat |

**Moat:** OEM CAD data licensing takes 12–18 months to acquire. Any competitor starting today faces the same timeline. We are 2 years ahead on data relationships.
