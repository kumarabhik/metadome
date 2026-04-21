# Unit Economics Model

**Covers:** Phases 1–5 (Months 1–30)
**Last updated:** Phase 5 spec complete

---

## Revenue Model

### Pricing Tiers (from `docs/pricing/subscription_model_spec.md`)

| Tier | Price / Bay / Year | Target Customer |
|---|---|---|
| Core | $2,400 | Independent dealers, low EV volume |
| Professional | $3,500 | Mid-size franchised dealers (1–5 bays) |
| Enterprise | $5,200 | Large dealer groups (5+ bays, custom SLA) |
| OEM Embedded (wholesale) | $2,400 | Sold through OEM dealer tooling channel |

**Assumed blended ASP:** $3,200/bay/year (mix: 30% Core, 50% Professional, 20% Enterprise)

---

## Bay Count Ramp by Phase

| Phase | Period | Bays (End of Period) | Cumulative ARR |
|---|---|---|---|
| Phase 0 | Months 0 | 0 | $0 |
| Phase 1 | Months 1–4 | 0 (internal only) | $0 |
| Phase 2 | Months 5–8 | 0 (internal test bay) | $0 |
| Phase 3 Beta | Months 9–14 | 3 (non-paying beta) | $0 |
| Phase 4 GA | Months 15–20 | 50 (all paying) | $160,000 |
| Phase 5 Scale | Months 21–30 | 500 (US) + 20 (CA+UK) | $1,664,000 |

Note: All Phase 3 bays are non-paying beta. Revenue begins at Phase 4 GA launch.

---

## ARR Model (Phase 4 → Phase 5)

### Phase 4 (50 bays, Month 15–20)

Ramp assumption: bays go live over 6 months (not all on Day 1):
- Month 15: 5 bays live → $16,000 ARR
- Month 16: 15 bays → $48,000 ARR
- Month 17: 25 bays → $80,000 ARR
- Month 18: 35 bays → $112,000 ARR
- Month 19: 45 bays → $144,000 ARR
- Month 20: 50 bays → $160,000 ARR

**Phase 4 cumulative recognized revenue (Months 15–20):** ~$560,000

### Phase 5 (500+ bays, Months 21–30)

Ramp: 50 bays at start of Phase 5 → add ~45 bays/month → 500 bays by Month 30.

| Month | Bays | Monthly Revenue |
|---|---|---|
| 21 | 95 | $25,333 |
| 22 | 140 | $37,333 |
| 23 | 185 | $49,333 |
| 24 | 230 | $61,333 |
| 25 | 275 | $73,333 |
| 26 | 320 | $85,333 |
| 27 | 365 | $97,333 |
| 28 | 410 | $109,333 |
| 29 | 455 | $121,333 |
| 30 | 520 | $138,667 |

**Phase 5 cumulative recognized revenue:** ~$798,000
**ARR at Month 30 (520 bays × $3,200):** $1,664,000

---

## Cost Model

### Cost of Revenue (COR)

| Item | Phase 4 (50 bays) | Phase 5 (500 bays) |
|---|---|---|
| Cloud infrastructure | $12,000/year | $98,000/year |
| Edge server amortization (3-year life) | $50,000/year (50 × $3,000/3) | $500,000/year (500 × $3,000/3) |
| HoloLens 2 amortization (3-year life) | $58,333/year (50 × $3,500/3) | $583,333/year (500 × $3,500/3) |
| OEM data licensing (3 OEMs) | $450,000/year | $450,000/year |
| Field install labor ($1,200/bay, one-time amortized) | $20,000/year | $200,000/year |
| Support (1 CS engineer per 50 bays) | $80,000/year (1 FTE) | $800,000/year (10 FTE) |
| **Total COR** | **$670,333/year** | **$2,631,333/year** |

**Gross margin at Phase 4:** Revenue $160K, COR $670K → negative (investment phase)
**Gross margin at Phase 5 (500 bays):** Revenue $1.664M, COR $2.631M → still negative on COR alone

