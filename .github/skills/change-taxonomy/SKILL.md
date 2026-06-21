---
name: change-taxonomy
description: >-
  The canonical change-type taxonomy and classification heuristics for the change-validation
  toolkit — how to classify an incoming change into one or more types and scope its blast radius.
  Used by change-classifier and by define-testing-strategy (rules are keyed by these types).
---

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

### How affected existing tests are found (test‑impact analysis)

Blast radius is computed **from the code**, never from a stored story↔test link — so a change that touches a *previous* story's code is caught **because it reaches that code**, with nothing to maintain and nothing to go stale. The affected tests are re‑discovered every change, per **test type** (from the Source‑Map's typed `tests` entries):

- **Fine‑grained** (`unit`, `component`, `integration`) — **static reachability** (changed symbol → call/import graph) and/or a **coverage map** (`test → lines`; the diff's lines → the tests that cover them).
- **Contract** — implicated when a contract source it pins (`api-spec`/`event-schema`) is in the blast radius.
- **System / e2e** — implicated by **surface/flow participation**: any in‑scope surface that participates in a flow the test covers. Their impact is **system‑level and not call‑graph‑reachable** (they cross HTTP/queue hops), so they are found by *where they live + what flows they cover*, not by reachability. A coverage map, where available, catches every type uniformly.

The link is recomputed from the code each change (stale‑proof); no story reference is kept. This — not an AC id — is what detects impact on a prior story's tests; the **behavior‑baseline** then confirms which of them actually moved.

A blast‑radius surface that **no acceptance criterion covers** still needs a **behavior‑preservation witness** — a regression guard whose assertions come from the behavior baseline, not the change. The acceptance criteria scope the *intended‑behavior* evidence; the blast radius scopes the *unchanged‑behavior* (regression) evidence. `internal-refactor` is the case where every surface is of this second kind.

## Output schema (the Change Classification)

Consumed by `plan-validation`, `capture-baseline`, and `run-validation`.

```
change-ref:        <diff|branch|PR>
change-types:      [ <taxonomy key> ... ]                 # one or more
blast-radius:
  changed:         [ <surfaces the diff changes> ]
  dependents:      [ <transitive callers / consumers> ]
  boundaries:      [ <contract / consumer boundaries crossed> ]
affected-tests:    [ { test-ref, type: unit|component|contract|integration|e2e, via: reachability|coverage|flow } ]
source-kinds:      [ { kind, location | proposed-addition | blocking } ]     # resolved via the Source-Map
claim-authorities: [ { claim, authoritative-source, note } ]                 # contract change → decision
rule-coverage:     [ { change-type, rule | strategy-gap } ]
notes:             <refactor/baseline flags · multi-type interactions · missing-criteria note>
```
