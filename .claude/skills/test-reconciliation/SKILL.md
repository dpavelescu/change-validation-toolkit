---
name: test-reconciliation
description: >-
  The disposition→action mapping and provenance/honesty guards for Test Reconciliation — the toolkit owns
  what each test asserts (from the criteria, or the baseline for regression tests) and verifies
  the result; authoring is delegated to the external implementer via a test-request. Used by
  specify-tests; evidence comes from the runner. Phase 3.
---

Test Reconciliation turns the Validation Plan's **provisional dispositions** into **verified tests** along both of the plan's tracks: an **acceptance test** for each active AC (intended behavior) and a **regression test** for each blast‑radius surface no AC owns (unchanged behavior). The toolkit (`specify-tests`) **owns *what* each test asserts** (from the Criteria IDs, or the pinned baseline for regression tests) and **verifies the authored result**; the **code is authored by the external implementer** via a **test‑request** (the toolkit writes none). **Independence comes from the toolkit — not the author — deciding the assertion**, then verifying the authored test isn't coupled to the implementation. That verification is the model's coupling check (below) — bounded, but a judgment, not a mechanically enforced guarantee. Per‑change, in‑repo.

> *Below, "author X" means **specify X (a test‑request) and verify the authored result** — the toolkit owns the spec and the check; the external implementer writes the code.*

