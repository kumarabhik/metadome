# Data Governance Policy

**Version:** 1.0 — Phase 5
**Owner:** Legal + Head of Engineering
**Review cadence:** Annually, or when entering a new market
**Applies to:** All data processed by X-Ray Vision Diagnostics across all deployments

---

## Data Classification

All data processed by the platform is classified into one of four tiers. Classification determines encryption requirements, access controls, retention, and cross-border transfer rules.

| Tier | Definition | Examples |
|---|---|---|
| **Restricted** | Data whose exposure creates legal liability, safety risk, or contractual breach | VIN (linked to individual), tech employee ID, video recordings (contain VIN + face-adjacent view), HV-STOP audit logs, OEM CAD files |
| **Confidential** | Business-sensitive data not for public disclosure | FTFR data by dealer, pricing details, OEM contract terms, session telemetry with dealer ID |
| **Internal** | Operational data accessible to all employees; not for external disclosure | Aggregate platform metrics, de-identified fleet baseline data, system logs without PII |
| **Public** | Approved for external disclosure | Published case study data, marketing materials, API documentation |

---

## Data Inventory

| Data Type | Classification | Created By | Stored Where | Contains PII? | Encrypted at Rest? | Encrypted in Transit? |
|---|---|---|---|---|---|---|
| OEM CAD files | Restricted | OEM (licensed) | S3 (regional) | No | Yes — AES-256 SSE-KMS | Yes — TLS 1.3 |
| VIN-indexed repair session | Restricted | Edge server | S3 (regional) | Yes (VIN) | Yes | Yes |
| Tech employee ID ↔ headset mapping | Restricted | Manager dashboard | Aurora PostgreSQL | Yes | Yes | Yes |
| Video recordings | Restricted | Edge server | S3 (regional) | Yes (VIN + context) | Yes | Yes |
| HV-STOP audit logs | Restricted | SafetyGuard (edge) | S3 + local SSD | Yes (tech_id) | Yes | Yes |
| Voice command transcripts | Restricted | VoiceNLP (edge) | S3 (regional, 90 days) | Potentially (names in audio) | Yes | Yes |
| OEM telematics sensor data | Confidential | OEM (inbound) | S3 (regional) + Timescale DB | Yes (VIN) | Yes | Yes |
| Session telemetry events | Confidential | Edge server → Kinesis | S3 (regional) | Yes (dealer_id, tech_id) | Yes | Yes |
| Manager dashboard data | Confidential | Dashboard backend | Aurora PostgreSQL | Yes (tech names, dealer names) | Yes | Yes |
| De-identified fleet baseline | Internal | Data pipeline | S3 us-east-1 (global) | No | Yes | Yes |
| System and application logs | Internal | All services | CloudWatch + S3 | Potentially (IP addresses) | Yes | Yes |
| Aggregate FTFR metrics | Internal | Analytics pipeline | S3 + Athena | No | Yes | Yes |
| Published case study data | Public | Marketing | Website | No | N/A | N/A |

---

## Data Residency

| Market | Restriction | Implementation |
|---|---|---|
| United States | No federal data residency law; CCPA requires residency for CA residents if applicable | All US data in us-east-1 (primary) + us-west-2 (HA failover) |
| Canada | PIPEDA (federal) + Quebec Law 25 — personal data must remain in Canada | All Canadian dealer data in ca-central-1 only; no replication to US |
| United Kingdom | UK GDPR — data must not leave UK without adequate safeguards | All UK dealer data in eu-west-2 only; no replication outside EU/UK |

**Cross-border data transfers permitted:**
- De-identified fleet baseline data (no VIN, no PII) → replicated globally for predictive maintenance model
- OEM CAD files → replicated to relevant regional buckets (not PII)
- Aggregate platform metrics → us-east-1 for global reporting (no PII)

**Cross-border transfers prohibited:**
- Any data containing VIN, tech_id, dealer_id, or video recordings across market boundaries

---

## Retention Schedules

| Data Type | Standard Retention | Extended Retention Trigger | Max Retention |
|---|---|---|---|
| Session repair events (with VIN) | 36 months | Active warranty dispute | Dispute close + 2 years |
| HV-STOP audit logs | 36 months (OSHA 1910.147 minimum) | Safety incident investigation | 7 years |
| Video recordings | 90 days | Manager extends, active dispute, or OEM certification | 5 years (OEM cert), 2 years (dispute) |
| Voice command transcripts | 90 days | None — fixed | 90 days |
| OEM telematics sensor snapshots | 36 months per VIN | Predictive model training | 36 months |
| Tech employee ID mappings | Duration of account + 90 days | Legal hold | Legal hold duration |
| Manager dashboard data | Duration of account + 1 year | Legal hold | Legal hold duration |
| Billing records | 7 years (tax and accounting requirement) | N/A | 7 years |
| OEM CAD files | Duration of OEM license agreement | N/A | License duration + 30 days (deletion) |

