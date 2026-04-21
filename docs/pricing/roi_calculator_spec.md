# ROI Calculator Spec — Dealer Principal Tool
## Step 39 — Phase 4, Pricing 2

**Roadmap reference:** Phase 4, Pricing 2
**Status:** `[ ]` Not started
**Owner:** PM + Sales + Frontend Engineering
**Target:** Live for sales team use by Month 15

---

## Objective

Build a web-based ROI calculator that lets a dealer principal (or their service director) enter their own numbers and see a personalized financial case for X-Ray Vision Diagnostics. The calculator must output a credible payback period and annual value estimate in under 2 minutes.

**Strategic purpose:** Dealer principals buy on financial ROI, not feature lists. The calculator bypasses the "sounds interesting, let me think about it" objection by making the ROI concrete and personalized before the salesperson leaves the room.

---

## User

**Primary:** Dealer principal or general manager (the person signing the check)
**Secondary:** Service director (provides the input numbers)
**Scenario:** Salesperson uses the calculator on a laptop or tablet during a 30-minute discovery meeting. The output is a leave-behind PDF.

---

## Inputs

| Input | Label in UI | Default | Notes |
|---|---|---|---|
| Number of service bays | "How many bays do you want to equip?" | 3 | Slider: 1–20 |
| Annual EV/hybrid repairs | "How many EV or hybrid repairs does your shop complete per year?" | 200 | Numeric input |
| Current FTFR (%) | "What % of EV/hybrid repairs are fixed correctly the first time?" | 67 | Slider: 40–95%; pre-filled with industry baseline |
| Average repair order value ($) | "What is your average EV repair order value?" | $450 | Numeric input |
| Average EV tech hourly rate ($) | "What is your average technician hourly rate (fully loaded)?" | $85 | Numeric input |
| Average diagnosis time (hours) | "How long does a typical EV diagnosis take today?" | 3.4 | Slider: 1–8h; pre-filled with industry baseline |
| Monthly subscription tier | "Which plan are you considering?" | Professional ($499/bay) | Dropdown |

---

## Calculation Logic

### Value from FTFR Improvement

```
Projected FTFR = min(current_ftfr + 23%, 95%)  # 23pp lift from design_doc.md
Repairs avoided (comebacks) = annual_repairs × (projected_ftfr - current_ftfr)
Value per avoided comeback = avg_ro_value × 0.5  # 50% margin on a rework is lost
Annual comeback savings = repairs_avoided × value_per_avoided_comeback
```

### Value from Diagnosis Time Reduction

```
Projected diagnosis time = current_diag_time × 0.73  # 27% reduction (MTTD target)
Time saved per repair (hours) = current_diag_time - projected_diag_time
Annual hours saved = annual_repairs × time_saved_per_repair
Annual labor value saved = annual_hours_saved × tech_hourly_rate
```

### Total Annual Value Created

```
Total annual value = comeback_savings + labor_value_saved
```

### Cost

```
Annual software cost = bay_count × monthly_price × 12
Hardware cost (one-time) = bay_count × 9896  # per bay hardware cost
```

### Payback Period

```
Year 1 net value = total_annual_value - annual_software_cost - hardware_cost
Year 2+ net value = total_annual_value - annual_software_cost
Payback period (months) = hardware_cost / (monthly_value - monthly_software_cost)
```

---

## Output Display

### Summary Card (large, visible from across a table)

```
┌─────────────────────────────────────────────────────────┐
│  Annual Value Created:          $47,200                 │
│  Annual System Cost:            $17,964                 │
│  Net Annual Benefit:            $29,236                 │
│  Hardware Payback:              11 months               │
│  5-Year ROI:                    412%                    │
└─────────────────────────────────────────────────────────┘
```

### Value Breakdown (two-bar chart)
- Bar 1: "Reduced comeback cost" (in dollars)
- Bar 2: "Technician time recovered" (in dollars)
- Total bar: combined

### Assumptions Panel (collapsed by default, expandable)
Lists all inputs + data sources:
- "23% FTFR lift based on beta program data (3 dealerships, 50+ repairs)"
- "27% diagnosis time reduction based on internal MTTD tracking"
- Industry baselines cited with source (J.D. Power 2023, NADA 2024)

### Leave-Behind PDF
Button: "Download My ROI Report"
- Generates PDF with dealer name, input values, output, and calculation methodology
- Includes X-Ray Vision Diagnostics contact info and next steps
- PM can track PDF downloads by dealership (analytics event on button click)

---

## Technical Implementation

- **Frontend:** React SPA, hosted on AWS S3 + CloudFront
- **No backend required:** all calculations done client-side in JavaScript
- **PDF generation:** jsPDF library (client-side; no server dependency)
- **Analytics:** PostHog event tracking (page view, calculate click, PDF download)
- **Auth:** None — public-facing tool; no login required
- **URL:** `app.xrayvision.io/roi` (marketing site subdomain)

---

## Validation

Before sales team use, validate calculator outputs against 3 real beta site scenarios:
- Input beta site actual data → confirm output matches observed beta results
- If calculator overstates value vs. actual data: adjust FTFR lift assumption downward before launch

**Key risk to avoid:** A calculator that promises more than the product delivers. Overpromise here destroys trust at renewal time.

---

## Exit Criteria

- [ ] Calculator live at public URL
- [ ] PDF export generates correctly on Chrome, Safari, Edge
- [ ] Outputs validated against 3 beta site actual results (within 15% of observed values)
- [ ] Sales team trained on how to use calculator in discovery meetings
- [ ] Analytics tracking confirmed (PDF downloads tracked by dealership)