**Critical note:** The OEM data licensing cost ($450K/year) is largely fixed regardless of bay count. At 500 bays, it represents 17% of revenue. At 2,000 bays it drops to ~4%. This is the key scale lever — OEM data licensing is the primary fixed cost to grow through.

### When Does the Unit Become Profitable?

Excluding OEM data licensing (fixed cost), variable COR per bay:
- Cloud: $196/year
- Edge server amortization: $1,000/year
- HoloLens 2 amortization: $1,167/year
- Install labor amortized: $400/year
- Support: $1,600/year (1 CS FTE per 50 bays)
- **Variable COR per bay: $4,363/year**

At blended ASP $3,200: **negative unit economics per bay in isolation.**

Resolution: hardware must be sold, not provided. Under the hardware-sold model (see pricing spec):
- Dealer buys hardware outright ($5,200 for headset + edge server)
- Software subscription only: $3,200/bay/year
- Variable COR without hardware: $196 + $400 + $1,600 = **$2,196/year**
- **Gross margin per bay (hardware-sold model): $1,004/year or 31%**

Target: reach 31% gross margin by Month 24 (when hardware-sold model is default for new customers).

---

## Customer Acquisition Cost (CAC)

### Direct Sales (Phases 3–4)

- Sales team: 2 AEs at $120K OTE each = $240K/year
- Marketing: $80K/year (conference, content, digital)
- Total sales + marketing: $320K/year
- Deals closed per year (Phase 4): 10 dealer groups × avg 5 bays = 50 bays/year
- **CAC per bay (direct): $6,400**

At $3,200 ASP and 31% gross margin = $992 gross profit per bay per year. Payback on CAC: **6.5 years.**

This is unacceptably long for direct sales. It confirms the OEM embedded channel is not optional — it is survival.

### OEM Embedded Channel (Phase 5)

- OEM channel cost: $150K/year OEM account management (1 FTE)
- Bays acquired via OEM channel (Phase 5 target): 200 bays/year
- **CAC per bay (OEM embedded): $750**

Payback at 31% gross margin: **2.4 years.** Acceptable for B2B SaaS with 3-year contracts.

**Strategic conclusion:** Phase 5 OEM embedded deal is the single highest-leverage action in the company's history. It transforms unit economics from unviable (6.5-year payback) to healthy (2.4-year payback).

---

## LTV Model

**Assumptions:**
- Annual churn: 8% (B2B SaaS average; automotive dealer tech churn is lower due to high switching cost)
- Average customer lifetime: 12.5 years (1 / 8% churn)
- Annual gross margin per bay: $992 (31% of $3,200)
- LTV per bay: $992 × 12.5 = **$12,400**

**LTV:CAC ratios:**
- Direct channel: $12,400 / $6,400 = **1.9x** (below healthy 3x threshold)
- OEM embedded: $12,400 / $750 = **16.5x** (excellent)

---

## Path to Profitability

| Milestone | Bay Count | Monthly Burn Status |
|---|---|---|
| Phase 3 beta end | 3 | Pre-revenue; R&D spend only |
| Phase 4 GA | 50 | Revenue starts; COR > revenue |
| Hardware-sold model live | 100 | COR starts to normalize |
| OEM embedded bays = 50% of fleet | 300 | LTV:CAC turns healthy |
| OEM data licensing < 15% of revenue | ~450 | Gross margin crosses 40% |
| Cash flow positive (operating) | ~600–700 | Estimated Month 36–40 |

---

## Key Financial Risks

| Risk | Impact | Mitigation |
|---|---|---|
| OEM data licensing cost doesn't decrease with volume | Gross margin ceiling | Renegotiate to per-VIN-activated royalty at Phase 3 renewal; reduces fixed cost |
| Hardware churn (headset damage / loss at dealer sites) | Unexpected COR spike | Require dealers to insure hardware; replacement at dealer cost after 1-year warranty |
| OEM embedded deal fails | CAC stays at 6.5x payback | Continue direct sales; reduce AE headcount; extend runway by cutting non-essential R&D |
| Dealer churn exceeds 8% | LTV deteriorates | Monitor NPS ≥ 50 target as early warning; CS intervention at NPS < 40 |
