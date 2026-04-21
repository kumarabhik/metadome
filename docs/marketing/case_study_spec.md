# Case Study Publication Spec — Toyota/Ford Dealer Group
## Step 48 — Phase 4, Marketing 1

**Roadmap reference:** Phase 4, Marketing 1
**Status:** `[ ]` Not started
**Owner:** Product Marketing + VP Sales + Customer Success
**Target:** Case study published and in active sales use by Month 19
**Prerequisite:** n ≥ 200 repairs; FTFR ≥ 90% demonstrated (`docs/metrics/ftfr_90_achievement_spec.md`); dealer principal approval

---

## Objective

Publish a named, data-backed case study from a Phase 4 commercial customer that demonstrates FTFR improvement, MTTD reduction, and safety outcomes. A named case study is the single highest-value sales asset in B2B enterprise sales — dealer principals trust peer stories more than vendor claims.

**Priority customer for case study:** Toyota dealer group with highest FTFR improvement in Phase 4 data. Toyota is the preferred OEM because: (1) Toyota bZ4X was the initial enrolled vehicle; (2) Toyota OEM partnership is most mature; (3) Toyota's dealer network is the largest in the US and most recognizable for prospects.

**Ford case study (secondary):** If Ford Mustang Mach-E site achieves comparable results, a second case study published targeting Ford franchise dealers. Ford's EV growth trajectory (F-150 Lightning adoption) makes this a growing market segment.

---

## Case Study Structure

### Target Length and Format

| Format | Length | Use Case |
|---|---|---|
| PDF leave-behind | 2 pages | Direct sales meetings, dealer conference distribution |
| Web landing page | 800–1,000 words | SEO, inbound leads, press reference |
| Slide deck version | 5 slides | Conference presentations, investor updates |
| Video testimonial (optional) | 90 seconds | Trade show, website hero content |

### Section Structure (All Formats)

**1. The Customer (1 paragraph)**
- Dealer group name and OEM affiliation (if dealer consents to named reference)
- Number of service bays, technicians, annual service revenue (if shareable)
- The challenge: "Like most EV-certified dealers, [Dealer] struggled with EV diagnostic complexity — first-time fix rates on battery faults averaged 67%, and technicians spent 3+ hours per EV diagnosis."

**2. The Problem (1 paragraph)**
- Specific pain: EV TMS fault diagnosis complexity, technician uncertainty, comeback repairs
- Quantify the cost: "Each comeback repair costs approximately $450 in technician labor + parts re-handling. At [Dealer]'s volume, low FTFR represented $X,XXX per month in rework cost."

**3. Why X-Ray Vision Diagnostics (1 paragraph)**
- Decision factors: safety compliance, voice-primary hands-free interface, OEM-sourced repair data, edge processing (no cloud dependency during repair)
- Quote from service manager or dealer principal (requires written approval)

**4. The Results (core of case study)**
- FTFR before/after: exact percentage from data (e.g., "67% → 93% FTFR on EV battery faults — 26 percentage point improvement")
- MTTD before/after: exact hours (e.g., "3.4 hours → 1.7 hours mean time to diagnose — 50% faster")
- Safety record: zero HV safety incidents attributable to system error during deployment period
- Adoption: technician adoption rate and confidence score average
- Business impact: estimated annual rework cost avoided (calculated from FTFR improvement × average repair cost × volume)

**5. What Technicians Say**
- 1–2 direct quotes from enrolled technicians (written consent required)
- Focus on concrete, specific experience: not "great product" but "I used to spend two hours guessing which thermal sensor was reading wrong — now the system shows me exactly where the heat signature is off, and I confirm it in 20 minutes."

**6. The Road Ahead (1 paragraph)**
- Dealer plans to expand to additional bays / OEM lines
- Forward-looking quote from dealer principal or service director

---

## Required Data Points

All data points must be sourced from the analytics pipeline — no estimates or projections in case study. Each data point requires data provenance documentation (which system, which time period, sample size).

| Metric | Required Value | Source |
|---|---|---|
| Baseline FTFR (pre-deployment) | Site-specific historical data from DMS | CDK Drive / R&R historical query |
| Post-deployment FTFR | Point estimate + 95% CI from analytics pipeline | `docs/metrics/ftfr_90_achievement_spec.md` |
| Sample size (n) | ≥ 200 repairs at that specific site | Analytics pipeline |
| Baseline MTTD | Pre-deployment control group MTTD | `docs/metrics/mttd_control_group_spec.md` |
| Post-deployment MTTD | Point estimate from analytics pipeline | `docs/metrics/mttd_18hr_target_spec.md` |
| Safety incidents | Zero (or count if any — cannot suppress facts) | SafetyGuard audit log |
| Technician confidence score | Average composite score from post-repair survey | `docs/features/confidence_survey_spec.md` |
| Adoption rate | 30-day rolling DAU / licensed bays | `docs/metrics/platform_adoption_spec.md` |

