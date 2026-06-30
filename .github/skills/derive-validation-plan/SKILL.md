---
name: derive-validation-plan
description: >-
  The schema and derivation procedure for the per-change Validation Plan — criterion identity (stable
  AC ids), the AC→test map, the evidence required per acceptance criterion, the provisional test
  dispositions, and the local/CI gates. Used by plan-validation to derive the plan and by review-plan to gate it. Phase 2.
---

## Inputs

- **Change Classification** — change-types, blast radius, and resolved sources, from classify-change.
- **Criteria IDs / active ACs + status** — each active acceptance criterion with a stable id (`AC-1`, `AC-2`…) assigned for this change's run, its `text`, `status`, and `source-ref`. Read the story; never modify it. The human owns the prose ACs; the toolkit owns identity. These IDs are per-change run-state: they live on the branch and are discarded after merge; never accumulate them across changes or into `main`. Set `status` against the existing tests the change reaches (blast radius); match semantically (the AC's meaning vs. what a test asserts), not positionally:

  | `status` | meaning | provisional disposition |
  |---|---|---|
  | `unchanged` | an existing test covers it, wording stable | `keep` |
  | `moved` | a test covers it but the wording shifts its meaning | `change` |
  | `new` | nothing covers it yet | `add` |
  | `retired` | a test covers a criterion no longer in the story | `remove` |

- **Validation Rules** — the required-evidence projection keyed by change-type, from derive-validation-rules.
- **Source-Map** — locate related existing tests via the `tests` kind; see resolve-source-map.

## Procedure

1. **Assign the Criteria IDs.** Give each active AC its stable id and `status` (table above). `moved`/`retired` are provisional — capture-behavior-baseline confirms whether behavior actually moved. If "same one reworded, or new?" is genuinely ambiguous, emit a decision; never guess silently. Sets `criteria[].ac-id`.
2. **Pull and check required-evidence per active AC.** Take `required-evidence` from the Validation Rules for the matched change-type(s), then confirm it proves *this* criterion — not merely that some rule applies. If the type-evidence does not prove the AC (a story-specific need, or an NFR/security aspect the Strategy does not cover), emit a Strategy-gap proposal and set the AC's evidence to that proposal; do not lower coverage. The Strategy still wins; the plan only proposes. Sets `criteria[].required-evidence`.
3. **Map a test to each AC.** Discover related existing tests read-only (via the Source-Map `tests` kind) and set `test`. Where confidence cannot be proven pre-merge, set `kind: runtime-monitor` and list it under `non-automatable`; never fake a green test. An AC with no test yet gets `kind: none-yet` with a reason — a surfaced coverage gap, not a hidden one. Sets `criteria[].test` and `non-automatable`.
4. **Propose dispositions for related existing tests (provisional).**
   - AC `moved` → `change` (the test now defends an outdated wording).
   - AC `retired` → `remove`.
   - AC `unchanged` with a test → `keep`.
   - AC `new` / `none-yet` → `add`.
   - Trace every `change`/`remove` rationale to a criteria delta (AC moved/retired); never to a test result. Result-driven test edits are an execution-time concern and are forbidden as a plan justification. Sets `criteria[].proposed-disposition` and `criteria[].rationale`.
