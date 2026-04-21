# SOC 2 Type II Certification

**Phase:** 5
**Roadmap Item:** Infrastructure: SOC 2 Type II certification for customer data

---

## Objective

Obtain SOC 2 Type II certification for the X-Ray Vision Diagnostics platform. SOC 2 Type II is the standard enterprise security certification required by large dealer groups and OEMs before signing multi-year contracts. Without it, enterprise deals stall at procurement review.

SOC 2 Type I proves controls exist at a point in time. SOC 2 Type II proves those controls operated effectively over an observation period (typically 6–12 months). We target the full Type II.

Target: SOC 2 Type II audit completed and report issued by Month 30.

---

## Trust Service Criteria in Scope

| Criteria | In Scope | Rationale |
|---|---|---|
| Security (CC) | Yes | Mandatory for all SOC 2 reports |
| Availability (A) | Yes | Dealers depend on uptime for active repairs — SLA commitments require availability controls |
| Confidentiality (C) | Yes | OEM CAD data, repair telemetry, and technician PII are confidential |
| Processing Integrity (PI) | No | Out of scope for Phase 5 — financial transaction processing not in product |
| Privacy (P) | No | Out of scope — covered separately by PIPEDA/UK GDPR compliance programs |

---

## Audit Observation Period

- Observation period: Months 22–28 (6 months)
- Auditor engaged: Month 21 (readiness assessment before observation period begins)
- Report issued: Month 30 (auditor requires ~2 months post-observation-period for fieldwork + report)

### Why 6 Months Not 12
12-month observation is standard for the most stringent enterprise buyers (banks, healthcare). 6 months is acceptable for automotive dealer groups and OEMs — confirmed with legal team and sample enterprise procurement requirements from Toyota and Ford dealer groups. We will target 12-month observation for the next annual renewal.

---

## Control Environment

### Security (CC) Controls

**Access Control**
- CC6.1: Logical access provisioning via Okta SSO — all internal systems; MFA required for all users; quarterly access review
- CC6.2: Service accounts use IAM roles (no long-lived keys); secrets in AWS Secrets Manager with 90-day rotation
- CC6.3: Separation of duties: production deployment requires ≥ 2 approvals in GitHub; no single engineer can push to prod and approve their own change

**System Operations**
- CC7.1: Vulnerability scanning: Snyk in CI/CD pipeline (blocks merge if CVSS ≥ 7.0); weekly AWS Inspector scans on EC2/container images
- CC7.2: Intrusion detection: AWS GuardDuty enabled in all regions; alerts to PagerDuty on high severity
- CC7.3: Incident response playbook — documented procedure for breach, data exposure, service disruption; annual tabletop exercise

**Change Management**
- CC8.1: All code changes via PR with code review; production deployments via CI/CD pipeline only (no manual deploys); change log maintained in Jira

**Risk Assessment**
- CC9.1: Annual risk assessment (next: Month 22); risks documented in risk register with owner and mitigation
- CC9.2: Third-party vendor review: all vendors with access to customer data assessed annually (AWS, Okta, PagerDuty, OpenSearch)

### Availability (A) Controls

- A1.1: Availability commitments documented in customer contracts: 99.5% monthly uptime SLA for cloud services
- A1.2: Capacity monitoring: CloudWatch dashboards with auto-scaling; load tested at 2× current peak quarterly
- A1.3: Disaster recovery: multi-region failover spec (see `docs/infrastructure/multi_region_cloud_spec.md`); DR drill performed semi-annually; RTO < 5 minutes documented

### Confidentiality (C) Controls

- C1.1: Data classification policy: OEM CAD data = Confidential; repair telemetry with VIN = Restricted; aggregate de-identified data = Internal
- C1.2: Encryption: all data encrypted at rest (AES-256, S3 SSE-KMS; RDS storage encryption); all data encrypted in transit (TLS 1.3 minimum; TLS 1.0/1.1 disabled)
- C1.3: Data retention and disposal: retention schedules documented (see video recording spec); S3 lifecycle policies enforce deletion; data disposal certificates issued on customer offboarding

---

