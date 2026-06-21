---
name: test-reconciliation
description: >-
  The fate→action mapping and provenance/honesty guards for Test Reconciliation — materializing or
  adjusting the witnessing tests per the Validation Plan's fates, with assertions derived from the
  criteria (or the baseline for regression witnesses), never the new implementation. Used by
  implement-tests; evidence comes from the runner. Phase 3.
---

*Derived copy — canonical source is `Change-Validation-Playbook.md`; if they disagree, the playbook wins.*

Test Reconciliation turns the Validation Plan's **provisional fates** into **real witnesses** along both of the plan's tracks: a **criterion witness** for each active AC (intended behavior) and a **regression witness** for each blast‑radius surface no AC owns (unchanged behavior). It materializes new tests and adjusts existing ones so every active AC and every touched surface has a witness that traces to it, then hands them to the **Execution Runner** for evidence. It is owned by an **independent test‑implementer** (`implement-tests`) — never the producer of the production change — because a witness authored by the thing it judges is no witness at all. Its authority for *what to assert* is the **Criteria Ledger** (or, for regression witnesses, the **pinned baseline**) — **never the new implementation**. Per‑change, in‑repo, committed.

**The two provenances (and the one that's forbidden).**
- A **criterion test** asserts the **AC** — what *should* be true. Authored from the Ledger; the test‑implementer must be able to write it from the criterion alone.
- A **regression witness** asserts the **pinned baseline** — what *is* true — for any blast‑radius surface no AC owns, as a behavior‑preservation net. (`internal-refactor` is the case where *every* surface is of this kind.)
- **Forbidden:** an assertion derived from the **new implementation**. Reading the impl for *mechanical wiring* (how to invoke the surface) is allowed and flagged; reading it to decide *what is correct* is the failure the separation exists to prevent.

## Fate → action (confirming the plan's provisional fates against reality)

| Plan fate | Trigger (from the Ledger) | Action | Justified by |
|---|---|---|---|
| `add` | new AC / `none-yet` witness | author a new criterion test | the AC (expect red until the impl satisfies it — **loop input**, not a fault) |
| `change` | AC `moved` | adjust the witness to defend the **new** wording | the ledger `moved` delta — **never a red result** |
| `keep` | AC unchanged | leave the witness untouched | — |
| `remove` | AC `retired` | remove / retire the witness | the ledger `retired` delta — **never a red result** |
| (regression) | blast‑radius surface no active AC owns (refactor = every surface) | author/keep a test pinning the **baseline** behavior | the pinned baseline; a divergence is a **caught regression** (loop input), never a test edit |

## The honesty lock (three keys)

A test `change`/`remove` is honest only when all three agree:
- **Ledger** — a criteria delta (`moved`/`retired`) justifies it.
- **Baseline** — the behavior delta is `justified`, not a `regression`.
- **Runner** — the evidence is *observed*, never asserted.

A test edited **because it went red, with no criteria delta**, is **regression‑laundering** and is forbidden — the same guard the plan reviewer applies to *fates*, now enforced on *actual test edits*. If behavior genuinely changed without a criterion moving, that is a **regression** (loop input) or a **criteria gap** (decision) — never a silent test edit.

## Done = evidence, never assertion

An AC's witness is **satisfied** only on **green evidence from the Runner**. A red witness with no implementation yet is the normal state → **loop input** for the implementer (never a handoff, never a test softened to pass). A `runtime-monitor` / non‑automatable AC is **admitted** (recorded as a runtime witness), **never faked green**. This is why a witness cannot be declared done without running it, and why the test must be implemented to close the loop: the evidence *is* the test, run.

## Schema (the reconciliation record)

```
reconcile-ref:   <change-ref>
per-ac:
  - ac-id:        AC-<n>
    fate:         add | change | keep | remove        # confirmed (no longer provisional)
    provenance:   criterion | baseline        # never "implementation"
    witness-ref:  <test file::case the action produced/adjusted>
    justified-by: <ledger delta | baseline-justified | n/a (add/keep)>
    evidence:     green | red(loop-input) | runtime-monitor | none-yet
open-decisions:  [ un-observable AC → criteria gap ]
limitations:     [ can't author/invoke a surface → toolkit gap ]
```

## Guards

- **Independent witness** — tests are authored by the test‑implementer, never by the producer of the change; the implementer never writes its own witnesses.
- **Criteria provenance** — assertions derive from the AC (or baseline for regression witnesses), never from the new impl; impl is read for wiring only, flagged.
- **Honesty lock** — every `change`/`remove` traces to a criteria delta **and** a `justified` baseline delta; a red‑driven edit with no delta is regression‑laundering, forbidden.
- **Traceability tag** — each materialized/adjusted witness is **stamped with its `AC-N` tag from the Ledger** (a regression witness carries its `surface-id`), as a native annotation (`@Tag("AC-N")` / `[AC-N]`) extractable by one `AC-[0-9]+` regex, so witness→criterion is machine‑traceable. The tool stamps it from the ledger; humans never type it.
- **Done‑on‑evidence** — an AC is satisfied only on green Runner evidence; red is loop input, never a softened test or a handoff.
- **Regression coverage** — every blast‑radius surface no AC owns gets a behavior‑preservation witness from the baseline (or an explicit `out-of-scope`); a red regression witness is a **caught regression** (loop input), never softened to pass.
- **No faked green** — non‑automatable → admitted runtime witness, never a fake pass.
- **Un‑observable AC → decision; can't‑author → limitation** — the standard decision/limitation split.
- **Persisted** — witnesses and the reconciliation record live in‑repo so local and CI share them.
