---
name: test-reconciliation
description: >-
  The fate→action mapping and provenance/honesty guards for Test Reconciliation — the toolkit owns
  what each witness asserts (from the criteria, or the baseline for regression witnesses) and verifies
  the result; authoring is delegated to the external implementer via a test-request. Used by
  specify-tests; evidence comes from the runner. Phase 3.
---

Test Reconciliation turns the Validation Plan's **provisional fates** into **verified witnesses** along both of the plan's tracks: a **criterion witness** for each active AC (intended behavior) and a **regression witness** for each blast‑radius surface no AC owns (unchanged behavior). The toolkit (`specify-tests`) **owns *what* each witness asserts** (from the Criteria IDs, or the pinned baseline for regression witnesses) and **verifies the authored result**; the **code is authored by the external implementer** via a **test‑request** (the toolkit writes none). **Independence holds because the toolkit — not the author — decides the assertion**, and verifies the authored test isn't coupled to the implementation: a witness authored by the thing it judges is no witness at all. Per‑change, in‑repo.

> *Below, "author X" means **specify X (a test‑request) and verify the authored result** — the toolkit owns the spec and the check; the external implementer writes the code.*

**The two provenances (and the one that's forbidden).**
- A **criterion test** asserts the **AC** — what *should* be true. Specified from the Criteria IDs; the spec must be derivable from the criterion alone.
- A **regression witness** asserts the **pinned baseline** — what *is* true — for any blast‑radius surface no AC owns, as a behavior‑preservation net. (`internal-refactor` is the case where *every* surface is of this kind.)
- **Forbidden:** an assertion derived from the **new implementation**. Reading the impl for *mechanical wiring* (how to invoke the surface) is allowed and flagged; reading it to decide *what is correct* is the failure the separation exists to prevent.

## Fate → action (confirming the plan's provisional fates against reality)

| Plan fate | Trigger (from the Criteria IDs) | Action | Justified by |
|---|---|---|---|
| `add` | new AC / `none-yet` witness | author a new criterion test | the AC (expect red until the impl satisfies it — **loop input**, not a fault) |
| `change` | AC `moved` | adjust the witness to defend the **new** wording | the criteria IDs `moved` delta — **never a red result** |
| `keep` | AC unchanged | leave the witness untouched | — |
| `remove` | AC `retired` | remove / retire the witness | the criteria IDs `retired` delta — **never a red result** |
| (regression) | blast‑radius surface no active AC owns (refactor = every surface) | author/keep a test pinning the **baseline** behavior | the pinned baseline; a divergence is a **caught regression** (loop input), never a test edit |
| (repair) | *execution‑time, from the correction loop:* **brittle** — behavior **preserved** but the test is red (it asserted internal structure a fix changed) | **decouple** the witness from the internals — **the asserted behavior unchanged** | the baseline's `preserved` verdict — **not** a criteria delta |

## Existing tests — alignment & re‑alignment (brownfield)

The blast radius surfaces *existing* tests that are often **implementation‑aligned** (assert internals), not criteria‑aligned. They are never inherited blindly, and **never just flagged** — each is classified and **driven to resolution**, with *signal always attached to an action the toolkit performs or prepares for one‑step approval*:

| Existing test | Detected by | Action (AI‑driven, never a dead‑end) |
|---|---|---|
| **criterion‑aligned** — defends a behavior a current criterion owns | maps to an active AC's behavior | `keep` (or `change` if the AC `moved`) |
| **behavior‑guard** — defends observable behavior, no criterion | behavior preserved, no AC owns it | `keep` as a regression guard, **and flag it upstream** as a candidate criterion |
| **implementation‑coupled, salvageable** — red while behavior is **preserved**, but a real observable behavior exists | the brittle signal + a behavioral counterpart | **`repair` automatically** — rewrite it to assert that **behavior** (from the baseline), decoupled from internals; re‑tag to the owning criterion if any |
| **implementation‑coupled, empty** — asserts only internal structure; guards **no** observable behavior | the brittle signal + no behavioral counterpart | **`remove` recommendation** — a structured `decision` (test ref · what it asserts · why low‑relevance · recommended delete), **human‑approved**, never silent |

**Re‑alignment is the default, deletion the exception.** Each time the toolkit touches a biased test it either **converts it** (raising its relevance, lowering the bias) or hands you a **one‑step, fully‑justified delete**. The suite migrates toward criteria/behavior alignment over time; you are never stuck with low‑relevance tests and no path forward. (A `remove` here is justified by *guards‑nothing‑observable*, not a criteria delta, so it is a **human‑approved decision** — losing coverage is hard to undo.)

## The honesty lock (three keys)

A test `change`/`remove` is honest only when all three agree:
- **Criteria IDs** — a criteria delta (`moved`/`retired`) justifies it.
- **Baseline** — the behavior delta is `justified`, not a `regression`.
- **Runner** — the evidence is *observed*, never asserted.

A test edited **because it went red, with no criteria delta**, is **regression‑laundering** and is forbidden — the same guard the plan reviewer applies to *fates*, now enforced on *actual test edits*. If behavior genuinely changed without a criterion moving, that is a **regression** (loop input) or a **criteria gap** (decision) — never a silent test edit.

**Repair is not a behavior change** (so it's not laundering). The lock governs edits that alter *what behavior a test accepts*. A **`repair`** — decoupling a brittle test from internal structure a fix legitimately changed — does **not** alter the asserted behavior, so it's justified by the Baseline's **`preserved`** verdict instead of a criteria delta. The guard still bites: you may repair **only** when behavior is preserved; a red test whose behavior *moved* is a regression or a justified `change`, never a "repair."

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
    justified-by: <criteria IDs delta | baseline-justified | n/a (add/keep)>
    evidence:     green | red(loop-input) | runtime-monitor | none-yet
open-decisions:  [ un-observable AC → criteria gap ]
limitations:     [ can't author/invoke a surface → toolkit gap ]
```

## Guards

- **Independence by spec** — the toolkit owns *what* each witness asserts (from the criteria/baseline) and **verifies** it; **authoring is delegated to the external implementer**. A witness that asserts the implementation is **rejected**, not accepted — independence is in the spec + the check, not in who types the code.
- **Criteria provenance** — assertions derive from the AC (or baseline for regression witnesses), never from the new impl; impl is read for wiring only, flagged.
- **Honesty lock** — every behavior‑altering `change`/`remove` traces to a criteria delta **and** a `justified` baseline delta; a red‑driven edit with no delta is regression‑laundering, forbidden. A **`repair`** is the only edit without a criteria delta, and only when the baseline says `preserved` (the asserted behavior is unchanged).
- **Traceability tag** — each materialized/adjusted witness is **stamped with its `AC-N` tag from the Criteria IDs** (a regression witness carries its `surface-id`), as a native annotation (`@Tag("AC-N")` / `[AC-N]`) extractable by one `AC-[0-9]+` regex, so the witness→criterion link is **greppable**. The tool stamps it from the criteria IDs; humans never type it.
- **Done‑on‑evidence** — an AC is satisfied only on green Runner evidence; red is loop input, never a softened test or a handoff.
- **Regression coverage** — every blast‑radius surface no AC owns gets a behavior‑preservation witness from the baseline (or an explicit `out-of-scope`); a red regression witness is a **caught regression** (loop input), never softened to pass.
- **Re‑align, don't inherit** — an implementation‑coupled existing test is **re‑aligned by default** (`repair` → assert behavior; re‑tag to a criterion if owned), and **deleted only when it guards nothing observable**, as a human‑approved `decision`. Signal is never the end state — it always carries the action.
- **No faked green** — non‑automatable → admitted runtime witness, never a fake pass.
- **Idiomatic & data‑backed** — witnesses are authored in the project's own **style** (xUnit, property/table‑based) and frameworks (from the typed `tests` exemplars + `coding-guidelines`), with fixtures from `test-data`. **BDD:** a Gherkin scenario *is* the criterion — implement the **step definitions** and run via the project's BDD tooling; **never fork a scenario into a parallel test.** A style/framework it can't author or un‑buildable test data is a *limitation*, surfaced, never faked.
- **Un‑observable AC → decision; can't‑author → limitation** — the standard decision/limitation split.
- **Persisted** — witnesses and the reconciliation record live in‑repo so local and CI share them.
