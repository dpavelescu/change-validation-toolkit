---
name: run-correction-loop
description: >-
  The loop that drives a failing change to green — diagnose, emit a structured fix-request to whoever
  implements (agent-agnostic), re-assess impact, re-validate — WITHOUT the toolkit writing production
  code. The Behavior Baseline gates regression vs brittle. Used by drive-correction. Phase 3.
---

## Inputs

- **correction-log** — the committed loop state for this change (pass count, routed surfaces, prior outcomes); read it to know the pass number and which surfaces are already routed. Start a fresh log at pass 1 if none exists.
- **change-ref** — the change under correction.
- **blast radius** — the affected surfaces/dependencies/contracts the loop validates over and re-assesses against.
- **Validation Plan** — the AC→test map, required evidence, and local/CI gates for this change (from derive-validation-plan).
- **Behavior Baseline** — the pinned current behavior per surface; gates each clean fail into regression / justified / brittle (from capture-behavior-baseline).
- **runner results** — the structured behavior observations and flakiness check from run-execution over the affected slice.

## Procedure

Run one pass per invocation as a fresh agent; persist state to the correction-log, never to agent memory.

1. **Validate** — read the correction-log (pass count, or start at pass 1), then run run-execution over the affected slice (fail-fast: local first). Green and complete → emit the `green` outcome and hand off to record-evidence-ledger; the loop is done.
2. **Re-assess (on re-invocation after a fix)** — the diff moved, so impact moved: recompute the blast radius over the fix's delta only and union it into the existing radius (not a full re-plan). If the scope change is **material**, return `re-plan` (re-run derive-validation-plan, then resume); the loop never re-plans itself. **Material** = the fix's delta touches a surface, dependency, or contract not already in the blast radius, OR touches a criterion. A fix confined to surfaces already in the radius is not material; continue.
3. **Sort each clean fail by the Behavior Baseline** — map the test to the surface it defends, then read the baseline (the no-edit-to-pass rule):
   - behavior **moved**, no criterion owns it → **regression**
   - behavior **moved**, a criterion owns it → **justified**
   - behavior **preserved**, test red → **brittle** (it asserted internal structure the fix changed)
   - behavior **preserved**, the test guards **nothing observable** → **obsolete** (it pins no surface behavior)
4. **Route:**
   - **regression** / **failing-criterion** → emit a `fix-request` (code fix, external).
   - **justified** → `change` the test (a criteria delta justifies it).
   - **brittle** → `repair` it (decouple, re-align to assert the behavior, gated by the baseline's `preserved` verdict).
   - **obsolete** → emit a `remove` recommendation (a human-approved `decision`, never a silent delete).
   - All test routes go via specify-tests, then re-validate — never to force green.
5. **Hand off and pause** — write the open `fix-request`s plus the re-assessment, append this pass to the correction-log (pass number, red surfaces, what was routed), and return control. A human or any implementation agent applies them and re-invokes.
6. **Terminate / escalate** — a criterion or contract must change → return a **decision** (structured question, via classify-escalation); **iteration limit reached** (pass count exceeds `max-iterations`) or **oscillation** (a surface returns red after an earlier pass resolved it) → escalate a diagnosis (a limitation), never loop silently.

## Output

The loop writes no production code and no test code. It emits two handoffs to the external implementer plus the loop state, and one terminal `green` outcome.

**fix-request** (author production code — agent-agnostic):

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

**test-request** (specify or adjust a test; authoring delegated, via specify-tests):

```
test-request:
  id:            TR-<n>
  disposition:   add | change | repair | remove
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

**correction-log** (resumable loop state; committed, one entry appended per pass):

```
correction-log:
  change-ref:    <change-ref>
  max-iterations: <n>
  passes:                                 # appended one per invocation
    - pass:        <n>
      red:         [ <surface-id / test-ref that failed this pass> ]
      routed:      [ { surface, as: fix-request(FR-n) | change | repair | remove } ]
      outcome:     handed-off | green | re-plan | decision | escalation
```

**green outcome** — when validation is green and complete, the loop emits a `green` correction-log outcome and hands the change to record-evidence-ledger to assemble the durable audit trail; the loop terminates.
