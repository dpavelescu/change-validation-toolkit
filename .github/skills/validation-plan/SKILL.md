---
name: validation-plan
description: >-
  The schema and derivation procedure for the per-change Validation Plan — the AC→witness map, the
  evidence required per acceptance criterion, the provisional test fates, and the local/CI gates.
  Used by plan-validation to derive the plan and by validation-plan-reviewer to gate it. Phase 2.
---

The Validation Plan states **what evidence is needed to trust this change** along two tracks: **criterion evidence** (per active AC — *does the intended change work?*) and **behavior‑preservation evidence** (per blast‑radius surface no AC owns — *does the change break anything around it?*). The first is keyed to stable acceptance‑criterion IDs; the second is keyed to surfaces and exists to catch regression. It is *intent* — it does not run or edit anything. Test fates it proposes are **provisional**, confirmed against the real suite at execution by `implement-tests` (against the Behavior Baseline), Phase 3. It is per‑change, in‑repo, and travels with the change.

## Plan schema

```
change-ref:        <diff|branch|PR>
change-types:      [ ... ]              # from change-classifier
blast-radius:      <changed surfaces · dependents · boundaries crossed>
strategy-version:  <provenance of the Rules used>

criteria:                               # one block per ACTIVE acceptance criterion
  - ac-id:            AC-<n>
    required-evidence: [ ... ]          # from the matched Rules for the change-type(s)
    witness:          { kind: test | runtime-monitor | manual | none-yet, ref, status }
    proposed-fate:    keep | change | add | remove      # PROVISIONAL — re: related existing tests
    rationale:        <why; for change/remove MUST trace to a criteria delta (AC moved/retired)>

behavior-preservation:                  # one block per BLAST-RADIUS surface NOT owned by an active AC
  - surface-id:        <endpoint | consumer | shared method | contract | component>
    why-in-radius:     <how the change reaches it — caller, consumer, shared dependency>
    required-evidence: [ ... ]          # behavior-preservation (regression) evidence from the Rules
    witness:           { kind: test | runtime-monitor | none-yet, ref }   # provenance: the pinned baseline
    proposed-fate:     keep | add        # keep = existing witness suffices; add = author a REGRESSION witness (provenance: baseline)
out-of-scope:      [ { surface-id, why-excluded } ]     # blast-radius surfaces deliberately not witnessed

local-gate:        [ evidence that must pass locally before PR ]
ci-gate:           [ broader evidence the CI gate adds ]
non-automatable:   [ { ac-id, runtime-witness } ]       # admitted, not faked
open-decisions:    [ escalations blocking completeness ]
```

## Derivation procedure

1. **Take** the Change Classification (types, blast radius, resolved sources) and the **Criteria Identity** (active ACs with stable IDs).
2. **Per active AC**, pull `required-evidence` from the Validation Rules for the matched change‑type(s).
3. **Map a witness** to each AC: discover related existing tests **read‑only** (via the Source‑Map `tests` kind) and set `witness`. Where confidence can't be proven pre‑merge, set `kind: runtime-monitor` and list it under `non-automatable` — **never fake a green test**. An AC with no witness yet → `none-yet` (a coverage gap, surfaced, not hidden).
4. **Propose fates** for related existing tests — **provisional**:
   - AC `moved` → `change` (the test now defends an outdated wording)
   - AC `retired` → `remove`
   - AC unchanged, witness exists → `keep`
   - AC new / `none-yet` → `add`
   - **Every `change`/`remove` rationale must trace to a criteria delta** — never to a test result. (Result‑driven test edits are an execution‑time concern and are forbidden as a *plan* justification.)
5. **Cover the blast radius (regression).** List each blast‑radius surface **no active AC owns** — **minimal**, the smallest sufficient set, not every transitive node. Give each a **behavior‑preservation** witness: an existing test → `keep`; none yet → `add` a **regression witness** (provenance: the pinned baseline, *never* the change). A surface deliberately excluded gets an explicit **`out-of-scope`** note. (`internal-refactor` is the limit case: *every* surface is behavior‑preservation, since no AC moved.)
6. **Set gates (lowest level, fail‑fast).** Choose each witness at the **lowest test level that proves it** — don't reach for integration/e2e where a unit/component/contract test gives the same guarantee (brittleness control). Then split both tracks into `local-gate` / `ci-gate` by level + runnability: fast low‑level tests run **locally first**; cross‑boundary / infra‑dependent tests (integration, e2e) that can't run locally are `ci-gate`. CI‑only by necessity is **expected, not a coverage gap**.
7. **Mark provisional** — fates are intentions; the real reconciliation against the suite (with a behavior baseline, by `implement-tests`) happens at execution.

## Gate (readiness to capture the plan)

The plan is ready when:

- **Coverage** — every active AC has a `witness` (test, runtime‑monitor, manual) or an explicit `none-yet` with a reason.
- **Fate justification** — every proposed `change`/`remove` traces to a criteria delta.
- **Testable** — each AC is observable/verifiable as written (else it's a criteria gap → escalate as a decision).
- **Blast radius covered (regression)** — every blast‑radius surface is either owned by an active AC, carries a **behavior‑preservation** witness (or explicit `none-yet`), or has an explicit `out-of-scope` note. A silently uncovered touched surface is a regression hole, not a pass.
- **Lowest sufficient level** — each witness is at the lowest level that proves it; any integration/e2e is justified by what only it can verify, not chosen by default.
- **No unresolved blocking decision** — open decisions are answered, deferred, or out‑of‑scope.

Otherwise → **Not ready**: a resumable agenda; write no plan. Never partial.
