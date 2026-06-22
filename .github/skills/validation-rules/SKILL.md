---
name: validation-rules
description: >-
  The schema and derivation procedure for Validation Rules — the thin, machine-usable projection of
  the Testing Strategy, keyed by change-type, that agents act on. Used by define-testing-strategy to
  generate Rules from the Strategy, and downstream to select the evidence a classified change needs.
---

Validation Rules are the **operational layer**: a thin, structured projection of the Strategy that an agent can act on directly. The Strategy is prose for humans; the Rules are addressable by change‑type and name exactly what's required.

## The hard rule

**Rules are GENERATED from the Strategy, never hand‑edited beside it.** When the Strategy moves, regenerate. Two hand‑maintained documents drift, and the agent then enforces expectations the Strategy abandoned. Every generated rule set records the Strategy version it was derived from.

## Rule schema (one entry per change‑type)

```
change-type:        <taxonomy key>
required-evidence:  [ { id, what, level, why (traces to a confidence in the Strategy) } ]
                    # level: unit | component | contract | integration | e2e
                    #   | performance | load | failure-resilience | security-scan | a11y  (CONDITIONAL — need a declared capability; usually runtime-monitor, else out of scope)
source-kinds:       [ <kind> ]            # resolved to locations via the Source-Map
local-gate:         [ <evidence ids that must pass locally before PR> ]
ci-gate:            [ <evidence ids the broader CI gate adds> ]
risk-modifiers:     [ { when, stronger-evidence } ]   # e.g. public-contract, regulated, cross-team
non-automatable:    [ { what, runtime-witness } ]     # evidence that can't be proven pre-merge
escalates-as-decision: [ <conditions> ]   # e.g. ownership boundary, public-contract change
```

- **`required-evidence`** items carry a `why` that traces to a confidence stated in the Strategy — the rule is a projection, not a new opinion.
- **`source-kinds`** are kinds, not paths; the Source‑Map Manifest resolves them per project.
- **`escalates-as-decision`** pre‑declares the conditions that are legitimate human decisions for this type (feeds the escalation model so limitations never masquerade as decisions).
- **Level & gate placement.** Each evidence item names its **`level`** at the **lowest category that proves it** — integration/e2e only where a lower level can't (anti‑brittleness). The `local-gate`/`ci-gate` split then follows level + runnability: fast low‑level tests (unit/component/contract) go in `local-gate` to **fail fast**; cross‑boundary / infra‑dependent tests (integration, e2e) that can't run on a dev machine go in `ci-gate`. CI‑only by necessity is **expected placement, not a gap**.
- **Behavior‑preservation (regression) evidence** — a change‑type's `required-evidence` applied in a *regression lens* to a blast‑radius surface of this type that **no AC owns** (the Plan's behavior‑preservation track). The Rules name the evidence per type; the Plan decides which touched surfaces need it. No separate rule field — same evidence, asserted against the pinned baseline instead of an AC.

## Derivation procedure (Strategy → Rules)

1. For each change‑type section in the Strategy, emit one rule entry.
2. Turn each "what confidence matters" + "required evidence" line into `required-evidence` items with a `why` back‑reference.
3. Map the Strategy's source kinds into `source-kinds`.
4. Split the Strategy's local/CI roles into `local-gate` / `ci-gate`.
5. Lift risk modifiers and non‑automatable admissions verbatim into their fields.
6. Pre‑declare `escalates-as-decision` from the Strategy's ownership/contract/regulated notes.
7. Stamp the source Strategy version.

## Coverage check

A rule set is complete when **every change‑type the Strategy covers has an entry**, every `required-evidence` item traces to the Strategy, and every `source-kind` referenced exists (or is flaggable) in the Source‑Map. A change‑type present in the taxonomy but absent from the Rules is a generation gap — fix at the Strategy/derivation, not downstream.

**Generate only over a clean Strategy.** A gap (a missing/Open change‑type) or an **inconsistency** (contradictory expectations, a `source-kind` that won't resolve) is **surfaced to the human as a decision — one at a time — before generation**, never generated over. The Rules are a faithful projection; they cannot be more coherent than the Strategy they derive from, so the Strategy is made coherent first.