**Automated enforcement:** All retention periods are enforced by S3 Lifecycle policies. Policies are reviewed quarterly by the Data Engineer and approved by Legal.

---

## Access Control

**Principle of least privilege:** All access to Restricted and Confidential data requires justification, is role-based, and is reviewed quarterly.

| Role | Restricted Data Access | Confidential Data Access |
|---|---|---|
| Dealer Principal | Own dealer's data only | Own dealer's data only |
| Service Manager | Own dealer's data only | Own dealer's data only |
| Technician | Own session data only | None |
| CSM (X-Ray Vision team) | Read-only, assigned accounts only | All assigned accounts |
| Engineering (production) | Break-glass access only (time-limited, logged) | Read for debugging; write for incident response |
| ML/Data team | De-identified data only (no VIN, no tech_id) | Aggregate data only |
| CEO/Executives | Read-only (aggregate), no PII | All |
| Auditors (external) | Audit scope only, time-limited token | Audit scope only |

**Break-glass access:** Any engineer accessing production PII data outside their normal role requires:
1. Ticket in Jira with business justification
2. VP Engineering approval
3. Access granted via time-limited IAM role (maximum 4-hour window)
4. All access logged to CloudTrail; reviewed by Security Engineer weekly

---

## Data Subject Rights (CCPA / UK GDPR / PIPEDA)

Technicians and other individuals whose data we process have rights under applicable law.

| Right | Scope | Process |
|---|---|---|
| Right to access | Tech can request all data we hold about them | Tech submits request via manager dashboard or email; fulfilled within 30 days |
| Right to deletion | Tech can request deletion of their personal data | Fulfilled within 30 days; exceptions: data subject to legal hold or OSHA retention requirement (notified in response) |
| Right to correction | Tech can request correction of inaccurate data | Fulfilled within 15 days |
| Right to data portability | Tech can request their data in machine-readable format (JSON) | Fulfilled within 30 days |
| Right to opt out of sale | We do not sell personal data | N/A — confirm in privacy policy |

**Data subject requests** are routed to `privacy@xvd.com` and tracked in a requests register maintained by Legal. All requests logged and responses documented for compliance audit.

---

## Data Breach Response

**Definition of breach:** Unauthorized access, disclosure, or loss of Restricted or Confidential data.

**Response timeline:**

| Hour | Action |
|---|---|
| 0–1 | Detect breach; Engineering Lead + Security Engineer + CEO notified |
| 1–4 | Contain: revoke affected access credentials; isolate affected systems |
| 4–24 | Assess scope: what data was exposed? How many individuals? |
| 24–48 | Initial notification: affected dealers notified with known facts |
| 72 | UK ICO notification required (UK GDPR) if ≥ 1 UK resident affected |
| 30 days | Full incident report issued; root cause and remediation documented |
| 30 days | CCPA notification to CA residents if applicable |

**OSHA breach (HV-STOP audit log compromise):** Treat as P0 regardless of scope — audit log integrity is a compliance requirement. Notify OSHA within 48 hours if log chain of custody is broken.

---

## Third-Party Data Processors

| Processor | Data Shared | Legal Basis | DPA Signed? |
|---|---|---|---|
| AWS | All data (IaaS) | Contractual necessity | Yes (AWS DPA) |
| Segment | Session events (de-identified) | Legitimate interests | Yes |
| Amplitude | Product analytics (de-identified, no PII) | Legitimate interests | Yes |
| Salesforce | CS health scores, dealer contact info | Contractual necessity | Yes |
| OpenAI | DiagnosticCore RAG embeddings (no PII, no VIN in embedding inputs) | Contractual necessity | Yes (OpenAI DPA) |
| PagerDuty | Alert routing (no PII in alerts by policy) | Legitimate interests | Yes |
| Microsoft (HoloLens 2) | Device telemetry (battery, temp) — controlled by MS privacy policy | Product feature | MS EULA |

All third-party processors are reviewed annually. New processors require Legal approval before data is shared.