## Readiness Assessment (Month 21)

Engage auditor (target: A-LIGN, Schellman, or Coalfire — all specialize in cloud-native SOC 2) for a pre-audit readiness assessment before the observation period begins.

Readiness assessment deliverable: gap report listing controls that are missing or insufficient. Common gaps at this stage:
- Access review process not documented or not being performed
- Vendor risk assessments not formalized
- Incident response playbook exists but tabletop exercise never conducted
- Penetration test not performed in last 12 months

### Gap Remediation (Month 21–22)
All gaps identified in readiness assessment must be closed before Month 22 (start of observation period). Engineering, Security, and Legal own remediation — tracked in a dedicated Jira project.

---

## Penetration Test

SOC 2 Type II does not require a pentest, but auditors expect to see one. Enterprise buyers ask for the pentest report separately.

- Schedule external pentest: Month 22 (before observation period — findings remediated before audit clock starts)
- Scope: web APIs (dashboard backend, edge server management API), cloud infrastructure (AWS account), HoloLens 2 communication (WebSocket session protocol)
- Vendor: NCC Group or Bishop Fox (both experienced in embedded/IoT and cloud)
- Findings: Critical and High findings remediated before observation period starts; Medium findings remediated within 30 days; Low findings tracked in risk register

---

## Key Evidence Types for Auditor

| Control Area | Evidence Collected During Observation Period |
|---|---|
| Access reviews | Quarterly access review reports (Okta → Jira) |
| Vulnerability scanning | Snyk scan results per merge (CI logs); weekly AWS Inspector reports |
| Incident response | All incidents logged (even minor); tabletop exercise record |
| Change management | GitHub PR history; Jira deployment records |
| Availability | CloudWatch uptime dashboards (monthly screenshots); DR drill records |
| Encryption | AWS KMS key rotation logs; TLS configuration screenshots |
| Vendor reviews | Vendor assessment questionnaire responses (AWS, Okta, etc.) |

**Evidence collection is automated where possible** — manual screenshot collection is a SOC 2 failure mode. Implement:
- Okta API → auto-export access reviews to S3 evidence bucket monthly
- GitHub API → auto-export deployment records to S3 evidence bucket
- CloudWatch → scheduled export of uptime metrics to S3 evidence bucket

---

## Team and Timeline

| Month | Milestone |
|---|---|
| 21 | Engage auditor; conduct readiness assessment |
| 21 | Engage penetration testing firm |
| 22 | Close all readiness assessment gaps; remediate pentest Critical/High findings |
| 22 | Observation period begins |
| 22 | Evidence collection automation in place |
| 25 | Mid-period check with auditor: no surprises |
| 28 | Observation period ends |
| 28 | Auditor begins fieldwork (interviews, evidence review) |
| 30 | SOC 2 Type II report issued |

### Internal Owners
- **DRI:** Head of Engineering (or VP Engineering)
- **Security controls:** Security Engineer (dedicated hire by Month 20)
- **Evidence collection automation:** Platform Engineering
- **Policy documentation:** Legal + DRI
- **Vendor management:** Operations

---

## Customer Communication

SOC 2 Type II report is shared with enterprise customers and prospects under NDA. It is not public.

- Add "SOC 2 Type II certified" to enterprise sales materials from Month 30
- Report made available in customer portal (Manager Dashboard → Security & Compliance tab)
- OEM procurement teams: proactively share report at contract renewal or new OEM embedded deal negotiation

---

## Cost Estimate

| Item | Cost |
|---|---|
| Auditor (readiness + Type II audit) | $35,000–$55,000 |
| Penetration test | $15,000–$25,000 |
| Security Engineer hire (partial year) | $80,000–$120,000 (prorated) |
| Evidence collection tooling (Vanta or Drata) | $12,000–$18,000/year |
| Remediation engineering time | ~200 hours × $150/hr = $30,000 |
| **Total** | **~$170,000–$250,000** |

One-time cost (audit repeats annually at ~$35K for surveillance audit). Justified by: single enterprise deal at $3,500/bay × 50 bays = $175,000 ARR that SOC 2 unblocks.
