---
name: validation-plan
description: >-
  The schema and derivation procedure for the per-change Validation Plan — the AC→witness map, the
  evidence required per acceptance criterion, the provisional test fates, and the local/CI gates.
  Used by plan-validation to derive the plan and by validation-plan-reviewer to gate it. Phase 2.
---

*Derived copy — canonical source is `Change-Validation-Playbook.md`; if they disagree, the playbook wins.*

The Validation Plan states **what evidence is needed to trust this change**, keyed to stable acceptance‑criterion IDs. It is *intent* — it does not run or edit anything. Test fates it proposes are **provisional**, confirmed against the real suite at execution by `implement-tests` (against the Characterization Baseline), Phase 3. It is per‑change, in‑repo, and travels with the change.

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

local-gate:        [ evidence that must pass locally before PR ]
ci-gate:           [ broader evidence the CI gate adds ]
non-automatable:   [ { ac-id, runtime-witness } ]       # admitted, not faked
open-decisions:    [ escalations blocking completeness ]
```

## Derivation procedure

1. **Take** the Change Classification (types, blast radius, resolved sources) and the **Criteria Ledger** (active ACs with stable IDs).
2. **Per active AC**, pull `required-evidence` from the Validation Rules for the matched change‑type(s).
3. **Map a witness** to each AC: discover related existing tests **read‑only** (via the Source‑Map `tests` kind) and set `witness`. Where confidence can't be proven pre‑merge, set `kind: runtime-monitor` and list it under `non-automatable` — **never fake a green test**. An AC with no witness yet → `none-yet` (a coverage gap, surfaced, not hidden).
4. **Propose fates** for related existing tests — **provisional**:
   - AC `moved` → `change` (the test now defends an outdated wording)
   - AC `retired` → `remove`
   - AC unchanged, witness exists → `keep`
   - AC new / `none-yet` → `add`
   - **Every `change`/`remove` rationale must trace to a criteria delta** — never to a test result. (Result‑driven test edits are an execution‑time concern and are forbidden as a *plan* justification.)
5. **Set gates** — split required evidence into `local-gate` / `ci-gate` per the Rules.
6. **Mark provisional** — fates are intentions; the real reconciliation against the suite (with a characterization baseline, by `implement-tests`) happens at execution.

## Gate (readiness to capture the plan)

The plan is ready when:

- **Coverage** — every active AC has a `witness` (test, runtime‑monitor, manual) or an explicit `none-yet` with a reason.
- **Fate justification** — every proposed `change`/`remove` traces to a criteria delta.
- **Testable** — each AC is observable/verifiable as written (else it's a criteria gap → escalate as a decision).
- **Blast radius covered** — dependents and crossed boundaries have evidence or an explicit out‑of‑scope note.
- **No unresolved blocking decision** — open decisions are answered, deferred, or out‑of‑scope.

Otherwise → **Not ready**: a resumable agenda; write no plan. Never partial.