5. **Cover the blast radius (regression).** List each blast-radius surface no active AC owns — the smallest sufficient set, not every transitive node. Give each a behavior-preservation test: an existing test → `keep`; none yet → `add` a regression test (authoring handed off; provenance: the pinned baseline, never the change). A surface deliberately excluded gets an explicit `out-of-scope` note. For `internal-refactor`, every surface is behavior-preservation, since no AC moved. Sets `behavior-preservation` and `out-of-scope`.
6. **Classify the existing coverage the blast radius surfaced (`coverage-alignment`).** Make the brownfield bias visible: mark each affected existing test `criterion-aligned` (→ keep/change), `behavior-guard` (→ keep; flag upstream as a candidate criterion), or `implementation-coupled` (→ `repair` to re-align by default; `remove-recommendation` only if it guards nothing observable). Cross-check the plan against the test specifications in any style (xUnit, BDD/Gherkin `.feature` scenarios, property/table): the plan owns relevance and coverage (criteria-driven). A criterion with no test scenario is a coverage gap (`add`); a scenario with no criterion is an orphan (→ `behavior-guard` or re-align/decision). The specs check the criteria; they never define coverage. Confirmed at execution by the baseline; see reconcile-tests. Carry a disposition on every entry. Sets `coverage-alignment`.
7. **Set gates (lowest level, fail-fast).** Choose each test at the lowest test level that proves it; do not reach for integration/e2e where a unit/component/contract test gives the same guarantee. Split both tracks into `local-gate` / `ci-gate` by level and runnability: fast low-level tests run locally first; cross-boundary / infra-dependent tests (integration, e2e) that cannot run locally are `ci-gate`. CI-only by necessity is expected, not a coverage gap. Sets `local-gate` and `ci-gate`.
8. **Mark provisional and check Gate readiness.** Dispositions are intentions; the real reconciliation against the suite (with a behavior baseline, by specify-tests) happens at execution. Then confirm the plan is ready:
   - **Coverage** — every active AC has a `test` (test, runtime-monitor, manual) or an explicit `none-yet` with a reason.
   - **Sufficiency** — each AC's evidence proves it (incl. its NFR/security aspects); an AC needing evidence the Strategy does not cover is a Strategy gap (propose an extension), not a silent under-test.
   - **Disposition justification** — every proposed `change`/`remove` traces to a criteria delta.
   - **Testable** — each AC is observable/verifiable as written; else it is a criteria gap (escalate as a decision).
   - **Blast radius covered (regression)** — every blast-radius surface is owned by an active AC, carries a behavior-preservation test (or explicit `none-yet`), or has an explicit `out-of-scope` note. A silently uncovered touched surface is a regression hole, not a pass.
   - **Lowest sufficient level** — each test is at the lowest level that proves it; any integration/e2e is justified by what only it can verify.
   - **No unresolved blocking decision** — open decisions are answered, deferred, or out-of-scope; record them in `open-decisions`.

   If any check fails → emit **Not ready**: a resumable agenda; write no plan. Never partial.

## Output

```
change-ref:        <diff|branch|PR>
change-types:      [ ... ]              # from classify-change
blast-radius:      <changed surfaces · dependents · boundaries crossed>
strategy-version:  <provenance of the Rules used>

criteria:                               # one block per ACTIVE acceptance criterion
  - ac-id:            AC-<n>
    required-evidence: [ ... ]          # from the matched Rules for the change-type(s)
    test:             { kind: test | runtime-monitor | manual | none-yet, ref, status }
    proposed-disposition:    keep | change | add | remove      # PROVISIONAL — re: related existing tests
    rationale:        <why; for change/remove MUST trace to a criteria delta (AC moved/retired)>

behavior-preservation:                  # one block per BLAST-RADIUS surface NOT owned by an active AC
  - surface-id:        <endpoint | consumer | shared method | contract | component>
    why-in-radius:     <how the change reaches it — caller, consumer, shared dependency>
    required-evidence: [ ... ]          # behavior-preservation (regression) evidence from the Rules
    test:              { kind: test | runtime-monitor | none-yet, ref }   # provenance: the pinned baseline
    proposed-disposition:     keep | add        # keep = existing test suffices; add = specify a REGRESSION test, authoring handed off (provenance: baseline)
out-of-scope:      [ { surface-id, why-excluded } ]     # blast-radius surfaces deliberately not covered by a test

coverage-alignment:                     # the existing tests the blast radius surfaced, classified (brownfield bias made visible)
  - test-ref:    <existing test>
    alignment:   criterion-aligned | behavior-guard | implementation-coupled
    disposition: keep | change | repair | remove-recommendation   # repair = behavior-preserving re-align (default); remove-recommendation = a human-approved decision; same vocabulary as reconcile-tests

local-gate:        [ evidence that must pass locally before PR ]
ci-gate:           [ broader evidence the CI gate adds ]
non-automatable:   [ { ac-id, runtime-monitor } ]       # admitted, not faked
open-decisions:    [ escalations blocking completeness ]
```
