---
name: validation-plan
description: >-
  The schema and derivation procedure for the per-change Validation Plan — the AC→test map, the
  evidence required per acceptance criterion, the provisional test dispositions, and the local/CI gates.
  Used by plan-validation to derive the plan and by review-plan to gate it. Phase 2.
---

The Validation Plan states **what evidence is needed to trust this change** along two tracks: **criterion evidence** (per active AC — *does the intended change work?*) and **behavior‑preservation evidence** (per blast‑radius surface no AC owns — *does the change break anything around it?*). The first is keyed to stable acceptance‑criterion IDs; the second is keyed to surfaces and exists to catch regression. It is *intent* — it does not run or edit anything. Test dispositions it proposes are **provisional**, confirmed against the real suite at execution by `specify-tests` (against the Behavior Baseline), Phase 3. It is per‑change, in‑repo, and travels with the change.

## Plan schema

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
    disposition: keep | change | repair | remove-recommendation   # repair = behavior-preserving re-align (default); remove-recommendation = a human-approved decision; same vocabulary as test-reconciliation

local-gate:        [ evidence that must pass locally before PR ]
ci-gate:           [ broader evidence the CI gate adds ]
non-automatable:   [ { ac-id, runtime-monitor } ]       # admitted, not faked
open-decisions:    [ escalations blocking completeness ]
```

## Derivation procedure

1. **Take** the Change Classification (types, blast radius, resolved sources) and the **Criteria IDs** (active ACs with stable IDs).
2. **Per active AC**, pull `required-evidence` from the Validation Rules for the matched change‑type(s), then **check it is *sufficient* to prove *this* criterion** — not merely that some rule applies. If the generic type‑evidence doesn't suffice for what the AC means (a story‑specific need, or an NFR/security aspect the Strategy doesn't cover), that's a **Strategy gap**: surface it and **propose a Strategy addition** (don't silently under‑test). The Strategy still wins; the plan only proposes.
3. **Map a test** to each AC: discover related existing tests **read‑only** (via the Source‑Map `tests` kind) and set `test`. Where confidence can't be proven pre‑merge, set `kind: runtime-monitor` and list it under `non-automatable` — **never fake a green test**. An AC with no test yet → `none-yet` (a coverage gap, surfaced, not hidden).
4. **Propose dispositions** for related existing tests — **provisional**:
   - AC `moved` → `change` (the test now defends an outdated wording)
   - AC `retired` → `remove`
   - AC unchanged, test exists → `keep`
   - AC new / `none-yet` → `add`
   - **Every `change`/`remove` rationale must trace to a criteria delta** — never to a test result. (Result‑driven test edits are an execution‑time concern and are forbidden as a *plan* justification.)
5. **Cover the blast radius (regression).** List each blast‑radius surface **no active AC owns** — **minimal**, the smallest sufficient set, not every transitive node. Give each a **behavior‑preservation** test: an existing test → `keep`; none yet → `add` a **regression test** (authoring handed off; provenance: the pinned baseline, *never* the change). A surface deliberately excluded gets an explicit **`out-of-scope`** note. (`internal-refactor` is the limit case: *every* surface is behavior‑preservation, since no AC moved.)
6. **Classify the existing coverage** the blast radius surfaced (`coverage-alignment`) — make the brownfield bias **visible**: each affected existing test is `criterion-aligned` (→ keep/change), `behavior-guard` (→ keep; flag upstream as a candidate criterion), or `implementation-coupled` (→ `repair` to re‑align by default; `remove-recommendation` only if it guards nothing observable). This is a **cross‑check of the plan against the test specifications** in *any* style (xUnit, **BDD/Gherkin `.feature` scenarios**, property/table): the **plan owns relevance & coverage** (criteria‑driven) — a criterion with **no** test scenario is a coverage gap (`add`); a scenario with **no** criterion is an orphan (→ `behavior-guard` or re‑align/decision). The specs check the criteria; they never define coverage. Confirmed at execution by the baseline; see the **test‑reconciliation** skill. Signal is never the end state — every entry carries its disposition.
7. **Set gates (lowest level, fail‑fast).** Choose each test at the **lowest test level that proves it** — don't reach for integration/e2e where a unit/component/contract test gives the same guarantee (brittleness control). Then split both tracks into `local-gate` / `ci-gate` by level + runnability: fast low‑level tests run **locally first**; cross‑boundary / infra‑dependent tests (integration, e2e) that can't run locally are `ci-gate`. CI‑only by necessity is **expected, not a coverage gap**.
8. **Mark provisional** — dispositions are intentions; the real reconciliation against the suite (with a behavior baseline, by `specify-tests`) happens at execution.

## Gate (readiness to capture the plan)

The plan is ready when:

- **Coverage** — every active AC has a `test` (test, runtime‑monitor, manual) or an explicit `none-yet` with a reason.
- **Sufficiency** — each AC's evidence actually *proves* it (incl. its NFR/security aspects), not just that a test exists; an AC needing evidence the Strategy doesn't cover is a **Strategy gap** (propose an extension), not a silent under‑test.
- **Disposition justification** — every proposed `change`/`remove` traces to a criteria delta.
- **Testable** — each AC is observable/verifiable as written (else it's a criteria gap → escalate as a decision).
- **Blast radius covered (regression)** — every blast‑radius surface is either owned by an active AC, carries a **behavior‑preservation** test (or explicit `none-yet`), or has an explicit `out-of-scope` note. A silently uncovered touched surface is a regression hole, not a pass.
- **Lowest sufficient level** — each test is at the lowest level that proves it; any integration/e2e is justified by what only it can verify, not chosen by default.
- **No unresolved blocking decision** — open decisions are answered, deferred, or out‑of‑scope.

Otherwise → **Not ready**: a resumable agenda; write no plan. Never partial.
