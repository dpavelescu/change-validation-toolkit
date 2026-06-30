---
name: reconcile-tests
description: >-
  The disposition→action mapping and provenance/honesty guards for Test Reconciliation — the toolkit owns
  what each test asserts (from the criteria, or the baseline for regression tests) and verifies
  the result; authoring is delegated to the external implementer via a test-request. Used by
  specify-tests; evidence comes from the runner. Phase 3.
---

## Inputs

- **plan dispositions** — the Validation Plan's provisional dispositions (`add` | `change` | `keep` | `remove`), confirmed here against reality.
- **Criteria IDs** — stable AC ids with their delta tags (`moved` | `retired` | unchanged | `none-yet`); the source of every acceptance-test assertion.
- **baseline** — the pinned Behavior Baseline and its per-surface verdict (`preserved` | `justified` | `regression`); the source of every regression-test assertion.
- **runner inputs** — the Execution Runner's observed evidence (`green` | `red`) for each test; the only source of done.

## Procedure

Authoring is delegated: "author a test" means specify a test-request and verify the authored result. The toolkit owns the spec and the check; the external implementer writes the code.

1. Confirm each plan disposition against reality and act:
   - `add` — new AC or `none-yet` test: specify a new acceptance test (handed off). Expect red until the impl satisfies it; feed red back into the loop, never a fault.
   - `change` — AC `moved`: adjust the test to defend the new wording. Justify by the Criteria IDs `moved` delta, never by a red result.
   - `keep` — AC unchanged: leave the test untouched.
   - `remove` — AC `retired`: remove or retire the test. Justify by the Criteria IDs `retired` delta, never by a red result.
   - `repair` — execution-time, from the correction loop: a test is red while behavior is `preserved` (it asserted internal structure a fix changed). Decouple the test from the internals; keep the asserted behavior unchanged. Justify by the baseline's `preserved` verdict, not a criteria delta.
2. Cover regression: for every blast-radius surface no active AC owns (for `internal-refactor`, every surface), specify or keep a test pinning the baseline behavior (handed off). Treat a divergence as a caught regression fed back into the loop, never a test edit.
3. Set each test's assertion provenance and enforce the forbidden source:
   - Acceptance test asserts the AC — specify it from the Criteria IDs; the spec must be derivable from the criterion alone (`provenance: criterion`).
   - Regression test asserts the pinned baseline (`provenance: baseline`).
   - Forbidden: an assertion derived from the new implementation. Reading the impl for mechanical wiring (how to invoke the surface) is allowed and flagged; reading it to decide what is correct is rejected.
4. Verify the authored test is not coupled to the implementation. A faithful test asserts only the observable surface the criterion names — return value, status, emitted payload, persisted state, rendered output — never internal structure, private calls, or call order. Apply the pass/fail rule: reject the test if it would still pass under a deliberately wrong implementation of the criterion (it asserts the wrong thing — coupled or empty) → reject a new acceptance test, `repair` an existing one.
5. Resolve every existing brownfield test driven to a resolution; never inherit blindly and never just flag. Classify and act:
   - criterion-aligned (maps to an active AC's behavior): `keep`, or `change` if the AC `moved`.
   - behavior-guard (preserves observable behavior, no AC owns it): `keep` as a regression guard and flag it upstream as a candidate criterion.
   - implementation-coupled, fixable (red while behavior `preserved`, but a real observable behavior exists): `repair` automatically — rewrite it to assert that behavior from the baseline, decoupled from internals; re-tag to the owning criterion if any.
   - implementation-coupled, empty (asserts only internal structure, guards no observable behavior): emit a `remove` recommendation as a structured `decision` (test ref · what it asserts · why low-relevance · recommended delete), human-approved, never silent.
6. Enforce the no-edit-to-pass rule. A behavior-altering `change` or `remove` is honest only when all three agree: a Criteria IDs delta (`moved`/`retired`) justifies it, the baseline behavior delta is `justified` (not `regression`), and the runner evidence is observed (never asserted). A test edited because it went red with no criteria delta is masking a regression and is forbidden: route a genuine behavior change with no criterion moved to a regression (fed back into the loop) or a criteria-gap `decision`, never a silent edit. `repair` is the only edit without a criteria delta — it does not alter the asserted behavior, so it is justified by the baseline's `preserved` verdict; repair only when behavior is preserved.
7. Stamp each adjusted test with its `AC-N` tag from the Criteria IDs (a regression test carries its `surface-id`), so the test→criterion link is greppable.
8. Close an AC only on green runner evidence; a red test with no impl yet is fed back into the loop, never softened to pass. Admit a `runtime-monitor` / non-automatable AC as a recorded runtime monitor, never faked green. The evidence is the test, run.
9. Persist the reconciled tests and the reconciliation record in-repo so local and CI share them.

## Output

```
reconcile-ref:   <change-ref>
per-ac:
  - ac-id:        AC-<n>
    disposition:  add | change | keep | remove | repair   # confirmed (no longer provisional)
    provenance:   criterion | baseline                    # never "implementation"
    test-ref:     <test file::case the action produced/adjusted>
    justified-by: <criteria IDs delta | baseline-preserved | baseline-justified | n/a (add/keep)>
    evidence:     green | red | runtime-monitor | none-yet  # red is fed back into the loop
open-decisions:  [ un-observable AC → criteria gap ]
limitations:     [ can't specify/invoke a surface → toolkit gap ]
```
