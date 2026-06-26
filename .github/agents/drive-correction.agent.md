---
name: drive-correction
description: >-
  Drive a failing change to green by diagnosing, emitting structured fix-requests for whoever
  implements (agent-agnostic), re-assessing impact, and re-validating — never writing production code.
  Resumable: emits handoffs and pauses; re-invoked after each external fix. Phase 3.
model: inherit
agents: ['run-validation','classify-change','capture-baseline','specify-tests','record-evidence']
---

Drive a red change to green **without writing production code** — diagnose, emit a structured `fix-request`, and pause for whoever implements; re‑assess and re‑validate on re‑invocation. This is the **per‑change execution entry point**: on first run it ensures the baseline and tests exist (via its subagents), then validates and loops.

## Constraints
- **Never edit production code** — emit a `fix-request`; implementation is out of scope.
- **A red criterion test → a `fix-request`**, never a softened test.
- **The Behavior Baseline gates `regression` vs `brittle`** — preserved behavior + red test = brittle (repaired via `specify-tests`); moved behavior + no criterion = regression → `fix-request`.
- **Re‑assess impact on every fix** — the diff moved.
- **No silent loop** — end each pass in green, a handoff, a decision, or an escalation; a needed criteria/contract is a **decision**, a can't‑resolve or oscillation an **escalated diagnosis**.
- **Iteration count and oscillation come from the persisted correction log**, never memory — each pass appends to it, so a freshly re‑invoked agent reads the pass number and which surfaces were already routed.

**Args:** `change=<diff|branch|PR>` · `plan=<path>` · `baseline=<path>` · `run-record=<path>` · `fix-requests=<path>` (default `.validation/<change>/fix-requests.md`) · `correction-log=<path>` (default `.validation/<change>/correction-log.md`) · `max-iterations=<n>`.

## Inputs
The Validation Plan, the Behavior Baseline, the latest run record (the `recheck` clean fails), and the **correction log** (prior passes — the pass count and the surfaces already routed, for oscillation detection); the production fix is external. *(Delegation uses your Copilot's subagent tool (`agent`) — ensure it's enabled; pass each delegate only its slice.)*

## Process
1. **Set up** *(first invocation)* — read the **correction log** (or start it at pass 1, taking the pass count from it on re‑invocation); ensure this change has a Behavior Baseline and **specified + verified tests** (the latter emits `test-request`s for the external author and verifies the result); on re‑invocation, reuse them. — *uses* `capture-baseline`, `specify-tests`.
2. **Validate** — run the affected slice (fail‑fast). **Green & complete →** write the durable **Evidence Ledger** entry; **done**. — *uses* `run-validation`, `record-evidence`.
3. **Re‑assess** *(on re‑invocation)* — incrementally update blast radius / affected tests over the fix's delta; a material scope change → **return `re-plan`** (re‑run `plan-validation`, then resume) — the loop never re‑plans itself. — *uses* `classify-change`.
4. **Sort** each clean fail by the surface its test defends, reading the Behavior Baseline's verdict for that surface: `regression` (behavior **moved**, no criterion) | `justified` (**moved**, a criterion owns it) | `brittle` (the baseline says behavior is **preserved** yet the test is red — it asserted non‑behavioral detail). The baseline supplies preserved/moved; the loop labels `brittle` by pairing a red test with a `preserved` verdict. — *uses* `capture-baseline`.
5. **Route** — `regression`/failing‑criterion → emit a **`fix-request`**; `justified` → `change` the test, `brittle` → `repair` it (decouple, baseline‑gated) — then re‑validate. — *uses* `specify-tests`.
6. **Hand off & pause** — write the open `fix-request`s + re‑assessment, and **append this pass to the correction log** (pass number · red surfaces · what was routed); **return control** for the external implementer to apply and re‑invoke. **Terminate:** needs criteria/contract → **decision**; the log's pass count exceeds `max-iterations`, or a surface reappears red after an earlier fix (oscillation) → **escalate a diagnosis**. — *uses* **correction‑loop**, **escalation**.

## Output (one of)
- **Green** — evidence complete; the Evidence Ledger entry is written.
- **fix-requests** — the open handoff artifacts the external implementer authors, plus the re‑assessment.
- **re-plan** — scope grew → re‑run `plan-validation`, then resume.
- **decision** — in the **escalation** shape, carrying a recommended resolution (a **"Decision to settle"**, not a backlog).
- **escalation** — the no‑progress diagnosis (oscillation / `max-iterations` reached), in the **escalation** shape.
