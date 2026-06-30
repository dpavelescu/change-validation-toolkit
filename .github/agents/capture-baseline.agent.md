---
name: capture-baseline
description: >-
  Pin a change's current brownfield behavior across its blast radius, then reconcile post-change
  behavior against it — sorting each delta into justified (a criterion moved) or regression (none
  did) — to enforce the no-edit-to-pass rule that a test changes only because a criterion moved. Capture and
  reconcile delegate to run-validation; the baseline plan is advisory. Phase 3.
model: inherit
agents: ['run-validation']
tools: ["read", "edit"]
---

## Constraints
- **The pre-change baseline is immutable once captured** — the authoritative record of "what current behavior was," never widened to swallow a delta.
- **A delta earns `justified` only by matching a `moved`/new/`retired` AC** in the Criteria IDs; otherwise it is a regression — fed back into the loop, never a handoff.
- **Can't-capture is a limitation** (toolkit gap), never normalized as human-in-the-loop.
- **Pin the blast radius only** — minimal.
- **An `internal-refactor` has no criteria delta, so every delta is a regression**, and the baseline is its primary evidence.
- **A flaky surface is quarantined and a limitation**, never silently baselined.
- **Persist the baseline in-repo** so local and CI share identical pinned behavior.

## Inputs
**Args:** `change=<diff|branch|PR>` · `classification=<path>` (blast radius + change-types + `strategy-version`) · `plan=<path>` (the `behavior-preservation` track — the surfaces to pin) · `criteria-ids=<path>` (criteria deltas) · `pre-change-state=<commit|ref>` (default the change's merge-base) · `baseline=<path>` (default `.validation/<change>/baseline.md`).

The Validation Plan's **`behavior-preservation` track** (the authoritative list of surfaces to guard), the Change Classification (blast radius, change-types, the `strategy-version` of the Rules in force), the Criteria IDs' **delta summary** (`moved`/new/`retired` — the justification keys), the Source-Map (surfaces, tests, contracts), and the pre-change state (`pre-change-state`) pinned as `captured-at`.

## Process
1. **Scope** — pin exactly the **Plan's `behavior-preservation` surfaces** (plus any AC-owned surface whose disposition needs a before/after), reconciled against the classification's blast radius — minimal, smallest sufficient, never "pin everything." If the plan and blast radius disagree on a surface, raise a `boundary-decision` via **classify-escalation** and, until it resolves, pin the union of the two conservatively (don't silently widen or narrow past that). Per surface define its observation contract from its change-type (read from the classification). — *uses* **capture-behavior-baseline**.
2. **Pin** — drive the project's suite at `captured-at`, record observations as `pinned-behavior`; discover related tests **read-only** by test-impact analysis (reachability/coverage for fine-grained; surface/flow participation for `e2e`/`system`). A surface reported flaky is **quarantined**, not baselined → carry its **limitation**. — *uses* **capture-behavior-baseline**, `run-validation`.
3. **Reconcile** — re-observe each surface; sort the delta: `preserved` · `justified(AC-N)` (matches a criteria delta) · `regression` (no match) · `boundary-decision` (crosses a public contract / ownership boundary). — *uses* **capture-behavior-baseline**, `run-validation`.
4. **Dispose** — `justified` → **confirms** the Validation Plan's provisional disposition, recorded as evidence; `regression` → **fed back into the loop**; `boundary-decision` → a **structured question** (once answered, re-reconcile the surface as `preserved`/`justified`/`regression` — it does not dead-end); can't re-observe → **limitation**. Write the baseline + reconciliation in-repo, committed. — *uses* **classify-escalation**, **shape-output**.

**Done** when every in-scope surface is **either** pinned-and-reconciled (classified `preserved` · `justified` · `regression` · `boundary-decision` and disposed) **or** carried as a quarantine / can't-re-observe **limitation** — a partial-coverage run still terminates. **Give up** only if no in-scope surface can be pinned at all (the suite can't run, or every surface is flaky/un-observable): emit the limitation set and stop — do not loop.

## Output format
- **Behavior Baseline + reconciliation** — per the **capture-behavior-baseline** schema: the pinned `surfaces[]`, and a `reconciliation[]` entry per surface whose `classification` is `preserved` · `justified(AC-N)` (confirms the plan disposition) · `regression` (fed back into the loop) · `boundary-decision`. For `specify-tests` and the loop.
- **Decisions & limitations** — a `boundary-decision` or a `limitation`, each per the **classify-escalation** shape (a decision carries a recommended resolution).
