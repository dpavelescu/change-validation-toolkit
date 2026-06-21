---
name: change-taxonomy
description: >-
  The canonical change-type taxonomy and classification heuristics for the change-validation
  toolkit — how to classify an incoming change into one or more types and scope its blast radius.
  Used by change-classifier and by define-testing-strategy (rules are keyed by these types).
---

*Derived copy — canonical source is `Change-Validation-Playbook.md`; if they disagree, the playbook wins.*

The change‑types are the **key** the Testing Strategy and Validation Rules are organized by. A change may map to **several** types at once. Classify by what the diff *touches and changes in behavior*, not by file location alone.

## Change‑types

1. **rest-api** — a synchronous HTTP endpoint's behavior, contract, or error mapping changes. Signals: controller/route handlers, OpenAPI/schema, request/response DTOs, status‑code mapping.
2. **event-consumer** — an async SNS/SQS consumer's processing, deserialization, idempotency, retry/DLQ, or version tolerance changes. Signals: listeners/handlers, event schema, message‑mapping, ordering assumptions.
3. **event-producer** — a service starts/changes the events it emits. Signals: publish calls, new/changed event schema, downstream consumers.
4. **db-migration** — schema or data migration; backward/forward compatibility and rollback. Signals: migration scripts (Flyway/Liquibase), entity changes, old‑vs‑new app‑version coexistence.
5. **react-ui** — user‑visible component behavior or flow changes. Signals: components, hooks, routing, API‑state handling, accessibility‑relevant interactions.
6. **cross-service** — behavior that spans service boundaries via contract/event/integration. Signals: consumer‑driven contracts, end‑to‑end flow of an affected path, another team's owned component.
7. **internal-refactor** — behavior is intended to be unchanged; structure changes. Signals: no criteria delta, but a regression risk to pin.

## Classification heuristics

- **Behavior over location.** A change under `controller/` that only renames a private method is `internal-refactor`, not `rest-api`.
- **Multi‑type is normal.** An endpoint that also emits an event is `rest-api` + `event-producer`. Tag all that apply.
- **Refactor needs a baseline.** `internal-refactor` carries no criteria delta, so its evidence is *behavior‑preservation* — flag it for the **Behavior Baseline**, where every behavior delta is, by definition, a regression (see the **behavior‑baseline** skill).
- **Unknown type → blocking gap.** If the change fits no type and no Strategy rule covers it, that's a Strategy gap (a *decision* to extend the taxonomy/Strategy), not a guess.

## Blast radius

Scope **what the change touches transitively** — the callers, consumers, and contracts downstream of the diff. Blast radius drives later minimality (smallest sufficient evidence set) and tells the Source‑Map which sources to retrieve. Record it as: changed surfaces → direct dependents → contract/consumer boundaries crossed.

A blast‑radius surface that **no acceptance criterion covers** still needs a **behavior‑preservation witness** — a regression guard whose assertions come from the behavior baseline, not the change. The acceptance criteria scope the *intended‑behavior* evidence; the blast radius scopes the *unchanged‑behavior* (regression) evidence. `internal-refactor` is the case where every surface is of this second kind.

## Output (consumed by Validation Rules + the forthcoming Plan)

`change-types[]` · `blast-radius` (changed surfaces, dependents, boundaries) · `source-kinds-needed[]` (from the matched Rules) · `notes` (refactor/baseline flags, multi‑type interactions).
