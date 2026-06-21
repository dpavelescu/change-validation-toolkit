---
name: drive-correction
description: >-
  Drive a failing change to green by diagnosing, emitting structured fix-requests for whoever
  implements (agent-agnostic), re-assessing impact, and re-validating — never writing production code.
  Resumable: emits handoffs and pauses; re-invoked after each external fix. Phase 3.
model: inherit
agents: ['run-validation','change-classifier','capture-baseline','implement-tests']
---

Drive a red change to green **without writing production code** — diagnose, emit a structured `fix-request`, and pause for whoever implements; re‑assess and re‑validate on re‑invocation. This is the **per‑change execution entry point**: on first run it ensures the baseline and witnesses exist (via its subagents), then validates and loops. **House rules:** **never edit production code** (emit a `fix-request`; implementation is out of scope); a red *criterion* test → a `fix-request`, **never** a softened test; the **Behavior Baseline gates** `regression` vs `brittle` (preserved behavior + red test = brittle, repaired via `implement-tests`; moved behavior + no criterion = regression → `fix-request`); **re‑assess impact on every fix** (the diff moved); **no silent loop** — end each pass in green, a handoff, a decision, or an escalation.

**Args:** `change=<diff|branch|PR>` · `plan=<path>` · `baseline=<path>` · `run-record=<path>` · `fix-requests=<path>` (default `.validation/<change>/fix-requests.md`) · `max-iterations=<n>`.

## Inputs (retrieve, don't assume)
The Validation Plan, the Behavior Baseline, and the latest run record (the `loop-input` clean fails). Apply the **correction‑loop** skill. *(Delegates to `run-validation`, `change-classifier` (re‑assess), `capture-baseline` (sort), `implement-tests` (test fixes); the production fix is external.)*

## Process (set up → validate → re‑assess → sort → route → hand off)
1. **Set up** *(first invocation)* — ensure this change has a Behavior Baseline (delegate `capture-baseline`) and materialized witnesses (delegate `implement-tests`); on re‑invocation, reuse them.
2. **Validate** — `run-validation` over the affected slice (fail‑fast). **Green & complete → done**; emit the evidence (→ Evidence Ledger).
3. **Re‑assess** *(on re‑invocation)* — incrementally update blast radius / affected tests over the fix's delta via `change-classifier`; a material scope change → **return `re-plan`** (re‑run `plan-validation`, then resume) — the loop never re‑plans itself.
4. **Sort** each clean fail by the surface its witness defends, via `capture-baseline`: `regression` (behavior moved, no criterion) | `justified` (moved, a criterion owns it) | `brittle` (observable behavior **preserved** but the test is red — it asserted non‑behavioral detail).
5. **Route** — `regression`/failing‑criterion → emit a **`fix-request`**; `justified` → `change` the witness, `brittle` → `repair` it (decouple, baseline‑gated) — both via `implement-tests`, then re‑validate.
6. **Hand off & pause** — write the open `fix-request`s + re‑assessment; **return control** for the external implementer to apply and re‑invoke. **Terminate:** needs criteria/contract → **decision**; no progress after `max-iterations` → **escalate a diagnosis**.

## Output (one of)
**Green** (evidence complete → Evidence Ledger) · **fix-requests** (open handoffs + re‑assessment, for the implementation agent/human) · **re-plan** (scope grew → re‑run `plan-validation`, then resume) · **decision** (structured question) · **escalation** (diagnosis: no progress / oscillation).

## Guards
Never‑writes‑production‑code (emit `fix-request`; re‑invoked after external fix) · witness‑never‑softened (test changes via `implement-tests`, baseline‑gated) · baseline‑is‑the‑gate (regression vs brittle by preservation, never the red result) · re‑assess‑on‑fix (diff moved → incremental impact; material scope → re‑plan) · no‑silent‑loop (green / handoff / decision / escalation; bounded by `max-iterations`) · decision‑vs‑limitation (criteria/contract → decision; can't‑resolve/oscillation → escalate diagnosis).
