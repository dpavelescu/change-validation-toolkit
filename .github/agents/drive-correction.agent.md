---
name: drive-correction
description: >-
  Drive a failing change to green by diagnosing, emitting structured fix-requests for whoever
  implements (agent-agnostic), re-assessing impact, and re-validating — never writing production code.
  Resumable: emits handoffs and pauses; re-invoked after each external fix. Phase 3.
  Not for deriving the plan (plan-validation) or writing production/test code.
model: inherit
agents: ['run-validation','classify-change','capture-baseline','specify-tests','record-evidence']
tools: ["read", "edit"]
---

## Constraints
- **Never edit production code** — emit a `fix-request`; implementation is out of scope.
- **A red criterion test → a `fix-request`**, never a softened test.
- **The Behavior Baseline gates `regression` vs `brittle`** — preserved behavior + red test = brittle (repaired via `specify-tests`); moved behavior + no criterion = regression → `fix-request`.
- **Re-assess impact on every fix** — the diff moved.
- **No silent loop** — end each pass in green, a handoff, a decision, or an escalation; a needed criteria/contract is a **decision**, a can't-resolve or oscillation an **escalated diagnosis**.
- **Iteration count and oscillation come from the persisted correction log**, never memory — each pass appends to it, so a freshly re-invoked agent reads the pass number and which surfaces were already routed.

## Inputs
The Validation Plan, the Behavior Baseline, the latest run record (the `recheck` clean fails), and the **correction log** (prior passes — the pass count and the surfaces already routed, for oscillation detection); the production fix is external.

**Args:** `change=<diff|branch|PR>` · `plan=<path>` · `baseline=<path>` · `run-record=<path>` · `fix-requests=<path>` (default `.validation/<change>/fix-requests.md`) · `correction-log=<path>` (default `.validation/<change>/correction-log.md`) · `max-iterations=<n>` (default `3`).

## Process
1. **Set up** — read the **correction log** to discriminate the invocation: absent/empty correction log ⇒ first invocation (set up baseline + tests, start at pass 1); present ⇒ re-invocation (take the pass count from it, reuse baseline/tests, run validation). On first invocation, ensure this change has a Behavior Baseline and **specified + verified tests** (the latter emits `test-request`s for the external author and verifies the result). — *uses* `capture-baseline`, `specify-tests`.
2. **Validate** — run the affected slice (fail-fast). **Green & complete →** write the durable **Evidence Ledger** entry; **done**. — *uses* `run-validation`, `record-evidence`.
3. **Re-assess** *(on re-invocation)* — incrementally update blast radius / affected tests over the fix's delta; a material scope change → **return `re-plan`** (re-run `plan-validation`, then resume) — the loop never re-plans itself. — *uses* `classify-change`.
4. **Sort** each clean fail by the surface its test defends, reading the Behavior Baseline's verdict for that surface: `regression` (behavior **moved**, no criterion) | `justified` (**moved**, a criterion owns it) | `brittle` (the baseline says behavior is **preserved** yet the test is red — it asserted non-behavioral detail) | `obsolete` (behavior **preserved** and the test guards **nothing observable** — it pins no surface behavior). The baseline supplies preserved/moved; the loop labels `brittle` by pairing a red test with a `preserved` verdict. — *uses* `capture-baseline`.
5. **Route** — `regression`/failing-criterion → emit a **`fix-request`**; `justified` → `change` the test, `brittle` → `repair` it (decouple, baseline-gated), `obsolete` → emit a `remove` recommendation as a **`decision`** (human-approved, never a silent delete) — then re-validate. — *uses* `specify-tests`, `classify-escalation`.
6. **Hand off & pause** — write the open `fix-request`s + re-assessment, and **append this pass to the correction log** (pass number · red surfaces · what was routed); **return control** for the external implementer to apply and re-invoke. **Terminate:** needs criteria/contract → **decision**; the log's pass count exceeds `max-iterations`, or a surface reappears red after an earlier fix (oscillation) → **escalate a diagnosis**. — *uses* **run-correction-loop**, **classify-escalation**.

## Output format
Returns exactly one of:
- **Green** — evidence complete; the Evidence Ledger entry is written.
- **fix-requests** — the open handoff artifacts the external implementer authors, plus the re-assessment.
- **re-plan** — scope grew → re-run `plan-validation`, then resume.
- **decision** — in the **classify-escalation** shape, carrying a recommended resolution (a **"Decision to settle"**, not a backlog) — including an obsolete test's `remove` recommendation, emitted as a human-approved decision, never a silent delete.
- **escalation** — the no-progress diagnosis (oscillation / `max-iterations` reached), in the **classify-escalation** shape.
