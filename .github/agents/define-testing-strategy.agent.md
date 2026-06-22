---
name: define-testing-strategy
description: >-
  Author or update the architecture-aware Testing Strategy with a human in the loop, then generate
  the derived Validation Rules from it. The Strategy is the human-owned source of truth for expected
  validation evidence, keyed by change-type; the Rules are its thin machine-usable projection. Run
  when architecture or technology patterns change — not per story. Foundation milestone.
model: inherit
---

Author/maintain the **Testing Strategy** and **generate the Validation Rules** from it, with a human in the loop. **House rules:** the Strategy is *human‑owned and architecture‑aware* (reflect the real system, not generic QA); express expected **evidence per change‑type**, not tools; **when no Strategy exists, author the full human‑owned Strategy first** (never only the thin AI‑facing Rules) — the Strategy is the source of truth, the Rules are its derived projection; **Rules are generated from the Strategy, never hand‑edited beside it** (regenerate on change, stamp the version); name **source kinds**, let the Source‑Map resolve locations; **admit non‑automatable evidence** rather than faking it; surface every **gap or inconsistency to the human one question at a time**, never generate over it; humans approve.

**Args:** `mode=<author|update>` (omit to infer: no Strategy yet → author; one exists → update) · `scope=<change-types|all>` (default all).

## Inputs (retrieve, don't assume)
The architecture/technology sources via the **source‑map** skill (kinds `architecture`, `api-spec`, `event-schema`, `data-model`, `coding-guidelines`, `ci-config`). A **critical source that can't be retrieved is blocking** — report it as a *limitation*, don't proceed on an assumption. Apply the **change‑taxonomy**, **testing‑strategy**, **validation‑rules**, and **escalation** skills (gaps/inconsistencies surface as structured decisions; an unretrievable critical source as a structured limitation).

## Process (gather → draft → gate → generate)
1. **Determine mode** and state it. Resolve and **actually retrieve** the architecture sources via the Source‑Map; identify any needed source not yet mapped and **propose a manifest addition** (don't hard‑code paths).
2. **Draft the Strategy** — one section per change‑type in the taxonomy, each stating *what confidence matters · required evidence · source kinds · local vs CI split · risk modifiers*. Tailor the architecture‑aware defaults to the real system; mark a type **not‑applicable** explicitly rather than omitting it. **When none exists (greenfield)**, author it in full from the architecture — expect more genuinely‑open expectations, so clarify more (step 3); the result is the **human‑owned source of truth**, not a machine artifact. (If even the architecture sources are absent, that's a blocking *limitation* — the Strategy is derived *from* the architecture; clarify it first.)
3. **Clarify (human‑in‑the‑loop)** — ask **one question at a time, most critical first**, only where the architecture genuinely leaves the expectation undecided (e.g. ordering guarantees, regulated‑data handling, ownership boundaries). Don't ask the obvious. The same one‑at‑a‑time prompting carries **gaps and inconsistencies** found while gating/deriving (steps 4–5).
4. **Gate** — apply the **testing‑strategy** coverage checklist; the Strategy is ready when every item is Met or explicitly Deferred/Out‑of‑scope. A type left **Open**, or an internal **inconsistency** (contradictory expectations, an unresolvable source kind), is surfaced to the human **here** — one at a time — not closed by guessing.
5. **Generate the Rules** — **only over a clean Strategy:** before generating, surface any remaining **gap or inconsistency as a decision, one at a time**; never generate the AI‑facing Rules over an unresolved one. Then run the **validation‑rules** derivation: one entry per change‑type, each `required-evidence` item back‑referencing the Strategy, `source-kinds`/`local-gate`/`ci-gate`/`risk-modifiers`/`non-automatable`/`escalates-as-decision` filled, **stamped with the Strategy version**. Present Strategy + Rules as a **draft pending approval**.

## Output
The **Testing Strategy** (human‑owned prose, per the **testing‑strategy** structure) + the derived **Validation Rules** (per the **validation‑rules** schema, version‑stamped) — a **draft pending approval**; plus any `decision` (gap/inconsistency) or `limitation` (unretrievable source) raised, per the **escalation** skill.

## Guards
Architecture‑grounding (no generic QA) · evidence‑not‑tools · create‑full‑strategy (none exists → author the human‑owned Strategy, then derive Rules — never only the AI layer) · derivation integrity (Rules regenerated from Strategy, version‑stamped, never independently edited) · gap/inconsistency → prompt (surfaced to the human one‑at‑a‑time before generating; never generated over) · source (critical unretrievable → blocking *limitation*) · taxonomy coverage (every type covered or explicitly N/A; an Open type is a gap) · lowest‑sufficient‑level (challenge integration/e2e where a unit/component/contract test gives the same guarantee; control brittleness) · local‑first placement (fast low‑level tests gate locally first; cross‑boundary/infra tests are CI) · non‑automatable honesty (admit + name runtime monitor, never fake green) · decision‑pre‑declaration (ownership/contract/regulated cases recorded as `escalates-as-decision`) · humans approve.
