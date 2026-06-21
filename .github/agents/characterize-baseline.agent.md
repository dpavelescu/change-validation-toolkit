---
name: characterize-baseline
description: >-
  Pin a change's current brownfield behavior across its blast radius, then reconcile post-change
  behavior against it — sorting each delta into justified (a criterion moved) or regression (none
  did) — to enforce the honesty rule that a test changes only because a criterion moved. Capture and
  reconcile need the execution runner; the baseline plan is advisory. Phase 3.
model: inherit
---

Pin **current observable behavior** of this change's blast‑radius surfaces, then sort every post‑change behavior delta into **justified** (maps to a criteria delta) or **regression** (none did). **House rules:** the pre‑change baseline is the authoritative record of "what current behavior was" — **immutable once captured**, never widened to swallow a delta; a delta earns *justified* **only** by matching a `moved`/new/`retired` AC in the Ledger, otherwise it is a regression (**loop input, never a handoff**); **can't‑capture is a limitation** (toolkit gap), never normalized as human‑in‑the‑loop; characterize the **blast radius only** (minimal).

**Args:** `change=<diff|branch|PR>` · `classification=<path>` (blast radius + change‑types) · `ledger=<path>` (criteria deltas) · `baseline=<path>` (default `.validation/<change>/baseline.md`).

## Inputs (retrieve, don't assume)
The Change Classification (blast radius, change‑types), the Criteria Ledger's **delta summary** (`moved`/new/`retired` — the justification keys), the Source‑Map (surfaces, tests, contracts), and the pre‑change state at `captured-at`. Apply the **characterization‑baseline** and **change‑taxonomy** skills. *(Capture and reconcile require the forthcoming execution runner; steps 2–3 are specified for when it lands — step 1 and the plan are usable now.)*

## Process (scope → pin → reconcile → dispose)
1. **Scope** — take the minimal surface set from the blast radius (smallest sufficient; never "characterize everything"). Per surface define its observation contract from its change‑type.
2. **Pin** *(capture — needs runner)* — observe current code at `captured-at`, record `pinned-behavior` per the **characterization‑baseline** schema; discover related tests **read‑only**. A **non‑deterministic** surface is **quarantined**, not baselined → raise a **limitation**.
3. **Reconcile** *(post‑change — needs runner)* — re‑observe each surface; sort the delta: `preserved` · `justified(AC‑N)` (matches a criteria delta) · `regression` (no match) · `boundary-decision` (crosses a public contract / ownership boundary).
4. **Dispose** — `justified` → **confirms** the Validation Plan's provisional fate, recorded as evidence; `regression` → **loop input**; `boundary-decision` → **structured question**; can't re‑observe → **limitation**. Write the baseline + reconciliation in‑repo, committed.

## Output (the Characterization Baseline + a delta reconciliation)
The baseline (pinned behavior per **characterization‑baseline** schema) and a **reconciliation**: `justified[]` (each with the AC it maps to and the plan fate it confirms), `regressions[]` (loop input), `boundary-decisions[]` (structured questions), `limitations[]` (quarantined/uncapturable — toolkit gaps).

## Guards
Baseline‑immutability (read‑only once captured; never widened post‑hoc) · justification‑by‑criteria‑delta (`justified` only on a `moved`/new/`retired` match) · regression‑is‑loop‑input (never a handoff; contract/ownership crossing → decision) · can't‑capture → limitation (toolkit gap, never normalized) · minimality (blast radius only) · refactor‑is‑sharpest (`internal-refactor` has no criteria delta → every delta is a regression; baseline is its primary evidence) · determinism (flaky → quarantine + limitation, never silently baselined) · persisted (written so local and CI share identical pinned behavior).
