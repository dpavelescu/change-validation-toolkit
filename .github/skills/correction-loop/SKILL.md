---
name: correction-loop
description: >-
  The loop that drives a failing change to green — diagnose, emit a structured fix-request to whoever
  implements (agent-agnostic), re-assess impact, re-validate — WITHOUT the toolkit writing production
  code. The Behavior Baseline gates regression vs brittle. Used by drive-correction. Phase 3.
---

The correction loop closes the validation loop **without the toolkit touching production code**. When validation is red, it **diagnoses, emits a structured `fix-request`, and pauses** — whoever drives implementation (a human, or any implementation agent) applies the fix and **re-invokes** the loop. The toolkit owns tests, evidence, diagnosis, and the handoff; **implementation is out of scope**. It is per‑change, resumable, and runs your suite via the Execution Runner.

**One boundary, restated.** A red *criterion* test — a feature not built yet, or a regression — is **never fixed by editing the test**; it's a `fix-request` for the code. A red test whose *behavior is preserved* is a **brittle** test, repaired (decoupled) — but even that authoring is a handoff. **The toolkit writes no code at all**: it emits two kinds of handoff to the external implementer — a **`test-request`** (specify/adjust a test, from `specify-tests`) and a **`fix-request`** (author production code) — and *verifies* the result. It owns *what* must be true and *whether the result proves it*; the code (tests and production) is authored outside.

## The fix‑request (the handoff — agent‑agnostic)

```
fix-request:
  id:            FR-<n>
  trigger:       failing-criterion(AC-N) | regression(surface-id)
  expected:      <the criterion text, or the pinned baseline behavior>
  observed:      <what the Runner saw — assertion / output / status>
  test:          <the test that defends it: file::case>   # DO NOT edit — it is the test
  scope:         <where to fix — surface/code, from the blast radius>
  constraints:   [ honor <contract> (its authoritative source) · criteria are fixed · don't edit the test ]
  status:        open        # resolved when re-validation passes
```

Bounded and coherent: *what's wrong · the test · what you may not break · where to fix.* The loop writes it and stops; it does not call any particular implementer.

## The test‑request (the other handoff — specify a test)

The toolkit never writes test code either: `specify-tests` emits a **`test-request`** — a precise, self‑contained **test specification** the external implementer renders into your framework.

```
test-request:
  id:            TR-<n>
  disposition:   add | change | repair
  assert:        <the criterion text, or the pinned baseline behavior, the test must prove>   # the SPEC — from criteria/baseline, NOT the code; for BDD, the scenario ref + its Gherkin steps (the implementer authors the step-definition bindings)
  surface:       <what it exercises — endpoint / method / event / flow>
  level:         unit | component | contract | integration | e2e        # tier ①/② only
  style:         <project test style/framework; BDD → step definitions for scenario <ref>, never a parallel test>
  fixtures:      <from test-data, if needed>
  tag:           AC-<n>                                  # stamped from the Criteria IDs, for traceability
  constraints:   [ assert the criterion, NOT the implementation · honor <contract> (its authoritative source) ]
  acceptance:    <what makes it a faithful, runnable test — the toolkit verifies the authored test against this>
  status:        open        # resolved when the authored test is verified faithful AND runs
```

The two handoffs are symmetric: a **`test-request`** says *"author a test that proves this criterion, this way, without asserting the implementation,"* and a **`fix-request`** says *"make this test pass."* The toolkit owns the spec and the verification of both; the external implementer authors the code.

## The correction log (resumable loop state)

The loop runs one pass per invocation as a fresh agent — so the count of passes and the history needed to spot oscillation live **in a committed log**, never in agent memory. Each pass appends one entry; a re‑invoked agent reads it to know the pass number and which surfaces it has already routed.

```
correction-log:
  change-ref:    <change-ref>
  max-iterations: <n>
  passes:                                 # appended one per invocation
    - pass:        <n>
      red:         [ <surface-id / test-ref that failed this pass> ]
      routed:      [ { surface, as: fix-request(FR-n) | change | repair } ]
      outcome:     handed-off | green | re-plan | decision | escalation
```

**Oscillation** = a surface that returns `red` in this pass after an earlier pass already routed and resolved it. **Iteration limit reached** = `pass > max-iterations`. Either ends the loop in an escalated diagnosis.

## Loop procedure (resumable — one pass per invocation)

1. **Validate** — read the correction log (pass count; or start at pass 1), then `run-validation` over the affected slice (fail‑fast: local first). **Green & complete → done** (evidence → the Evidence Ledger).
2. **Re‑assess (on re‑invocation after a fix)** — the diff moved, so impact moved: recompute the blast radius over the **fix's delta only** and union it into the existing radius (not a full re‑plan). A **material** scope change → **return `re-plan`** (re‑run `plan-validation`, then resume); the loop never re‑plans itself. *Material* = the fix introduces a **surface, dependency, or contract boundary not already in the blast radius** (or touches a criterion); a fix confined to surfaces already covered is **not** material and the loop continues. When unsure whether a delta is material, re‑plan (the conservative default).
3. **Sort each clean fail by the Behavior Baseline** — map the test to the surface it defends, then read the baseline (the no-edit-to-pass rule):
   - behavior **moved**, no criterion owns it → **regression**
   - behavior **moved**, a criterion owns it → **justified**
   - behavior **preserved**, test red → **brittle** (it asserted internal structure the fix changed)
4. **Route:**
   - **regression** / **failing‑criterion** → emit a **`fix-request`** (code fix, external).
   - **justified** → `change` the test (a criteria delta justifies it); **brittle** → `repair` it (decouple, re‑align to assert the behavior, gated by the baseline's `preserved` verdict) — or, if it guards **nothing observable**, a **`remove` recommendation** (a human‑approved `decision`, never a silent delete). Both via `specify-tests`, then re‑validate — never to force green.
5. **Hand off & pause** — write the open `fix-request`s + the re‑assessment, and **append this pass to the correction log** (pass number · red surfaces · what was routed); **return control**. A human or any implementation agent applies them and re‑invokes.
6. **Terminate / escalate** — needs a criterion or contract to change → **decision** (structured question); **iteration limit reached** (the log's pass count exceeds `max-iterations`) or **oscillation** (a surface returns red after an earlier pass resolved it) → **escalate a diagnosis** (a limitation), never loop silently.

## Constraints

- **Never writes production code** — the loop emits a `fix-request`; implementation is out of scope.
- **Baseline is the gate** — `regression` vs `brittle` is decided by *behavior preservation*, never the red result.
- **Test never edited to pass** — test changes go through `specify-tests`, justified by a criteria delta or behavior preservation.
- **Re‑assess on every fix** — impact is re‑evaluated incrementally; a material scope change re‑plans.
- **No silent loop** — every pass ends in green, a handoff, a re‑plan, a decision, or an escalation.
- **State is persisted, not remembered** — pass count and oscillation are read from the committed correction log, since each pass is a fresh agent.
- **Decision vs limitation** — criteria/contract change → decision; can't‑resolve / oscillation → escalate a diagnosis.
