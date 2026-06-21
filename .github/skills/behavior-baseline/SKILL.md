---
name: behavior-baseline
description: >-
  The schema and capture/reconcile procedure for the Behavior Baseline — the per-change
  snapshot of current brownfield behavior that lets a behavior delta be sorted into justified (a
  criterion moved) vs regression (none did). The behavior-level enforcement of the honesty
  rule. Used by capture-baseline; confirms the Validation Plan's provisional fates. Phase 3.
---

The Behavior Baseline pins **current observable behavior** of a change's blast‑radius surfaces *before* the change, so that after the change every **behavior delta** can be sorted into **justified** (it maps to a criteria delta — an AC `moved`, new, or `retired`) or **regression** (no criterion moved). It is the **behavior‑level** enforcement of the honesty rule — *a test changes only because a criterion moved, never because it went red* — where the Criteria Identity matches at the level of *wording* and the plan reviewer at the level of *intent*. It is per‑change, tool‑owned, captured against the **pre‑change** state, and **persisted in‑repo and committed** so local and CI reconcile against identical pinned behavior.

**Execution boundary.** Pinning behavior requires **running current code** — the toolkit's first execution touch. The **plan** (which surfaces to pin, what is observable per surface, how to capture) is advisory and buildable now; **capture + reconcile** delegate to the **Execution Runner** (`run-validation` / the **execution‑runner** skill), which drives the project's own suite.

## Schema

```
baseline-ref:      <change-ref>
captured-at:       <pre-change commit/state the behavior was pinned from>
scope:             <blast-radius surfaces pinned — minimal>   # from classify-change
strategy-version:  <provenance of the Rules in force>

surfaces:                                  # one block per pinned surface in the blast radius
  - surface-id:        <code-derived handle: endpoint path | method sig | event name | contract — recomputed per change from the diff, NOT a stored registry id>
    kind:              <change-type>                  # rest-api | event-consumer | db-migration | ...
    observation:       <what is observable & pinned — response+status | emitted-payload | persisted-state | rendered-output>
    pinned-behavior:   <recorded observation(s) at captured-at>       # the witness of "current"
    determinism:       deterministic | quarantined(<reason>)          # flaky → not baselined
    related-witnesses: [ <test-ref> ... ]             # existing tests defending it (read-only discovery)

reconciliation:                            # produced post-change, via the Execution Runner (run-validation)
  - surface-id:        ...
    observed-delta:    <how post-change behavior differs from pinned-behavior, or "none">
    classification:    preserved | justified(<AC-N>) | regression | boundary-decision
    disposition:       fate-confirmed(<plan fate>) | loop-input | structured-question | limitation
```

## Capture procedure (pre‑change)

1. **Scope** the surfaces from the blast radius — **minimal**, the smallest sufficient set, never "pin everything." This includes blast‑radius surfaces **no acceptance criterion owns** (the plan's behavior‑preservation track) — pinning them is exactly how regression in untouched‑but‑reachable code is caught. (`internal-refactor` is the limit case: every surface is of this kind.)
2. **Per surface**, define the observation contract: what is observable for its change‑type (HTTP response + status for `rest-api`; emitted payload for `event-producer`; persisted state and old‑vs‑new coexistence for `db-migration`; rendered output for `react-ui`; the contract itself for `cross-service`).
3. **Pin** behavior by observing current code at `captured-at`; record `pinned-behavior`. Discover related existing tests **read‑only** via **test‑impact analysis** over the Source‑Map's typed `tests` entries — reachability/coverage for fine‑grained tests, **surface/flow participation for `e2e`/`system`** (so a system‑level test a changed surface participates in is pinned too, even though it isn't call‑graph‑reachable).
4. **Determinism gate** — non‑reproducible / flaky behavior is **not** baselined: quarantine the surface and raise a **limitation** (you cannot pin what you cannot reproduce; baselining noise would lie).

## Reconciliation procedure (post‑change)

Re‑observe each in‑scope surface and compare to `pinned-behavior`:

| Situation | Classification | Disposition |
|---|---|---|
| no behavior delta | `preserved` | confirms a `keep`; no test change earns justification |
| delta maps to an AC that `moved` / is new / `retired` in the Criteria Identity | **`justified(AC‑N)`** | confirms the plan's provisional `change`/`add`/`remove`; recorded as evidence |
| delta with **no** matching criteria delta | **`regression`** | **loop input** (correction loop / finding) — never a handoff |
| delta crosses a public contract / ownership boundary | **`boundary-decision`** | structured question (legitimate human decision) |
| surface can't be re‑observed (can't run / reproduce) | **`limitation`** | toolkit gap logged; never normalized as human‑in‑the‑loop |

A provisional `change` fate from the Validation Plan is **honest only if its surface's delta is `justified` here**; otherwise the "change" is laundering a regression — the same failure the plan reviewer's fate‑justification guard names, now caught against *behavior* instead of *wording*.

## Guards

- **Baseline immutability** — once captured the baseline is read‑only; never widen it after the fact to make a delta look "expected" (the behavior analogue of *no silent supersession*).
- **Justification by criteria delta** — a delta earns `justified` only by matching a `moved` / new / `retired` AC in the Criteria Identity; absent that match it is a `regression`.
- **Preserved ≠ endorsed** — pinning current behavior holds it **stable** (changing it silently would be a regression), but does **not** bless it as correct. A surface guarded by a witness yet owned by **no criterion** is flagged **`guarded-but-unspecified`** — a candidate criterion routed upstream — so latent bias/bugs surface instead of being silently treated as "correct."
- **Regression is loop input, not a handoff** — an unjustified delta feeds the correction loop / surfaces as a finding — a toolkit task, never normalized as human‑in‑the‑loop — *unless* it crosses a public contract / ownership boundary, which is a **decision** (structured question).
- **Can't capture → limitation** — can't run / build / reproduce is a toolkit gap, logged as such, never a handoff.
- **Minimality** — pin the blast radius only.
- **Refactor is sharpest** — `internal-refactor` carries no criteria delta by definition, so the baseline is its **primary** evidence: every behavior delta is, necessarily, a regression.
- **Persisted** — written in‑repo so local and CI reconcile against identical pinned behavior.
