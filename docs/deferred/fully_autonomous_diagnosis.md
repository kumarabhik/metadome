# Deferred: Fully Autonomous Diagnosis (No Technician in Loop)

**Status:** Deferred — not viable pre-2030
**Earliest viable horizon:** 2030+ for limited non-HV scope
**Decision date:** Phase 0 product definition

---

## What It Is

A diagnostic and repair guidance system that operates without a human technician present in the bay — the system diagnoses the fault, determines the repair procedure, orders parts, and either guides a robot or waits for a technician to execute. No human judgment is in the diagnostic loop.

---

## Why It's Deferred

### 1. OSHA 1910.147 Requires Human Lockout/Tagout

This is the hard stop. OSHA's Control of Hazardous Energy standard explicitly requires a human worker to perform the lockout procedure and physically verify isolation before any work on hazardous energy sources. There is no OSHA exception for automated systems.

Until OSHA revises 1910.147 to accommodate automated energy isolation — and creates a parallel certification framework for automated systems performing LOTO — any product that touches HV work without a human in the loop is per-se non-compliant. This is not a matter of risk tolerance or insurance; it is a federal regulatory prohibition.

**Timeline:** OSHA rule changes take 5–10 years from proposal to final rule. No proposal for automated LOTO has been filed as of 2026.

### 2. LLM Reliability Is Insufficient for Autonomous Medical/Safety Decisions

Our product already enforces a zero-LLM-generated-repair-instruction constraint. This constraint exists because LLMs hallucinate, and a hallucinated repair step on an EV HV system can kill someone. For the current product (human in loop, human verifies before acting), this is manageable: the tech sees the step and applies professional judgment.

In a fully autonomous system, there is no human verification step. The system must be 99.99%+ reliable on repair step generation across all vehicle models, fault types, and edge cases. No LLM system in existence meets this bar, and the trajectory suggests this reliability level is 10+ years away for open-domain automotive repair.

A narrowly scoped autonomous diagnostic (single OEM, single fault type, controlled conditions) could plausibly reach 99.9% — but that is not a general autonomous diagnosis product.

### 3. Liability Attribution Is Legally Unresolved

If an autonomous diagnosis system causes property damage or injury, who is liable?
- The software company (us)?
- The dealer?
- The OEM whose data was used?
- The vehicle owner who authorized the autonomous service?

No US court has established precedent for autonomous vehicle service liability. No insurance product exists for this risk profile at a price a startup can afford. Until liability is resolved — either by legislation or settled case law — building and deploying a fully autonomous system is an unacceptable legal risk.

---

## What Would Make This Viable

| Condition | Requirement | Timeline |
|---|---|---|
| OSHA amends 1910.147 for automated energy isolation | Federal rule change | 2031+ |
| LLM reliability reaches 99.99% on automotive repair scope | AI capability | 2030+? |
| Autonomous vehicle service liability legislation passes | Federal or state law | Unknown |
| SAE develops autonomous service safety standard | Industry standard | 5–8 years after OSHA |

---

## What We Can Build Now (Human-on-Loop, Not Human-in-Loop)

The appropriate near-term framing is not "fully autonomous" but **"human-on-loop"**: the system performs all cognitive work (diagnosis, step generation, confidence scoring) and the technician only performs physical actions and provides final confirmation.

This is essentially our current product taken one step further:
- **Current:** Tech issues voice commands to advance through steps; tech performs physical actions
- **Human-on-loop (Year 3–4):** System proactively advances to next step when CV confirms previous step completion; tech intervenes only if something is wrong

The distinction from "fully autonomous": a human is still present, still physically performing actions, still has override control. OSHA compliance is preserved. Liability is clear. But the cognitive burden on the tech drops by 80%.

This is the right intermediate goal. "Human-on-loop" diagnostic guidance is achievable in Year 3–4. "No technician in loop" is not viable before 2030.
