---
name: capture-baseline
description: >-
  Pin a change's current brownfield behavior across its blast radius, then reconcile post-change
  behavior against it — sorting each delta into justified (a criterion moved) or regression (none
  did) — to enforce the no-edit-to-pass rule that a test changes only because a criterion moved. Capture and
  reconcile delegate to run-validation; the baseline plan is advisory. Phase 3.
model: inherit
agents: ['run-validation']
---

Pin **current observable behavior** of this change's blast‑radius surfaces, then sort every post‑change behavior delta into **justified** (maps to a criteria delta) or **regression** (none did). **House rules:** the pre‑change baseline is the authoritative record of "what current behavior was" — **immutable once captured**, never widened to swallow a delta; a delta earns *justified* **only** by matching a `moved`/new/`retired` AC in the Criteria IDs, otherwise it is a regression (**fed back into the loop, never a handoff**); **can't‑capture is a limitation** (toolkit gap), never normalized as human‑in‑the‑loop; pin the **blast radius only** (minimal).

**Args:** `change=<diff|branch|PR>` · `classification=<path>` (blast radius + change‑types) · `plan=<path>` (the `behavior-preservation` track — the surfaces to pin) · `criteria-ids=<path>` (criteria deltas) · `baseline=<path>` (default `.validation/<change>/baseline.md`).

## Inputs (retrieve, don't assume)
The Validation Plan's **`behavior-preservation` track** (the authoritative list of surfaces to guard), the Change Classification (blast radius, change‑types), the Criteria IDs' **delta summary** (`moved`/new/`retired` — the justification keys), the Source‑Map (surfaces, tests, contracts), and the pre‑change state at `captured-at`. Skills and agents are cited per step below.

## Process (scope → pin → reconcile → dispose)
1. **Scope** — pin exactly the **Plan's `behavior-preservation` surfaces** (plus any AC‑owned surface whose disposition needs a before/after), reconciled against the classification's blast radius — minimal, smallest sufficient, never "pin everything." If the plan and blast radius disagree on a surface, surface it (don't silently widen or narrow). Per surface define its observation contract from its change‑type. — *uses* **change‑taxonomy**, **behavior‑baseline**.
2. **Pin** — drive the project's suite at `captured-at`, record observations as `pinned-behavior`; discover related tests **read‑only** by **test‑impact analysis** (reachability/coverage for fine‑grained; surface/flow participation for `e2e`/`system`). A surface reported flaky is **quarantined**, not baselined → carry its **limitation**. — *uses* **behavior‑baseline**, `run-validation`.
3. **Reconcile** — re‑observe each surface; sort the delta: `preserved` · `justified(AC‑N)` (matches a criteria delta) · `regression` (no match) · `boundary-decision` (crosses a public contract / ownership boundary). — *uses* **behavior‑baseline**, `run-validation`.
4. **Dispose** — `justified` → **confirms** the Validation Plan's provisional disposition, recorded as evidence; `regression` → **fed back into the loop**; `boundary-decision` → **structured question**; can't re‑observe → **limitation**. Write the baseline + reconciliation in‑repo, committed. — *uses* **escalation**, **output‑style**.

## Output
- **Behavior Baseline + reconciliation record** — per the **behavior‑baseline** schema: `justified[]` (AC + the disposition it confirms) · `regressions[]` (fed back into the loop). For `specify-tests` and the loop.
- **Decisions & limitations** — per the **escalation** shape: a `boundary-decision` (the question · context · recommended resolution · owning authority · what it blocks) or a `limitation` (the gap · what it blocked · what would close it).

## Guards
Baseline‑immutability (read‑only once captured; never widened post‑hoc) · justification‑by‑criteria‑delta (`justified` only on a `moved`/new/`retired` match) · regression‑is‑recheck (never a handoff; contract/ownership crossing → decision) · can't‑capture → limitation (toolkit gap, never normalized) · minimality (blast radius only) · refactor‑is‑sharpest (`internal-refactor` has no criteria delta → every delta is a regression; baseline is its primary evidence) · determinism (flaky → quarantine + limitation, never silently baselined) · persisted (written so local and CI share identical pinned behavior).
