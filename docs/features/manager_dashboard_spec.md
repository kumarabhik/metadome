# Manager Dashboard Spec
## Step 26 — Phase 3, Feature 2

**Roadmap reference:** Phase 3, Feature 2
**Status:** `[ ]` Not started
**Owner:** PM + Frontend Engineering
**Target:** Live at beta deployment kickoff, Month 9

---

## Objective

Give service managers visibility into bay utilization, technician performance, safety events, and platform adoption — without requiring them to wear a headset. The dashboard is a browser-based web app, accessible from the manager's office PC or tablet.

**Key PM principle:** The manager dashboard must show outcomes (FTFR, repair time), not just activity (logins, sessions). Metrics that show "how many times it was used" without linking to business results will be ignored.

---

## Metrics Displayed

### 1. First-Time Fix Rate (FTFR) — North Star

| Display | Description |
|---|---|
| Current period FTFR | % of repairs where vehicle did not return for same fault within 30 days |
| FTFR trend | 7-day rolling line chart |
| FTFR by fault type | Bar chart: coolant, cell fault, timing chain, inverter, brake |
| FTFR by technician (anonymized) | Ranked list: Tech A, Tech B, etc. (no real names) |
| FTFR vs. pre-platform baseline | Comparison to dealership's historical FTFR (entered manually at onboarding) |

### 2. Bay Utilization

| Display | Description |
|---|---|
| Active bays now | Live count of bays with headset active |
| Session count today | Total sessions initiated |
| Avg session duration | Minutes per repair, by fault type |
| Bay idle time | Hours per bay per day not in use (headset charged, no session) |

### 3. HV-STOP Events

| Display | Description |
|---|---|
| HV-STOP events today | Count + list with timestamp, tech ID, fault type |
| False positive rate | Events acknowledged as non-hazardous by tech (tech pressed "clear" quickly) |
| Avg response time | Time from HV-STOP trigger to tech acknowledgement |
| Monthly trend | Line chart: total events per day over 30 days |

> **Safety note:** HV-STOP events are never filtered or hidden. They are surfaced prominently. Managers receive a push notification (SMS or email) for any HV-STOP event within 30 minutes.

### 4. Platform Adoption

| Display | Description |
|---|---|
| DAU / licensed bays | % of bays with at least one session today |
| Weekly active bays | 7-day rolling |
| Technician adoption rate | % of enrolled technicians who initiated ≥1 session this week |
| Voice command success rate | % of voice commands resolved vs. "not understood" |

### 5. Technician Confidence Score

| Display | Description |
|---|---|
| Rolling 7-day average | Composite score per `docs/features/confidence_survey_spec.md` |
| Score by fault type | Which repairs generate most/least confidence |
| Trend line | 30-day chart |
| Alert badge | Red badge if any fault type < 3.0 for 3 consecutive sessions |

---

## Technical Architecture

### Data Flow

```
Edge Server (per bay)
  └─ Events: session_start, session_end, hv_stop, survey_response, voice_command
      └─ Sync to Cloud Analytics DB (PostgreSQL on AWS RDS)
          └─ REST API (FastAPI)
              └─ Dashboard Web App (React + Recharts)
```

### Sync Cadence
- HV-STOP events: pushed immediately (< 5s) via WebSocket push
- Session data: synced at session close
- Survey data: synced at session close
- Bay utilization: polled every 60 seconds

### Auth
- Manager login: email + password; role = `manager`
- Admin login (PM/QA): role = `admin` — sees all dealerships
- Technician view (future): role = `technician` — sees own history only
- JWT tokens; 8-hour session expiry

### Hosting
- Cloud: AWS (us-east-1 for beta)
- DB: PostgreSQL RDS (single-AZ for beta; multi-AZ for GA)
- Dashboard: React SPA hosted on S3 + CloudFront

---

## UI Layout

### Header Bar
```
[X-Ray Vision Diagnostics]   [Bay 1: Active] [Bay 2: Idle] [Bay 3: Active]   [Manager: John K.] [Logout]
```

### Main Grid (3-column)
```
| FTFR Today: 84%    | HV-STOP Events: 2  | Sessions Today: 11  |
| vs baseline: +17%  | Last: 14:23 Bay 2  | Avg duration: 52min |

[FTFR Trend — 7 day line chart]        [Bay Utilization — bar chart]

[Technician Confidence — 30 day]       [Voice Command Success Rate]

[HV-STOP Event Log — table]
```

---

## Alert Rules

| Alert | Trigger | Channel |
|---|---|---|
| HV-STOP event | Any event | SMS + in-dashboard banner |
| Confidence score drop | Any fault type < 3.0 for 3 sessions | Email + dashboard badge |
| Voice success rate < 92% | Rolling 24h | Email |
| Bay offline > 4 hours during business hours | Edge server unreachable | Email + dashboard alert |

---

## Exit Criteria

- [ ] Dashboard loads in < 2 seconds on LTE connection
- [ ] HV-STOP notification delivered within 30 seconds of event
- [ ] FTFR calculation verified against manual count on 20 historical sessions
- [ ] Manager can filter by date range, fault type, and technician
- [ ] No PII (technician real names) exposed in any view
