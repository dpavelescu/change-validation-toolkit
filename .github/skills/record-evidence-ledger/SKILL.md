---
name: record-evidence-ledger
description: >-
  The schema and recording procedure for the Evidence Ledger — the durable, per-change audit trail of
  what was validated and why (criteria → test → evidence, behavior preserved, blast-radius coverage, justified test
  changes, decisions, limitations). Records only real evidence, never assertion; an output, never read
  back. Written by record-evidence when a change reaches green. Phase 3.
---

## Inputs

- **Validation Plan** — the active acceptance criteria and the local/CI gates (from derive-validation-plan), including the Plan's `out-of-scope` surfaces.
- **Behavior Baseline reconciliation** — per blast-radius surface, `preserved` or `justified(<criterion>)` (from capture-behavior-baseline).
- **run records** — the green-on-real-runs evidence per test (from run-execution).
- **reconcile-tests record** — the test dispositions (`add`/`change`/`remove`/`repair`) with their justifications (from reconcile-tests).
- **decisions/limitations** — the human calls and toolkit gaps raised along the way (per classify-escalation).
- **change-ref / strategy-version** — the merged-commit reference for the change and the provenance of the Rules in force.

## Procedure

1. Assemble the entry from the inputs above; record nothing that is not backed by one of them.
2. Record each active criterion by its **text** plus the **test** that validated it — never by the per-change Criteria ID, which is temporary run state. Record evidence as `green`, an admitted `runtime-monitor(<what is watched>)`, or `manual`; never record a runtime monitor as green.
3. Record every justified test change — each `add`/`change`/`remove`/`repair` with its justification (a criteria delta, or `baseline-preserved` for a repair). Record no test change without a justification.
4. Record the decisions (question + answer + by) and the limitations (toolkit gaps hit); hide nothing.
5. Reconcile coverage so every blast-radius surface is accounted for. Verified surfaces are already in `behavior-preserved`; into `not-verified`, list each surface not proven pre-merge — the Plan's `out-of-scope` (reason `excluded`) and any surface a limitation left uncovered (reason `unverified`, referencing the limitation). Record green only with this split shown, never as an unqualified pass.
6. Write the entry in-repo (kept; travels with the merge). Set verdict `green` only when every active criterion has evidence (green or an admitted runtime monitor); otherwise the change stays in the loop and is not written to the ledger. The ledger is an output: write to it, never read it back to drive a future change.

## Output

One entry per change:

```
change-ref:        <diff|branch|PR> @ <merged commit>
recorded-at:       <when validation reached green>
strategy-version:  <provenance of the Rules in force>

validated:                            # one per active acceptance criterion
  - criterion:   <the criterion text>
    test:        <test file::case>     # what validated it
    evidence:    green | runtime-monitor(<what is watched>) | manual
behavior-preserved:                   # the regression guard's result
  - surface:     <id>
    result:      preserved | justified(<criterion>)
test-changes:                         # justified changes, for fast review
  - test:        <ref>
    action:      add | change | remove | repair
    justified-by: <criteria delta | baseline-preserved>
decisions:        [ { question, answer, by } ]     # the human calls made
limitations:      [ <toolkit gaps hit, honestly logged> ]
not-verified:                         # blast-radius surfaces NOT proven pre-merge (verified ones are in behavior-preserved)
  - surface:   <id>
    reason:    excluded | unverified  # excluded = plan out-of-scope (deliberate); unverified = a limitation left it uncovered
    detail:    <why excluded, or the limitation that blocked it — reference it, don't restate>
verdict:          green               # every active criterion has evidence; read with `not-verified` for the blast-radius surfaces left unproven
```
