# FTFR Baseline Measurement Spec
## Step 31 — Phase 3, Metrics 1

**Roadmap reference:** Phase 3, Metrics 1
**Status:** `[ ]` Not started
**Owner:** PM + Data Analytics
**Target:** Baseline established by Month 10 (after 50 completed repairs)

---

## Objective

Establish a statistically valid First-Time Fix Rate (FTFR) baseline for beta technicians on enrolled fault types. This baseline is the single most important metric for Phase 3 — without it, there is no way to claim the product works.

**North Star:** FTFR lift from 67% (EV industry baseline) → ≥90%. Phase 3 must prove the trajectory is real with real dealership data.

---

## FTFR Definition

**First-Time Fix Rate** = % of repair orders where the vehicle did not return to the same dealership for the same fault within 30 days of repair close.

```
FTFR = (Total ROs closed) - (ROs with same-fault return within 30 days)
       ─────────────────────────────────────────────────────────────────
                         Total ROs closed
```

**Same fault** = same DTC family or same symptom category (defined below). A vehicle returning for an unrelated fault does not count against FTFR.

### DTC Family Groupings

| Fault Category | DTC Families | Return Counts As |
|---|---|---|
| EV coolant leak | P0A93, P0A94, P1DF1 | Same fault |
| EV battery cell | P0A80, P0A81, P0A7E | Same fault |
| Hybrid inverter | P0A94, P3000, P0A78 | Same fault |
| Timing chain | P0016, P0017, P0018, P0019 | Same fault |
| Brake system | C0031, C1201, brake symptom | Same fault |

---

## Data Sources

### Primary: Dealer DMS (CDK / R&R)
- Pull all ROs closed at beta sites on enrolled fault types during beta period
- Match VIN + DTC family; identify returns within 30-day window
- CDK: via Drive API `GET /repair-orders?vin={vin}&dateFrom={}&dateTo={}`
- R&R: via weekly XML export from service manager

### Secondary: Headset Session Log
- Every repair completed through X-Ray Vision Diagnostics generates a `session_close` event with VIN + fault type
- Cross-reference against DMS RO data to confirm system usage on each repair (avoid counting manual repairs in the FTFR numerator)

### Control Group
- Same 3 dealerships, same fault types, repaired WITHOUT the headset system (tech used standard scan tool)
- Identified by: RO closed at beta site on enrolled fault type, no matching headset session in log
- Provides pre/during-beta comparison within the same dealership environment

---

## Sample Size Requirements

For 80% statistical power to detect a 15-percentage-point FTFR lift (67% → 82%) at α = 0.05:

```
n = 2 × [(Z_α/2 + Z_β)² × p(1-p)] / (p1 - p0)²
  = 2 × [(1.96 + 0.84)² × 0.67×0.33] / (0.15)²
  ≈ 96 repairs per group (system vs. control)
```

**Phase 3 target:** 50 minimum repairs (for early signal); 96 for statistically significant result.
Phase 3 exit gate requires 50 repairs. Phase 4 GA claim requires 200+ repairs.

---

## Measurement Protocol

### Monthly Cadence
1. PM pulls DMS data from all 3 sites on the 1st of each month
2. Cross-reference with headset session log (system-assisted vs. control)
3. Compute FTFR for system group and control group separately
4. Check for 30-day return window (requires data to be ≥30 days old — no real-time FTFR)
5. Publish to Manager Dashboard and internal beta report

### Confounders to Track
| Confounder | How Tracked | Mitigation |
|---|---|---|
| Technician experience level | Years of experience collected at onboarding | Stratify analysis by experience bracket |
| Vehicle age/mileage | VIN + DMS records | Report FTFR by vehicle age bucket |
| Fault complexity | DiagnosticCore confidence score | Correlate confidence score with FTFR |
| Day-of-week / time pressure | DMS timestamp | Flag ROs closed in last 30 min of shift |

---

## Reporting Format

### Monthly Beta Report Table

```
| Month | System Repairs | System FTFR | Control Repairs | Control FTFR | Lift | Sig? |
|-------|---------------|-------------|-----------------|--------------|------|------|
| M9    | 12            | 75%         | 18              | 61%          | +14% | No (n<50) |
| M10   | 28            | 79%         | 22              | 64%          | +15% | Marginal |
| M11   | 51            | 83%         | 31              | 66%          | +17% | Yes (p<0.05) |
```

---

## Exit Criteria

- [ ] ≥ 50 system-assisted repairs logged (DMS-confirmed)
- [ ] FTFR calculated and reported for both system and control groups
- [ ] 30-day return window confirmed (data is ≥ 30 days old for all counted repairs)
- [ ] Analysis reviewed by PM and shared with beta site service managers
- [ ] Trending toward ≥90% FTFR target or documented root cause for gap
