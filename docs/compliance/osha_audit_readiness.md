# OSHA Audit Readiness Review
## Step 34 — Phase 3, Compliance 1

**Roadmap reference:** Phase 3, Compliance 1
**Status:** `[ ]` Not started
**Owner:** Compliance Lead + Safety Engineering
**Target:** Complete by Month 11; signed off before Phase 4 GA launch

---

## Objective

Validate that X-Ray Vision Diagnostics' HV-STOP audit log format and safety protocols meet OSHA 1910.147 (Control of Hazardous Energy — Lockout/Tagout) and OSHA 29 CFR 1910.303 (Electrical Safety) requirements. This review is conducted internally with outside counsel before submitting to a formal OSHA audit.

**Why now (Phase 3, not Phase 4):** The system is deployed with real technicians in live dealership bays. If OSHA inspects a beta site, the records must already be compliant. A post-incident finding during beta would be catastrophic for the program.

---

## Regulatory Scope

| Regulation | Relevance | Key Requirements |
|---|---|---|
| OSHA 1910.147 | Lockout/Tagout for HV battery service | Written procedures, authorized employee training, device-specific energy control procedures |
| OSHA 29 CFR 1910.303 | Electrical safety in the workplace | Working clearances from energized parts, insulated tools, PPE requirements |
| NFPA 70E (2021) | Electrical safety in the workplace (consensus standard) | Arc flash risk assessment, shock risk assessment, PPE category selection |
| SAE J2578 | General fuel cell vehicle safety | Referenced by OSHA for HV vehicle service environments |
| SAE J1742 | Electrical and electronic component qualification (EV) | HV system serviceability requirements |

---

## HV-STOP Log Review

### Current Log Format (from `docs/qa/hv_stop_safety_audit.md`)

```json
{
  "scenario_id": 1,
  "timestamp_utc": "2025-09-12T14:23:01.452Z",
  "session_id": "qa-audit-run-003",
  "bus_voltage_v": 397.2,
  "proximity_mm": 248,
  "capacitive_triggered": true,
  "thermal_triggered": true,
  "hv_stop_activated": true,
  "activation_latency_ms": 62,
  "pass": true,
  "notes": ""
}
```

### OSHA 1910.147 Log Requirements (Checklist)

| Requirement | Current Log | Gap | Action |
|---|---|---|---|
| Date and time of hazardous energy event | ✅ `timestamp_utc` | None | — |
| Identity of authorized employee | ⚠ `anon_hash` only | OSHA may require real employee ID on safety records | Add: encrypted employee ID field (decryptable by compliance officer only) |
| Equipment/machine identification | ⚠ `session_id` | Needs explicit vehicle VIN and HV component ID | Add: `vehicle_vin`, `hv_component_id` fields |
| Description of energy source | ⚠ `bus_voltage_v` | Needs energy type (DC vs. AC) and circuit label | Add: `energy_type: "DC"`, `circuit_label: "HV_MAIN_BUS"` |
| Verification that energy was controlled | ⚠ Not logged | Must log: tech acknowledgement + manual verification step | Add: `tech_acknowledged: bool`, `verification_method: str` |
| Return to service authorization | ⚠ Not logged | Must log who authorized resumption of work | Add: `resume_authorized_by: str`, `resume_timestamp_utc: str` |

### Revised Log Format (Post-Review)

```json
{
  "event_id": "uuid",
  "timestamp_utc": "2025-09-12T14:23:01.452Z",
  "session_id": "session_uuid",
  "technician_id_encrypted": "AES256_ENCRYPTED_EMP_ID",
  "vehicle_vin": "JTMEWRFV1PD012345",
  "hv_component_id": "HV_MAIN_BUS",
  "energy_type": "DC",
  "circuit_label": "Toyota_bZ4X_HV_Main",
  "bus_voltage_v": 397.2,
  "proximity_mm": 248,
  "capacitive_triggered": true,
  "thermal_triggered": true,
  "hv_stop_activated": true,
  "activation_latency_ms": 62,
  "tech_acknowledged": true,
  "tech_acknowledged_at_utc": "2025-09-12T14:23:08.012Z",
  "verification_method": "visual_confirm_plus_sensor",
  "resume_authorized_by": "TECH_SELF",
  "resume_timestamp_utc": "2025-09-12T14:23:12.881Z",
  "notes": ""
}
```

---

## Written Lockout/Tagout Procedure (Required by 1910.147)

OSHA requires a written, machine-specific energy control procedure for every piece of equipment where lockout/tagout applies. For X-Ray Vision Diagnostics, this means a procedure for each vehicle model:

### Toyota bZ4X — HV Isolation Procedure (summary)

1. Identify HV system components via CAD overlay
2. System verbally confirms: "HV bus energized at [voltage]V"
3. Power OFF vehicle (ignition off; key removed)
4. Wait 5 minutes for HV capacitor discharge
5. Disconnect 12V auxiliary battery (prevents HV relay re-engagement)
6. Verify HV bus voltage < 5V via OBD-II + headset confirmation
7. Apply lockout device to 12V battery terminal
8. System displays: "HV bus confirmed safe — proceed with service"
9. All steps logged to HV-STOP audit log

**Owner:** Safety Engineering Lead must author and sign VIN-specific procedure for each enrolled vehicle model before that model goes into production use at beta sites.

---

## Employee Training Records

OSHA requires documented training for each authorized employee who services HV systems. X-Ray Vision Diagnostics must maintain:

| Record | Content | Retention |
|---|---|---|
| Onboarding completion cert | Tech name, date, trainer, modules completed | Duration of employment + 3 years |
| HV awareness training cert | Completion of OSHA 1910.147 awareness module (< 30 min online) | Duration of employment + 3 years |
| PPE issuance record | Class 00 insulated gloves + face shield issued | Duration of employment + 3 years |

These records are maintained by the dealership HR/service director. X-Ray Vision Diagnostics provides the training content and certifies delivery.

---

## Compliance Review Process

1. **Internal review (Month 10):** Compliance Lead + Safety Engineering review all log formats and written procedures against the checklist above
2. **Outside counsel review (Month 10–11):** Labor & employment attorney with OSHA specialization reviews materials; issues legal opinion
3. **Mock inspection (Month 11):** Internal team conducts simulated OSHA inspection at one beta site; documents gaps
4. **Gap remediation (Month 11–12):** All gaps from mock inspection resolved before Phase 4 launch

---

## Exit Criteria

- [ ] Revised HV-STOP log format deployed to all 3 beta sites
- [ ] Written lockout/tagout procedure authored for Toyota bZ4X and Ford Mach-E
- [ ] Outside counsel legal opinion received (no material non-compliance findings)
- [ ] Mock OSHA inspection completed; all P0 gaps remediated
- [ ] Employee training records protocol confirmed with beta site HR teams
