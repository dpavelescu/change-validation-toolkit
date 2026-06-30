---
name: derive-validation-rules
description: >-
  The schema and derivation procedure for Validation Rules — the thin, machine-usable projection of
  the Testing Strategy, keyed by change-type, that agents act on. Used by author-testing-strategy to
  generate Rules from the Strategy, and downstream to select the evidence a classified change needs.
---

## Inputs

- **Testing Strategy + version** — the human-owned source of truth for expected validation evidence, keyed by change-type (from author-testing-strategy); carry its version for stamping.
- **change-taxonomy keys** — the canonical change-type keys from classify-change-type; every key the Strategy covers gets one rule entry.
- **Source-Map** — the Source-Map Manifest from resolve-source-map; resolves each source kind to project locations and to the claims it owns.

## Procedure

1. Generate Rules only from the Strategy; never hand-edit them beside it. On any Strategy change, regenerate and stamp the source Strategy version.
2. Before generating, run the coverage check: a rule set is complete when every change-type the Strategy covers has an entry, every `required-evidence` item traces to the Strategy, and every `source-kind` referenced exists (or is flaggable) in the Source-Map. A change-type present in the taxonomy but absent from the Rules is a generation gap — fix at the Strategy or derivation, not downstream.
3. Generate only over a clean Strategy. On a gap (a missing or Open change-type) or an inconsistency (contradictory expectations, a `source-kind` that will not resolve), surface it to the human as a decision — one at a time — before generation, and stop; never generate over it. The Rules are a faithful projection and cannot be more coherent than the Strategy they derive from, so make the Strategy coherent first.
4. For each change-type section in the Strategy, emit one rule entry.
5. Turn each "what confidence matters" + "required evidence" line into `required-evidence` items, each carrying a `why` that back-references the confidence stated in the Strategy — the rule is a projection, not a new opinion.
6. Map the Strategy's source kinds into `source-kinds` (kinds, not paths; resolve-source-map resolves them per project).
7. Split the Strategy's local/CI roles into `local-gate` / `ci-gate`. Name each evidence item's `level` at the lowest category that proves it — integration/e2e only where a lower level cannot (avoids brittle tests). Place fast low-level tests (unit/component/contract) in `local-gate` to fail fast; place cross-boundary or infra-dependent tests (integration, e2e) that cannot run on a dev machine in `ci-gate`. CI-only by necessity is expected placement, not a gap.
8. Lift risk modifiers and non-automatable admissions verbatim into their fields.
9. Pre-declare `escalates-as-decision` from the Strategy's ownership/contract/regulated notes — the conditions that are legitimate human decisions for this type (feeds classify-escalation so limitations never masquerade as decisions).
10. Stamp the source Strategy version on the generated rule set.

For behavior-preservation (regression) evidence: apply a change-type's `required-evidence` in a regression lens to a blast-radius surface of this type that no AC owns (the Plan's behavior-preservation track). The Rules name the evidence per type; derive-validation-plan decides which touched surfaces need it. No separate rule field — same evidence, asserted against the pinned baseline instead of an AC.

## Output

When the Strategy is clean: the generated rule set — one entry per change-type per the Rule schema — plus the stamped source Strategy version.

When the Strategy is not clean: a single decision (the surfaced gap or inconsistency) emitted instead of the rule set, blocking generation until resolved.

Rule schema (one entry per change-type):

```
change-type:        <taxonomy key>
required-evidence:  [ { id, what, level, why (traces to a confidence in the Strategy) } ]
                    # level: unit | component | contract | integration | e2e          (tier 1/2: the toolkit runs these)
                    #   | performance | load | failure-resilience | security-scan | a11y   (tier 3 externalized: integrate results / runtime monitor)
source-kinds:       [ <kind> ]            # resolved to locations via the Source-Map
local-gate:         [ <evidence ids that must pass locally before PR> ]
ci-gate:            [ <evidence ids the broader CI gate adds> ]
risk-modifiers:     [ { when, stronger-evidence } ]   # e.g. public-contract, regulated, cross-team
non-automatable:    [ { what, runtime-monitor } ]     # evidence that can't be proven pre-merge
escalates-as-decision: [ <conditions> ]   # e.g. ownership boundary, public-contract change
```
