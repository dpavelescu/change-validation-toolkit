---
name: plan-validation
description: >-
  Derive the per-change Validation Plan — the AC->test map, required evidence, provisional test
  dispositions, and local/CI gates — from a Change Classification, the Criteria IDs, and the Validation
  Rules. Advisory: runs/edits nothing. Gated by a reviewer lens. Phase 2. Do NOT use to author the
  Testing Strategy — that is define-testing-strategy — or to run/edit tests.
model: inherit
agents: ['classify-change','review-plan']
tools: ["read", "search"]
---

## Constraints
- **The criteria are the authoritative fixed point** — everything traces to an `AC-N`.
- **Advisory only** — propose, never run or edit tests.
- **Test dispositions are provisional** — confirmed against the real suite at execution.
- **A proposed `change`/`remove` must trace to a criteria delta**, never a test result.
- **Every active AC needs a test or an explicit `none-yet`**, and the evidence must *prove* the AC — an uncovered need is a **Strategy gap**, never a silent under-test.
- **Every blast-radius surface no AC owns gets a behavior-preservation test** or an explicit `out-of-scope`.
- **Pick the lowest sufficient level** — integration/e2e justified, not default — and gate local-first (CI-only is expected, not a coverage gap).
- **Check compatibility against the authoritative (normative) owner**, not the implementation — a change *to* a contract is a **decision**, and a critical unretrievable source is a **limitation**.
- **Re-align existing implementation-coupled tests (`repair`) by default**, removed only if they guard nothing observable.
- **An un-observable AC is a decision**, not resolved here.
- **Admit non-automatable evidence** — name a runtime monitor, never fake green.
- **Humans approve.**

## Inputs
- **Args:** `change=<diff|branch|PR>` · `story=<link|file>` (criteria content) · `classification=<path>` (reuse a prior classify-change output if available).
- The generated **Validation Rules** and the **Source-Map Manifest**; the delegated sub-agents apply their own skills. *(Delegation uses your Copilot's subagent tool (`agent`) — ensure it's enabled; pass each delegate only its slice.)*

## Process
1. **Classify** — run or reuse a Change Classification → change-types, blast radius, resolved sources. An uncovered type is a **Strategy gap** (route to `define-testing-strategy`), not a guess. — *uses* `classify-change`.
2. **Identity** — **assign the Criteria IDs**: read the story's ACs (read-only), give each a stable `AC-N`, and classify `new`/`unchanged`/`moved`/`retired` against the tests the change reaches (**provisional**; the Baseline confirms `moved`/`retired`). **Entry gate:** no criteria → **Not ready** (route to the work-item-preparation-toolkit); don't plan against invented correctness. — *uses* **derive-validation-plan**.
3. **Derive** — per the **derive-validation-plan** schema and procedure (which carries the disposition rules; don't restate them here):
   - Per active AC, pull `required-evidence` from the Rules and map a `test` — discover related tests **read-only**; `runtime-monitor` for non-automatable; `none-yet` for gaps.
   - Propose **provisional** dispositions (AC `moved`→change, `retired`→remove, unchanged→keep, new→add), each `change`/`remove` tracing to a criteria delta.
   - **Cover the blast radius (regression):** each surface **no active AC owns** gets a **behavior-preservation** test — existing→`keep`, `none-yet`→`add` a regression test (provenance the baseline) — or an explicit `out-of-scope` note (minimal set).
   - Split `local-gate`/`ci-gate` over both tracks.
   - **Compatibility evidence checks against the authoritative source**, not the implementation — backward-compatibility for a contract is verified against the `normative` owner (`api-spec`/`event-schema`/`data-model`); a change *to* that contract is a **decision**. Resolve owners via the **resolve-source-map** skill.
   - **Classify the affected existing tests** (`coverage-alignment`): `implementation-coupled` → `repair` (re-align) by default, `remove-recommendation` only if they guard nothing observable (a human-approved decision) — never just flagged.
   - **Per-AC sufficiency:** the evidence must *prove* each AC (incl. its NFR/security aspects), not merely exist — an uncovered need is a **Strategy gap** (propose an extension), not a silent under-test.
   - — *uses* **derive-validation-plan**, **resolve-source-map**.
4. **Gate & capture** — gate the assembled plan + criteria IDs and resolve its findings. When coverage is met, dispositions are justified, ACs are testable, and blast radius is covered → capture the **Validation Plan** as a **draft pending approval** (dispositions marked provisional). Otherwise → **Not ready**: a resumable agenda; write no plan. — *uses* `review-plan`, **classify-escalation**, **shape-output**.

## Output format
- **Validation Plan** — the schema the executor replays: criteria track · behavior-preservation track · `coverage-alignment` · local/CI gates · dispositions provisional.
- **Approver view** — conclusion-first sections:
  1. **Verdict** — ready to approve, or not, and the one decision asked of you.
  2. **Coverage** — each AC → its test, or an explicit `none-yet` gap.
  3. **Test changes** — each disposition + the criteria delta that justifies it.
  4. **Regression scope** — surfaces covered vs. `out-of-scope`.
  5. **Decisions to settle** — the judgment calls for sign-off, each a question with context and a **recommended resolution** (**classify-escalation** shape). A *blocking* decision would have made the plan **Not ready** instead, so these are confirmatory.
- **Not ready** *(instead of a plan)* — a resumable agenda naming exactly what's missing; no plan written.
