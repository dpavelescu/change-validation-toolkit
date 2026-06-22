---
name: plan-validation
description: >-
  Derive the per-change Validation Plan — the AC→test map, required evidence, provisional test
  dispositions, and local/CI gates — from a Change Classification, the Criteria IDs, and the Validation
  Rules. Advisory: runs/edits nothing. Gated by a reviewer lens. Phase 2.
model: inherit
agents: ['classify-change','reconcile-criteria','review-plan']
---

Derive the **Validation Plan**: what evidence is needed to trust this change, keyed to stable AC IDs. **House rules:** the criteria are the authoritative fixed point (everything traces to an `AC‑N`); **advisory only** — propose, never run or edit tests; **test dispositions are provisional** (confirmed against the real suite at execution); a proposed `change`/`remove` **must trace to a criteria delta**, never a test result; **every active AC needs a test or an explicit `none-yet`**, and the evidence must *prove* the AC (an uncovered need is a **Strategy gap**, never a silent under‑test); **every blast‑radius surface no AC owns gets a behavior‑preservation test or an explicit `out-of-scope`**; tests pick the **lowest sufficient level** (integration/e2e justified, not default) and gate **local‑first** (CI‑only is expected, not a coverage gap); **compatibility checks against the authoritative (normative) owner**, not the implementation — a change *to* a contract is a **decision**, and a **critical unretrievable source is a limitation**; existing implementation‑coupled tests are **re‑aligned (`repair`) by default**, removed only if they guard nothing observable; an **un‑observable AC is a decision**, not resolved here; **admit non‑automatable evidence** (name a runtime monitor), never fake green; humans approve.

**Args:** `change=<diff|branch|PR>` · `story=<link|file>` (criteria content) · `classification=<path>` (reuse a prior classify‑change output if available).

## Inputs (retrieve, don't assume)
The change, the story, the generated **Validation Rules**, the **Source‑Map Manifest**. Skills and agents are cited per step below; the delegated sub‑agents apply their own skills. *(Delegation uses your Copilot's subagent tool (`agent`) — ensure it's enabled; pass each delegate only its slice.)*

## Process (classify → identity → derive → gate)
1. **Classify** — run or reuse a Change Classification → change‑types, blast radius, resolved sources. An uncovered type is a **Strategy gap** (route to `define-testing-strategy`), not a guess. — *uses* `classify-change`.
2. **Identity** — reconcile the story's ACs (pass the classification's affected tests) → the **per‑change** Criteria IDs (one stable handle per AC, for this run) + a **provisional** delta summary (`moved`/`retired` confirmed later by the Baseline). **Entry gate:** no criteria → **Not ready** (route to the work‑item‑preparation‑toolkit); don't plan against invented correctness. — *uses* `reconcile-criteria`.
3. **Derive** — per active AC pull `required-evidence` from the Rules; map a `test` (discover related tests **read‑only**; `runtime-monitor` for non‑automatable; `none-yet` for gaps); propose **provisional** dispositions (AC `moved`→change, `retired`→remove, unchanged→keep, new→add), each `change`/`remove` tracing to a criteria delta. Then **cover the blast radius (regression)**: each surface **no active AC owns** gets a **behavior‑preservation** test — existing→`keep`, `none-yet`→`add` a regression test (provenance the baseline) — or an explicit `out-of-scope` note (minimal set). Split `local-gate`/`ci-gate` over both tracks. **Compatibility evidence checks against the authoritative source**, not the implementation — backward‑compatibility for a contract is verified against the `normative` owner (`api-spec`/`event-schema`/`data-model`); a change *to* that contract is a **decision**. **Classify the affected existing tests** (`coverage-alignment`) so brownfield bias is visible and actioned: `implementation-coupled` → `repair` (re‑align) by default, `remove-recommendation` only if they guard nothing observable (a human‑approved decision) — never just flagged. **Per‑AC sufficiency:** the evidence must *prove* each AC (incl. its NFR/security aspects), not merely exist — an uncovered need is a **Strategy gap** (propose an extension), not a silent under‑test. — *uses* **validation‑plan**, **source‑map**.
4. **Gate & capture** — gate the assembled plan + criteria IDs and resolve its findings. When coverage is met, dispositions are justified, ACs are testable, and blast radius is covered → capture the **Validation Plan** as a **draft pending approval** (dispositions marked provisional). Otherwise → **Not ready**: a resumable agenda; write no plan. — *uses* `review-plan`, **escalation**, **output‑style**.

## Output
- **Validation Plan** — the schema the executor replays: criteria track · behavior‑preservation track · `coverage-alignment` · local/CI gates · dispositions provisional.
- **Approver view** — conclusion‑first sections:
  1. **Verdict** — ready to approve, or not, and the one decision asked of you.
  2. **Coverage** — each AC → its test, or an explicit `none-yet` gap.
  3. **Test changes** — each disposition + the criteria delta that justifies it.
  4. **Regression scope** — surfaces covered vs. `out-of-scope`.
  5. **Decisions to settle** — the judgment calls for sign‑off, each a question with context and a **recommended resolution** (**escalation** shape). A *blocking* decision would have made the plan **Not ready** instead, so these are confirmatory.
- **Not ready** *(instead of a plan)* — a resumable agenda naming exactly what's missing; no plan written.
