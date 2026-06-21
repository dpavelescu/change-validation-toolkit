---
name: criteria-identity
description: >-
  The schema and reconciliation procedure for the Criteria Identity — the per-change record that gives
  each acceptance criterion a stable id for the duration of one change's validation. Humans own the
  prose; the toolkit owns identity. Scoped to the change (not a durable, cross-change, or growing
  artifact). Used by reconcile-criteria and read by plan-validation. Phase 2.
---

The Criteria Identity gives each acceptance criterion a **stable id for one change's validation run**, so the plan, the tests, the runner, and the evidence all point at the same criterion — locally and again in CI. The human owns the *content* (prose ACs in the story); the toolkit owns *identity* (the ids).

**Scope — per‑change run state, not a durable record.** It lives with the change (on its branch) so local and CI agree on the ids, and is **discarded after merge**. It does *not* accumulate across changes or live in `main`. Two changes have independent records — nothing shared, namespaced, or grown over time. Two jobs are deliberately **not** its:

- **Cross‑change continuity.** When a *later* change touches code an earlier change's tests cover, those tests are found by the **blast radius** (test‑impact analysis) and reconciled by the **Behavior Baseline** — by surface, never by a remembered id. The criteria identity never links across changes.
- **Durable audit.** The lasting "this test validated this criterion" trail is the **Evidence Ledger**, not this.

## Entry schema (one per acceptance criterion, for this change)

```
ac-id:       AC-<n>          # stable for THIS change's run (local↔CI); not durable, not cross-change
text:        <current criterion, from the story>
status:      new | unchanged | moved | retired    # relation to the existing suite, for fate selection
source-ref:  <where in the story it came from>     # optional, read-only trace
```

No `witnesses` field — the **Validation Plan** owns the AC→witness map. No accumulating `history` — the durable audit is the Evidence Ledger.

## Reconciliation procedure (story ACs → ids)

**Read the story; never modify it.** For each AC currently in the story, assign a stable id and classify it for fate selection — against **the existing tests the change reaches** (from the blast radius), or against this change's prior run‑state when re‑running (local→CI):

| Situation | Status | Drives the plan's fate |
|---|---|---|
| an existing test already covers it, unchanged | `unchanged` | `keep` |
| an existing test covers it but the wording shifts its meaning | `moved` | `change` — **provisional** |
| nothing covers it yet | `new` | `add` |
| an existing test covers a criterion no longer in the story | `retired` | `remove` — **provisional** |
| ambiguous — same criterion reworded, or a new one? | — | **escalate as a decision** (no silent guess) |

`moved` and `retired` are **provisional within the change** — the **Behavior Baseline confirms** whether behavior actually moved (justified) or it's a regression. The id is just the handle that lets the plan, tests, and baseline talk about the same criterion.

## Guards

- **Per‑change scope** — ids are stable for one change's run (local↔CI) and discarded after merge; never a durable, cross‑change, or global record.
- **Human owns content** — the story is read‑only; the toolkit tracks the ids, never edits the criteria.
- **Provisional delta** — `moved`/`retired` are confirmed by the Behavior Baseline, not asserted here.
- **Ambiguous → decision** — a borderline "same or new?" is a structured question, never a guess.
- **Cross‑change is the baseline's job** — impact on a prior change's tests is found by blast radius + Behavior Baseline (by surface), not by this criteria identity.
- **No accumulation** — no growing history here; the durable audit trail is the Evidence Ledger.
