# OEM Embedded Integration — Sold-Through Dealer Tooling

**Phase:** 5
**Roadmap Item:** OEM Integration: Begin negotiations to embed platform in OEM dealer tooling contracts (sold-through model)

---

## Objective

Transition X-Ray Vision Diagnostics from a direct-to-dealer sales model to an OEM-embedded model where the platform is bundled into the OEM's standard dealer tooling package — the same channel through which OEMs already distribute diagnostic scanners, lifts, and service software. This shifts customer acquisition cost to near-zero and creates a structural moat: once embedded in an OEM's dealer agreement, the product becomes part of mandatory dealer certification.

Target: At least one OEM embedded deal in active negotiation by end of Phase 5 (Month 30). Full embedded revenue from Months 30+.

---

## What "Sold-Through" Means

In the automotive dealer tooling market, OEMs maintain a "required tools" list that franchised dealers must purchase as a condition of their franchise agreement. Examples: Snap-on ZEUS scanner is required by several OEMs; Toyota TechStream software is mandatory for all Toyota dealers.

Under the sold-through model:
- The OEM licenses X-Ray Vision Diagnostics from us at a wholesale rate
- The OEM bundles it into their dealer tooling package (required or recommended tier)
- Dealers pay the OEM as part of their existing tooling invoice — no separate vendor relationship
- We receive a per-bay, per-year wholesale payment from the OEM; OEM marks it up to dealers
- Support is co-branded: "Powered by X-Ray Vision Diagnostics"

This model is standard practice: Toyota sells TechStream this way; Ford distributes FDRS (Ford Diagnostic and Repair Software) through the same channel.

---

## Target OEM Partners for First Embedded Deal

Priority order based on existing relationship depth and strategic fit:

### 1. Toyota (Primary Target)
- Strongest existing relationship — Toyota bZ4X was launch vehicle, Toyota dealers are Phases 2–4 beta sites
- Toyota's dealer tooling program ("Dealer Daily") is well-structured with defined onboarding for new tools
- FTFR data from Phase 4 gives us a proven ROI story to present to Toyota's Global Service Operations team
- Internal champion: Toyota regional technical training manager (established during Phase 3 beta)

### 2. Ford (Secondary Target)
- Ford Mustang Mach-E added in Phase 3; Ford dealers in beta since Phase 3
- Ford's dealer tooling is managed through FordDirect — separate business unit, longer sales cycle
- Ford's "Technician Certification Program" (FordTech) is the natural embedding point
- Risk: Ford is mid-transition of their own internal diagnostic platform (FNV4 architecture) — timing may conflict

### 3. Stellantis (Opportunistic)
- Phase 1 OEM partnership target — CAD licensing framework exists but signatures were pending
- Stellantis dealer network is large (Jeep, Ram, Dodge, Chrysler, Alfa Romeo) — high volume upside
- Lower EV battery fault volume than Toyota/Ford today; may require ICE fault expansion first

---

## Deal Structure

### Commercial Terms (Target)

| Term | Target | Floor |
|---|---|---|
| Wholesale per-bay per-year | $2,400 (vs. $3,500 direct) | $1,800 |
| Contract length | 3 years, auto-renew | 2 years |
| Volume commitment | 500 bays Year 1, 1,500 Year 2 | 300 bays Year 1 |
| Hardware bundling | OEM provides HoloLens 2 units (leverages their volume purchasing) | Optional |
| Revenue share on upsell | None — OEM keeps markup | None |
| Data rights | De-identified repair telemetry shared with OEM for vehicle quality program | OEM gets aggregate data only, never PII |

### Why the Discount is Justified
At $2,400 wholesale vs. $3,500 direct: we lose $1,100/bay/year in revenue per bay, but acquire customers at ~$0 CAC vs. estimated $1,800/bay direct CAC. Payback period on the discount: 1.6 years. At 1,500 bays in Year 2, embedded wholesale revenue ($3.6M) exceeds what direct sales could achieve in the same period given our current sales team capacity.

### IP and Control Terms (Non-Negotiable)
- We retain all IP; OEM receives a license only
- We control software updates and release cadence — OEM cannot block or delay safety updates
- HV-STOP protocol cannot be modified or disabled by OEM configuration
- If OEM terminates contract, dealers retain access through end of paid term; no forced cutoff
- Audit rights: we can audit OEM's bay count reporting quarterly

---

## Negotiation Playbook

### Phase 1: Internal Alignment (Months 22–23)
1. Board approval for sold-through model and floor pricing
2. Legal: draft OEM licensing agreement template (separate from existing CAD data licensing agreement)
3. Finance: model three scenarios (Toyota only, Toyota + Ford, all three) — revenue, margin, CAC impact
4. Product: define embedded vs. direct feature parity — embedded tier must not be inferior to direct Professional tier

### Phase 2: Toyota Approach (Months 23–25)
1. Request meeting with Toyota Global Service Operations VP through existing Phase 3/4 account relationship
2. Present deck: FTFR improvement data from Phase 4, ROI per bay, sold-through model structure
3. Propose 90-day exclusive negotiation window with Toyota before approaching Ford or Stellantis
4. Target: Letter of Intent by Month 25

### Phase 3: Term Sheet and Legal (Months 25–28)
1. Exchange term sheets — expect 3–5 rounds
2. Key negotiation points: data rights (Toyota will want vehicle quality data), support SLA (Toyota will want 4-hour response for dealer-facing issues), co-branding requirements
3. Legal review: antitrust counsel review (exclusive embedding in one OEM's required tools list may raise issues with competing dealers — structure as "preferred" not "exclusive mandatory")
4. Target: Executed agreement by Month 28

### Phase 4: Integration and Launch (Months 28–30)
1. Technical integration: embed license key provisioning into OEM dealer onboarding system (IT project — estimated 6 weeks)
2. Training: co-develop OEM-branded onboarding materials with Toyota training team
3. Soft launch: first 50 embedded bays (Toyota's highest-volume EV dealer group)
4. Target: First embedded invoice issued Month 30

---

## Internal Readiness Requirements

Before entering OEM negotiations, these must be in place:

| Requirement | Owner | Status |
|---|---|---|
| FTFR ≥ 90% demonstrated (Phase 4 exit criterion) | Product / Data | Phase 4 exit gate |
| OSHA + SAE J1742 certifications complete | Compliance | Phase 4 exit gate |
| SOC 2 Type II audit complete | Engineering / Security | Phase 5 parallel track |
| Multi-region cloud infrastructure live | Engineering | Phase 5 parallel track |
| Legal entity capable of multi-year OEM contract | Finance / Legal | Month 22 |

---

## Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| Toyota negotiation stalls past Month 28 | Medium | Begin Ford approach at Month 26 in parallel — use competitive pressure |
| OEM demands exclusivity (only their vehicles) | High | Counter: offer "preferred OEM" branding and first-look on new vehicle integrations; reject full exclusivity — it strands other OEM customers |
| OEM wants to white-label (remove our brand entirely) | Medium | Negotiate: co-branded "Powered by X-Ray Vision Diagnostics" minimum; full white-label only with 40% premium on wholesale rate |
| OEM demands source code escrow | Medium | Acceptable — standard in enterprise software; use standard escrow provider (Iron Mountain) |
| OEM insists on controlling update cadence | High | Non-negotiable on safety updates (HV-STOP); negotiate 30-day notice for feature updates |
| Deal falls through entirely | Low (given relationship depth) | Direct sales model continues; embedded is upside, not survival |
