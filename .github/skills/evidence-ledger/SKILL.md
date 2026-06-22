---
name: evidence-ledger
description: >-
  The schema and recording procedure for the Evidence Ledger — the durable, per-change audit trail of
  what was validated and why (criteria → witness → evidence, behavior preserved, justified test
  changes, decisions, limitations). Records only real evidence, never assertion; an output, never read
  back. Written by record-evidence when a change reaches green. Phase 3.
---

The Evidence Ledger is the **durable audit trail** of a change's validation: *what was validated, by what, and why* — structured so a human (or an auditor) reviews **behavior and decisions, not internals**. It is written when a change reaches **green on real evidence**, kept in‑repo, and travels with the merge. It is the lasting answer to "did we trust this change, and on what basis?"

**It is an output, not a source.** The toolkit records *to* it; it never reads it back to drive a future change — cross‑change impact is always recomputed from the code (blast radius + behavior baseline), so the Evidence Ledger never becomes a stored cross‑change reference. It accumulates as history (that is what an audit trail is for), but nothing downstream depends on querying it.

**Criterion by content, not by ephemeral id.** Because the Criteria IDs is per‑change run state, the ledger records each criterion by its **text** plus the **witness** that validated it — the durable test→criterion trace.

## Entry schema (one per change)

```
change-ref:        <diff|branch|PR> @ <merged commit>
recorded-at:       <when validation reached green>
strategy-version:  <provenance of the Rules in force>

validated:                            # one per active acceptance criterion
  - criterion:   <the criterion text>
    witness:     <test file::case>     # what validated it
    evidence:    green | runtime-monitor(<what is watched>) | manual
behavior-preserved:                   # the regression guard's result
  - surface:     <id>
    result:      preserved | justified(<criterion>)
test-changes:                         # justified changes, for fast review
  - witness:     <ref>
    action:      add | change | remove | repair
    justified-by: <criteria delta | baseline-preserved>
decisions:        [ { question, answer, by } ]     # the human calls made
limitations:      [ <toolkit gaps hit, honestly logged> ]
verdict:          green               # recorded only when every active criterion has evidence
```

## Recording procedure (on green)

1. **Assemble** from the change's own artifacts — the Validation Plan (criteria, gates), the Behavior Baseline reconciliation (preserved/justified), the run records (green evidence per witness), the test‑reconciliation record (fates + justifications), and any decisions/limitations raised along the way.
2. **Record criteria by content** — each active criterion's text + its witness + its evidence (green, or an **admitted** runtime‑monitor — **never faked green**).
3. **Record the justified changes** — every `add`/`change`/`remove`/`repair`, each with its justification (a criteria delta, or `baseline-preserved` for a repair).
4. **Record decisions and limitations** — the human calls (question + answer) and any toolkit gaps hit, honestly; nothing hidden.
5. **Write** the entry in‑repo (kept; travels with the merge). **Verdict `green` only when every active criterion has evidence** (green or an admitted runtime witness); otherwise it is not done — the change is still in the loop, not in the ledger.

## Guards

- **Evidence, never assertion** — only green‑on‑real‑runs and admitted runtime‑monitors are recorded.
- **Criterion by content** — recorded by text + witness, not the ephemeral per‑change id.
- **Output, not a source** — written for humans/audit; never read back to drive a future change.
- **Justified changes only** — every recorded test change carries its justification.
- **Honest gaps** — limitations and decisions are recorded; a non‑green change does not get a `green` verdict.
- **Behavior + decisions, not internals** — structured for fast review.