**Data accuracy standard:** All numbers in case study reviewed and signed off by data analytics lead before publication. Any number that could embarrass the company or dealer if later contested must be conservative (use lower bound of CI, not point estimate).

---

## Approval Chain

**Step 1 — Internal data review** (Product + Analytics)
- Verify all metrics meet publication threshold
- Legal review: confirm no NDA provisions prevent publishing deployment metrics
- Compliance review: confirm OSHA and SAE compliance status is accurately represented

**Step 2 — Dealer approval**
- Share draft with dealer principal and service director
- Required approvals:
  - Named reference (dealer group name in case study) — dealer principal sign-off
  - Specific metric publication (FTFR %, MTTD, revenue impact) — dealer principal sign-off
  - Technician quotes — individual written consent per technician
  - Logo use — dealer group marketing team approval
- Approval turnaround target: 2 weeks after draft shared

**Step 3 — OEM approval (if applicable)**
- If case study references Toyota or Ford branding or OEM partnership: OEM communications team must approve language
- Toyota and Ford both have dealer marketing guidelines; case study must comply with co-branding rules
- Turnaround: 3 weeks for OEM review (per typical OEM comms SLA)

**Step 4 — Legal final review**
- Outside counsel reviews final draft for: accuracy of technical claims, regulatory compliance statements, limitation of liability language in footnotes
- Standard sign-off: 1 week

**Step 5 — Publication**
- PDF to sales team via Salesforce content library
- Web page live on marketing site
- LinkedIn announcement (product team + company page)
- Press release (optional — if results are strong enough to warrant media outreach)

---

## Business Impact Calculation — Template

Used in case study "Business Impact" section and in ROI calculator (`docs/pricing/roi_calculator_spec.md`).

```
Annual Rework Cost Saved =
  (Baseline FTFR% - Post FTFR%) × Monthly Repair Volume (enrolled faults)
  × Average Rework Cost per Repair × 12 months

Example:
  (0.67 - 0.93) × -1 = 0.26 (26% fewer comeback repairs)
  × 80 EV repairs/month
  × $450 rework cost per repair
  × 12 months
  = $112,320 annual rework cost avoided
```

Additional impact lines (include if data available):
- Bay throughput gain from MTTD reduction: (Baseline MTTD - New MTTD) × repairs/month × technician hourly rate
- Technician confidence score improvement → proxy for reduced error-related warranty claims

---

## Distribution Plan

| Channel | Timing | Audience |
|---|---|---|
| Sales team (Salesforce) | Day 1 of publication | All AEs for direct customer meetings |
| Website landing page | Day 1 | Inbound prospects, press, OEM partners |
| Email to dealer prospect list | Week 1 | Top 50 unconverted dealer prospects |
| NADA conference distribution | Month 20 (NADA annual) | Dealer principals at conference |
| LinkedIn post (company) | Week 1 | Industry followers, OEM partnerships, media |
| OEM partner sharing | Upon OEM approval | Toyota/Ford dealer network newsletters |
| Investor deck update | Next board meeting | Investors — validates North Star metric |

---

## What to Do If Dealer Declines Named Reference

If preferred case study partner declines to be named publicly:

1. Publish anonymized case study: "A multi-bay Toyota dealer in [Metro region]" — use regional descriptor, not name
2. Anonymized results are less impactful but acceptable for sales conversations
3. Simultaneously pursue second-choice dealer as named reference (next highest FTFR improvement site)
4. Named reference is required for NADA conference presentation — continue pursuing while using anonymized version in interim

---

## Links to Related Specs

- FTFR data source: `docs/metrics/ftfr_90_achievement_spec.md`
- MTTD data source: `docs/metrics/mttd_18hr_target_spec.md`
- Confidence score data: `docs/features/confidence_survey_spec.md`
- Adoption data: `docs/metrics/platform_adoption_spec.md`
- ROI calculator (case study data feeds into): `docs/pricing/roi_calculator_spec.md`
- Conference presentation (case study as centerpiece): `docs/marketing/conference_presentation_spec.md`
