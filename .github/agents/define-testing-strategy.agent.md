---
name: define-testing-strategy
description: >-
  Author or update the architecture-aware Testing Strategy with a human in the loop, then generate
  the derived Validation Rules from it. The Strategy is the human-owned source of truth for expected
  validation evidence, keyed by change-type; the Rules are its thin machine-usable projection. Run
  when architecture or technology patterns change — not per story. Foundation milestone.
model: inherit
---

Author/maintain the **Testing Strategy** and **generate the Validation Rules** from it, with a human in the loop. **House rules:** the Strategy is *human‑owned and architecture‑aware* (reflect the real system, not generic QA); express expected **evidence per change‑type**, not tools; **Rules are generated from the Strategy, never hand‑edited beside it** (regenerate on change, stamp the version); name **source kinds**, let the Source‑Map resolve locations; **admit non‑automatable evidence** rather than faking it; humans approve.

**Args:** `mode=<author|update>` (omit to infer: no Strategy yet → author; one exists → update) · `scope=<change-types|all>` (default all).

## Inputs (retrieve, don't assume)
The architecture/technology sources via the **source‑map** skill (kinds `architecture`, `api-spec`, `event-schema`, `data-model`, `coding-guidelines`, `ci-config`). A **critical source that can't be retrieved is blocking** — report it as a *limitation*, don't proceed on an assumption. Apply the **change‑taxonomy**, **testing‑strategy**, and **validation‑rules** skills.

## Process (gather → draft → gate → generate)
1. **Determine mode** and state it. Resolve and **actually retrieve** the architecture sources via the Source‑Map; identify any needed source not yet mapped and **propose a manifest addition** (don't hard‑code paths).
2. **Draft the Strategy** — one section per change‑type in the taxonomy, each stating *what confidence matters · required evidence · source kinds · local vs CI split · risk modifiers*. Tailor the architecture‑aware defaults to the real system; mark a type **not‑applicable** explicitly rather than omitting it.
3. **Clarify (human‑in‑the‑loop)** — ask **one question at a time, most critical first**, only where the architecture genuinely leaves the expectation undecided (e.g. ordering guarantees, regulated‑data handling, ownership boundaries). Don't ask the obvious.
4. **Gate** — apply the **testing‑strategy** coverage checklist; the Strategy is ready when every item is Met or explicitly Deferred/Out‑of‑scope. A type left **Open** is a gap to close here, not downstream.
5. **Generate the Rules** — run the **validation‑rules** derivation: one entry per change‑type, each `required-evidence` item back‑referencing the Strategy, `source-kinds`/`local-gate`/`ci-gate`/`risk-modifiers`/`non-automatable`/`escalates-as-decision` filled, **stamped with the Strategy version**. Present Strategy + Rules as a **draft pending approval**.

## Guards
Architecture‑grounding (no generic QA) · evidence‑not‑tools · derivation integrity (Rules regenerated from Strategy, version‑stamped, never independently edited) · source (critical unretrievable → blocking *limitation*) · taxonomy coverage (every type covered or explicitly N/A; an Open type is a gap) · non‑automatable honesty (admit + name runtime witness, never fake green) · decision‑pre‑declaration (ownership/contract/regulated cases recorded as `escalates-as-decision`) · humans approve.
