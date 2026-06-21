---
name: capture-baseline
description: >-
  Pin a change's current brownfield behavior across its blast radius, then reconcile post-change
  behavior against it — sorting each delta into justified (a criterion moved) or regression (none
  did) — to enforce the honesty rule that a test changes only because a criterion moved. Capture and
  reconcile delegate to run-validation; the baseline plan is advisory. Phase 3.
model: inherit
agents: ['run-validation']
---

Pin **current observable behavior** of this change's blast‑radius surfaces, then sort every post‑change behavior delta into **justified** (maps to a criteria delta) or **regression** (none did). **House rules:** the pre‑change baseline is the authoritative record of "what current behavior was" — **immutable once captured**, never widened to swallow a delta; a delta earns *justified* **only** by matching a `moved`/new/`retired` AC in the Criteria Identity, otherwise it is a regression (**loop input, never a handoff**); **can't‑capture is a limitation** (toolkit gap), never normalized as human‑in‑the‑loop; pin the **blast radius only** (minimal).

**Args:** `change=<diff|branch|PR>` · `classification=<path>` (blast radius + change‑types) · `plan=<path>` (the `behavior-preservation` track — the surfaces to pin) · `criteria-ids=<path>` (criteria deltas) · `baseline=<path>` (default `.validation/<change>/baseline.md`).

## Inputs (retrieve, don't assume)
The Validation Plan's **`behavior-preservation` track** (the authoritative list of surfaces to guard), the Change Classification (blast radius, change‑types), the Criteria Identity's **delta summary** (`moved`/new/`retired` — the justification keys), the Source‑Map (surfaces, tests, contracts), and the pre‑change state at `captured-at`. Apply the **behavior‑baseline** and **change‑taxonomy** skills. *(Capture and reconcile delegate to **`run-validation`** — the execution substrate that drives the project's own suite; step 1 is usable without running.)*

## Process (scope → pin → reconcile → dispose)
1. **Scope** — pin exactly the **Plan's `behavior-preservation` surfaces** (plus any AC‑owned surface whose fate needs a before/after), reconciled against the classification's blast radius — minimal, smallest sufficient, never "pin everything." If the plan and blast radius disagree on a surface, surface it (don't silently widen or narrow). Per surface define its observation contract from its change‑type.
2. **Pin** *(capture — delegate to `run-validation`)* — drive the project's suite at `captured-at` via **`run-validation`**, record its observations as `pinned-behavior` per the **behavior‑baseline** schema; discover related tests **read‑only** by **test‑impact analysis** (reachability/coverage for fine‑grained; surface/flow participation for `e2e`/`system`). A surface `run-validation` reports as flaky is **quarantined**, not baselined → carry its **limitation**.
3. **Reconcile** *(post‑change — re‑run via `run-validation`)* — re‑observe each surface; sort the delta: `preserved` · `justified(AC‑N)` (matches a criteria delta) · `regression` (no match) · `boundary-decision` (crosses a public contract / ownership boundary).
4. **Dispose** — `justified` → **confirms** the Validation Plan's provisional fate, recorded as evidence; `regression` → **loop input**; `boundary-decision` → **structured question**; can't re‑observe → **limitation**. Write the baseline + reconciliation in‑repo, committed.

## Output (the Behavior Baseline + a delta reconciliation)
The baseline (pinned behavior per **behavior‑baseline** schema) and a **reconciliation**: `justified[]` (each with the AC it maps to and the plan fate it confirms), `regressions[]` (loop input), `boundary-decisions[]` (structured questions), `limitations[]` (quarantined/uncapturable — toolkit gaps).

## Guards
Baseline‑immutability (read‑only once captured; never widened post‑hoc) · justification‑by‑criteria‑delta (`justified` only on a `moved`/new/`retired` match) · regression‑is‑loop‑input (never a handoff; contract/ownership crossing → decision) · can't‑capture → limitation (toolkit gap, never normalized) · minimality (blast radius only) · refactor‑is‑sharpest (`internal-refactor` has no criteria delta → every delta is a regression; baseline is its primary evidence) · determinism (flaky → quarantine + limitation, never silently baselined) · persisted (written so local and CI share identical pinned behavior).
