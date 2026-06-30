---
name: capture-behavior-baseline
description: >-
  The schema and capture/reconcile procedure for the Behavior Baseline — the per-change
  snapshot of current brownfield behavior that lets a behavior delta be sorted into justified (a
  criterion moved) vs regression (none did). The behavior-level enforcement of the no-edit-to-pass
  rule. Used by capture-baseline; confirms the Validation Plan's provisional dispositions. Phase 3.
---

## Inputs

- **change-ref** — the change under validation; becomes `baseline-ref`.
- **blast-radius scope** — the blast-radius surfaces to pin, from classify-change.
- **Criteria IDs** — the stable AC ids and their deltas (`moved` / new / `retired`), used to justify a behavior delta.
- **Validation Rules / strategy-version** — the Rules in force and their provenance; recorded as `strategy-version`.
- **pre-change state ref** — the pre-change commit/state to pin behavior from; becomes `captured-at`.

## Procedure

1. Capture and reconcile against the **pre-change** state and **persist** the baseline in-repo, committed, so local and CI reconcile against identical pinned behavior. Once captured, treat the baseline as **read-only** (immutable). Build the plan (which surfaces to pin, what is observable per surface, how to capture) directly; delegate the actual **capture + reconcile** to run-validation (run-execution), which runs current code over the project's own suite.
2. Scope the surfaces from the blast-radius input. Pin **only** the blast radius. Widen the scope beyond the minimal blast-radius set when test-impact analysis (step 4) finds a reachable surface — call-graph-reachable, or an `e2e`/`system` test a changed surface participates in — that the minimal set does not already include. Include blast-radius surfaces **no acceptance criterion owns** (the plan's behavior-preservation track); pinning them is how regression in untouched-but-reachable code is caught. For `internal-refactor`, every surface is of this kind. Regression coverage is bounded by what is locally observable: a surface that cannot be pinned (no infra, flaky, needs prod data) is a flagged **limitation**, never a silent hole.
3. Per surface, define the observation contract — what is observable for its change-type: HTTP response + status for `rest-api`; emitted payload for `event-producer`; persisted state and old-vs-new coexistence for `db-migration`; rendered output for `react-ui`; the contract itself for `cross-service`.
4. Pin behavior by observing current code at `captured-at`; record `pinned-behavior`. Derive each `surface-id` deterministically from the diff as the code-derived handle for the surface — endpoint path for `rest-api`, method signature for a method change, event name for `event-producer`, contract identifier for `cross-service` — recomputed per change from the diff, never read from a stored registry. Discover related existing tests **read-only** via test-impact analysis over the resolve-source-map typed `tests` entries: reachability/coverage for fine-grained tests, surface/flow participation for `e2e`/`system` (so a system-level test a changed surface participates in is pinned too, even though it is not call-graph-reachable).
5. Apply the determinism gate: do **not** baseline non-reproducible / flaky behavior. Set `determinism: quarantined(<reason>)`, quarantine the surface, and raise a **limitation** — you cannot pin what you cannot reproduce, and baselining noise would lie.
6. Post-change, re-observe each in-scope surface via run-validation and compare to `pinned-behavior`. Classify and dispose each surface per the reconciliation table in Output. A provisional `change` disposition from the Validation Plan is honest **only if** its surface's delta is `justified` here; otherwise the "change" masks a regression — the same failure review-plan's disposition-justification guard names, now caught against behavior instead of wording. A delta earns `justified` **only** by matching a `moved` / new / `retired` AC. `preserved` holds behavior stable; it does not bless it as correct. A delta that crosses a public contract / ownership boundary is a `boundary-decision`, disposed as a `structured-question` (a legitimate human decision, not a toolkit gap). A `regression` is fed back into the correction loop (run-correction-loop) as a finding, never emitted as a handoff. A surface that cannot be re-observed is a `limitation` — a toolkit gap logged, never normalized as human-in-the-loop.

## Output

Persist the baseline (capture) and its reconciliation in-repo as a committed run record.

```
baseline-ref:      <change-ref>
captured-at:       <pre-change commit/state the behavior was pinned from>
scope:             <blast-radius surfaces pinned — minimal>   # from classify-change
strategy-version:  <provenance of the Rules in force>

surfaces:                                  # one block per pinned surface in the blast radius
  - surface-id:        <code-derived handle: endpoint path | method sig | event name | contract — recomputed per change from the diff, NOT a stored registry id>
    kind:              <change-type>                  # rest-api | event-producer | db-migration | ...
    observation:       <what is observable & pinned — response+status | emitted-payload | persisted-state | rendered-output>
    pinned-behavior:   <recorded observation(s) at captured-at>       # the evidence of "current"
    determinism:       deterministic | quarantined(<reason>)          # flaky → not baselined
    related-tests:     [ <test-ref> ... ]             # existing tests defending it (read-only discovery)

reconciliation:                            # produced post-change, via run-validation (run-execution)
  - surface-id:        ...
    observed-delta:    <how post-change behavior differs from pinned-behavior, or "none">
    classification:    preserved | justified(<AC-N>) | regression | boundary-decision
    disposition:       disposition-confirmed(<plan disposition>) | recheck | structured-question | limitation
```

Reconciliation mapping — for each re-observed surface:

| Situation | Classification | Disposition |
|---|---|---|
| no behavior delta | `preserved` | confirms a `keep`; no test change earns justification |
| delta maps to an AC that `moved` / is new / `retired` in the Criteria IDs | `justified(AC-N)` | confirms the plan's provisional `change`/`add`/`remove`; recorded as evidence |
| delta with no matching criteria delta | `regression` | fed back into the loop (correction loop / finding) — never a handoff |
| delta crosses a public contract / ownership boundary | `boundary-decision` | structured question (legitimate human decision) |
| surface can't be re-observed (can't run / reproduce) | `limitation` | toolkit gap logged; never normalized as human-in-the-loop |