**The two provenances (and the one that's forbidden).**
- An **acceptance test** asserts the **AC** — what *should* be true. Specified from the Criteria IDs; the spec must be derivable from the criterion alone.
- A **regression test** asserts the **pinned baseline** — what *is* true — for any blast‑radius surface no AC owns, as a behavior‑preservation net. (`internal-refactor` is the case where *every* surface is of this kind.)
- **Forbidden:** an assertion derived from the **new implementation**. Reading the impl for *mechanical wiring* (how to invoke the surface) is allowed and flagged; reading it to decide *what is correct* is the failure the separation exists to prevent.

**Telling a faithful test from a coupled one (the verification check).** A faithful test asserts only the **observable surface the criterion names** — return value, status, emitted payload, persisted state, rendered output — never internal structure, private calls, or call order. The operational test: *would it still pass under a deliberately wrong implementation of the criterion?* If yes, it asserts the wrong thing (coupled, or vacuous) → **rejected** (acceptance test) or **repaired** (existing test). This is the concrete check behind "not coupled to the implementation"; it's a judgment, but a bounded one.

## Disposition → action

Confirming the plan's provisional dispositions against reality.

| Plan disposition | Trigger (from the Criteria IDs) | Action | Justified by |
|---|---|---|---|
| `add` | new AC / `none-yet` test | specify a new acceptance test (authoring handed off) | the AC (expect red until the impl satisfies it — **fed back into the loop**, not a fault) |
| `change` | AC `moved` | adjust the test to defend the **new** wording | the criteria IDs `moved` delta — **never a red result** |
| `keep` | AC unchanged | leave the test untouched | — |
| `remove` | AC `retired` | remove / retire the test | the criteria IDs `retired` delta — **never a red result** |
| (regression) | blast‑radius surface no active AC owns (refactor = every surface) | specify/keep a test pinning the **baseline** behavior (authoring handed off) | the pinned baseline; a divergence is a **caught regression** (fed back into the loop), never a test edit |
| (repair) | *execution‑time, from the correction loop:* **brittle** — behavior **preserved** but the test is red (it asserted internal structure a fix changed) | **decouple** the test from the internals — **the asserted behavior unchanged** | the baseline's `preserved` verdict — **not** a criteria delta |

## Existing tests — alignment & re‑alignment (brownfield)

The blast radius surfaces *existing* tests that are often **implementation‑aligned** (assert internals), not criteria‑aligned. They are never inherited blindly, and **never just flagged** — each is classified and **driven to resolution**, with *signal always attached to an action the toolkit performs or prepares for one‑step approval*:

| Existing test | Detected by | Action (AI‑driven, never a dead‑end) |
|---|---|---|
| **criterion‑aligned** — defends a behavior a current criterion owns | maps to an active AC's behavior | `keep` (or `change` if the AC `moved`) |
| **behavior‑guard** — defends observable behavior, no criterion | behavior preserved, no AC owns it | `keep` as a regression guard, **and flag it upstream** as a candidate criterion |
| **implementation‑coupled, salvageable** — red while behavior is **preserved**, but a real observable behavior exists | the brittle signal + a behavioral counterpart | **`repair` automatically** — rewrite it to assert that **behavior** (from the baseline), decoupled from internals; re‑tag to the owning criterion if any |
| **implementation‑coupled, empty** — asserts only internal structure; guards **no** observable behavior | the brittle signal + no behavioral counterpart | **`remove` recommendation** — a structured `decision` (test ref · what it asserts · why low‑relevance · recommended delete), **human‑approved**, never silent |

**Re‑alignment is the default, deletion the exception.** Each time the toolkit touches a biased test it either **converts it** (raising its relevance, lowering the bias) or hands you a **one‑step, fully‑justified delete**. The suite migrates toward criteria/behavior alignment over time; you are never stuck with low‑relevance tests and no path forward. (A `remove` here is justified by *guards‑nothing‑observable*, not a criteria delta, so it is a **human‑approved decision** — losing coverage is hard to undo.)

## The no-edit-to-pass rule (three keys)

A test `change`/`remove` is honest only when all three agree:
- **Criteria IDs** — a criteria delta (`moved`/`retired`) justifies it.
- **Baseline** — the behavior delta is `justified`, not a `regression`.
- **Runner** — the evidence is *observed*, never asserted.

A test edited **because it went red, with no criteria delta**, is **masking a regression** and is forbidden — the same guard the plan reviewer applies to *dispositions*, now enforced on *actual test edits*. If behavior genuinely changed without a criterion moving, that is a **regression** (fed back into the loop) or a **criteria gap** (decision) — never a silent test edit.

**Repair is not a behavior change** (so it's not masking a regression). The rule governs edits that alter *what behavior a test accepts*. A **`repair`** — decoupling a brittle test from internal structure a fix legitimately changed — does **not** alter the asserted behavior, so it's justified by the Baseline's **`preserved`** verdict instead of a criteria delta. The guard still bites: you may repair **only** when behavior is preserved; a red test whose behavior *moved* is a regression or a justified `change`, never a "repair."

## Done = evidence, never assertion

An AC's test is **satisfied** only on **green evidence from the Runner**. A red test with no implementation yet is the normal state → **fed back into the loop** for the implementer (never a handoff, never a test softened to pass). A `runtime-monitor` / non‑automatable AC is **admitted** (recorded as a runtime monitor), **never faked green**. This is why a test cannot be declared done without running it, and why the test must be implemented to close the loop: the evidence *is* the test, run.

## Schema (the reconciliation record)

```
reconcile-ref:   <change-ref>
per-ac:
  - ac-id:        AC-<n>
    disposition:  add | change | keep | remove        # confirmed (no longer provisional)
    provenance:   criterion | baseline        # never "implementation"
    test-ref:     <test file::case the action produced/adjusted>
    justified-by: <criteria IDs delta | baseline-justified | n/a (add/keep)>
    evidence:     green | red(recheck) | runtime-monitor | none-yet
open-decisions:  [ un-observable AC → criteria gap ]
limitations:     [ can't specify/invoke a surface → toolkit gap ]
```

## Constraints

- **Independence by spec** — the toolkit owns *what* each test asserts and **verifies** it; authoring is delegated to the external implementer.
- **Criteria provenance** — assertions derive from the AC (or baseline for regression tests), never from the new impl.
- **The no-edit-to-pass rule** — every behavior‑altering `change`/`remove` traces to a criteria delta **and** a `justified` baseline delta; a **`repair`** is the only edit without a criteria delta.
- **Traceability tag** — each adjusted test is **stamped with its `AC-N` tag from the Criteria IDs** (a regression test carries its `surface-id`), so the test→criterion link is greppable.
- **Done‑on‑evidence** — an AC is satisfied only on green Runner evidence; red is fed back into the loop.
- **Regression coverage** — every blast‑radius surface no AC owns gets a behavior‑preservation test from the baseline.
- **Re‑align, don't inherit** — an implementation‑coupled existing test is **re‑aligned by default** (`repair`), deleted only when it guards nothing observable.
- **No faked green** — non‑automatable → admitted runtime monitor, never a fake pass.
- **Idiomatic & data‑backed** — tests are authored in the project's own **style** and frameworks, with fixtures from `test-data`.
- **Un‑observable AC → decision; can't‑specify → limitation** — the standard decision/limitation split.
- **Persisted** — tests and the reconciliation record live in‑repo so local and CI share them.
