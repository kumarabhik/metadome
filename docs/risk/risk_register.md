# Risk Register

**Last reviewed:** Phase 5 complete
**Review cadence:** Monthly (Product + Engineering leads); Quarterly (Executive team)

**Scoring:** Likelihood (1=Low, 2=Medium, 3=High) × Impact (1=Low, 2=Medium, 3=High) = Risk Score (1–9)

---

## Risk Categories

- **S** — Safety
- **T** — Technical
- **L** — Legal / Regulatory / Compliance
- **C** — Commercial / Market
- **O** — Operational

---

## Active Risks

| ID | Category | Risk | Likelihood | Impact | Score | Owner | Mitigation | Status |
|---|---|---|---|---|---|---|---|---|
| R-001 | S | HV-STOP fails to fire in ≥1 real-world high-voltage proximity event | 1 | 3 | 3 | Engineering / SafetyGuard | Dual-sensor confirmation; 50-scenario test matrix pre-deployment; mandatory re-certification annually; HV-STOP audit log reviewed by compliance team quarterly | Mitigated — no incidents to date |
| R-002 | S | LLM generates incorrect repair instruction; tech follows it; injury results | 1 | 3 | 3 | Engineering / DiagnosticCore | Architectural constraint: zero LLM-generated repair steps; all steps OEM-sourced and RAG-cited; audited before deployment | Mitigated — enforced by architecture |
| R-003 | T | VoiceNLP misrecognizes command in noisy bay; tech performs wrong step | 2 | 2 | 4 | ML / VoiceNLP | Wake-word gating; intent confirmation for HV-adjacent commands; recognition rate ≥ 97% target; weekly monitoring | Active — monitor weekly |
| R-004 | T | Edge server hardware failure during active repair session | 2 | 2 | 4 | Engineering | Graceful degradation UX (simplified overlay + manual fallback); edge server has local cache valid 72h without cloud; hot-swap spare dispatched same day for Enterprise accounts | Active — mitigation in place |
| R-005 | T | SLAM anchoring drift causes CAD overlay to misalign with physical vehicle | 2 | 2 | 4 | Engineering / CADRenderer | ArUco marker re-anchor on session start; alignment verification step in onboarding; tech can say "Re-anchor overlay" to force recalibration | Active — monitor |
| R-006 | L | OEM CAD data licensing agreement not renewed; data must be purged | 1 | 3 | 3 | BD / Legal | 3-year minimum term with 90-day notice; renewal process begins 6 months before expiry; data replacement plan: graceful sunset of affected vehicle models with 90-day dealer notice | Active — renewal tracker maintained |
| R-007 | L | SAE J1742 certification denied or delayed beyond Phase 4 timeline | 2 | 3 | 6 | Compliance | Interim compliance documentation package enables continued operation; outside counsel maintains relationship with panel; parallel submission track | **High risk — escalate if Phase 3 submission not accepted within 12 weeks** |
| R-008 | L | OSHA 1910.147 audit finds non-compliant HV-STOP procedure at a site | 2 | 2 | 4 | Compliance | Pre-audit mock inspection checklist; remediation protocol in place; HV-STOP audit logs available on demand; outside counsel on retainer | Active — annual audit cadence |
| R-009 | L | UK GDPR enforcement action for improper data residency | 1 | 3 | 3 | Legal / Engineering | All UK data in eu-west-2 only; data residency tested pre-launch; ICO registration complete; DPA 2018 compliance confirmed | Active — monitor |
| R-010 | C | OEM refuses to sign embedded deal; direct CAC remains unviable | 2 | 3 | 6 | BD / Product | Two-track: Toyota primary + Ford parallel; competitive pressure play (approach Ford at Month 26 if Toyota stalls); direct sales continues as fallback; cost reduction plan for direct model | **High risk — most important commercial risk** |
| R-011 | C | Competitor launches spatial diagnostics product before Phase 4 GA | 2 | 2 | 4 | Product / BD | Speed-to-market via OEM data moat (competitors face same 12–18 month data licensing process); HV-STOP certification creates regulatory barrier to entry; strong OEM relationships create customer stickiness | Active — competitive monitoring monthly |
| R-012 | C | Key dealer churns before 90-day adoption threshold | 2 | 2 | 4 | CS | Health score monitoring; CS activation intervention if < 3 repairs in Week 1; 90-day retention focus | Active — CS playbook (`docs/cs/customer_success_playbook.md`) |
| R-013 | C | Hardware damage / loss at dealer sites creates unexpected COR | 2 | 2 | 4 | Finance / CS | Enterprise contract requires dealer to insure hardware; replacement at dealer cost after Year 1 warranty; asset register tracks all units by serial number | Active — monitor quarterly |
| R-014 | O | Key employee departure (Engineering Lead or ML Lead) during Phase 2/3 critical build | 2 | 3 | 6 | CEO / HR | Documentation-first culture: all agent specs and architecture decisions in writing; no single-person knowledge silos; competitive compensation review annually | **High risk — retention is critical through Phase 3** |
| R-015 | O | Edge server supply chain disruption (Jetson AGX Orin allocation) | 2 | 2 | 4 | Engineering / Procurement | Maintain 30-unit safety stock; identify secondary supplier (alternative: NVIDIA Jetson Orin Industrial); Phase 5 volume orders placed 6 months in advance | Active — monitor lead times |
| R-016 | O | SOC 2 Type II audit identifies material control weakness | 2 | 2 | 4 | Engineering / Security | Readiness assessment Month 21 closes gaps before observation period starts; evidence collection automated; Security Engineer hired Month 20 | Active — Month 21 readiness assessment is gating |
| R-017 | T | HoloLens 2 discontinued by Microsoft (end-of-life) | 1 | 3 | 3 | Engineering | HoloLens 2 confirmed in production through 2026+; monitor Microsoft hardware roadmap; abstraction layer in CADRenderer agent allows headset swap; evaluate Magic Leap 2 and Apple Vision Pro as alternatives in Phase 5 | Active — monitor |
| R-018 | S | Technician wears headset in HV-live zone without HV-STOP activating (SafetyGuard thermal sensor fails) | 1 | 3 | 3 | Engineering | Dual-sensor: thermal + OBD-II HV status; single sensor failure triggers warning + manual HV-STOP prompt; sensor health check at session start; field calibration annually | Mitigated — dual-sensor architecture |

---

## Closed / Resolved Risks

| ID | Risk | Resolution |
|---|---|---|
| R-C01 | Headset platform selection wrong (Magic Leap 2 had better specs) | HoloLens 2 selected after 2-week bakeoff — spatial resolution and SDK maturity better for this use case |
| R-C02 | OBD-II / CAN bus integration incompatible with Toyota bZ4X | Test harness built and validated — full integration confirmed |
| R-C03 | WiFi6E network not available in dealer service bays | IT checklist created; all Phase 3 sites upgraded before install |

---

## Risk Review Cadence

| Review | Frequency | Attendees | Action |
|---|---|---|---|
| Weekly risk check | Weekly | Engineering Lead + Product | Review R-score ≥ 4 risks; update status |
| Monthly risk review | Monthly | VP Engineering + VP Product + CFO | Full register review; add/close risks |
| Quarterly executive review | Quarterly | CEO + CTO + CLO + CFO | High-score risks (≥ 6) deep dive; resource allocation for mitigation |
| Annual risk audit | Annually | Board-level | Full register; update for next 12 months |

**Escalation trigger:** Any risk with score ≥ 6 must be reviewed by the executive team within 5 business days of identification.
