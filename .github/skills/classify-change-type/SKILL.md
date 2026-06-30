---
name: classify-change-type
description: >-
  The canonical change-type taxonomy and classification heuristics for the change-validation
  toolkit — how to classify an incoming change into one or more types and scope its blast radius.
  Use when an incoming change must be sorted into change-type(s) and have its blast radius scoped
  before any validation rule is selected. Used by classify-change and by define-testing-strategy
  (rules are keyed by these types).
---

## Inputs

- **change-ref** — the incoming change as a diff, branch, or PR reference.
- **validation-rules / strategy** — the available Validation Rules (the machine projection of the Testing Strategy), keyed by change-type.
- **source-map** — the Source-Map Manifest, used to resolve source kinds to locations and claim authorities (via resolve-source-map).
- **criteria** *(optional)* — the acceptance criteria (or a link); used to detect the criteria delta — its absence marks an `internal-refactor`, where every behavior delta is a regression.

## Procedure

1. **Classify by behavior, not location.** Determine what the diff touches and changes in behavior, not by file location alone. A change under `controller/` that only renames a private method is `internal-refactor`, not `rest-api`. Match against the change-types in the **Change-types** reference below.
2. **Tag all matching change-types.** A change may map to several types at once; tag every one that applies (an endpoint that also emits an event is `rest-api` + `event-producer`). If the change fits no type and no Strategy rule covers it, emit a Strategy gap as a *decision* to extend the taxonomy/Strategy — never guess. For `internal-refactor`, record that its evidence is behavior-preservation and flag it for the **Behavior Baseline** (capture-behavior-baseline), where every behavior delta is by definition a regression.
3. **Scope blast radius.** Scope what the change touches transitively — the callers, consumers, and contracts downstream of the diff. Record it as: changed surfaces → direct dependents → contract/consumer boundaries crossed. Find affected existing tests using the **Test-impact analysis** reference below. A blast-radius surface that no acceptance criterion covers still needs a behavior-preservation test — a regression guard whose assertions come from the behavior baseline, not the change.
4. **Resolve source-kinds.** For the matched Validation Rules and the blast radius, resolve the required source kinds to locations and claim authorities via resolve-source-map. A contract change resolves the authoritative source for the changed claim and is emitted as a *decision*.
5. **Emit the Change Classification.** Produce the schema in **## Output**, populating `change-types` (step 2), `blast-radius` and `affected-tests` (step 3), `source-kinds` and `claim-authorities` (step 4), `rule-coverage` (the Rule that applies per change-type from step 2, or a strategy-gap), and `notes` (refactor/baseline flags, multi-type interactions, missing-criteria notes).

### Change-types (reference for step 1)

1. **rest-api** — a synchronous HTTP endpoint's behavior, contract, or error mapping changes. Signals: controller/route handlers, OpenAPI/schema, request/response DTOs, status-code mapping.
2. **event-consumer** — an async SNS/SQS consumer's processing, deserialization, idempotency, retry/DLQ, or version tolerance changes. Signals: listeners/handlers, event schema, message-mapping, ordering assumptions.
3. **event-producer** — a service starts/changes the events it emits. Signals: publish calls, new/changed event schema, downstream consumers.
4. **db-migration** — schema or data migration; backward/forward compatibility and rollback. Signals: migration scripts (Flyway/Liquibase), entity changes, old-vs-new app-version coexistence (which needs two app versions running — usually a **limitation** to flag, not something pinnable locally).
5. **react-ui** — user-visible component behavior or flow changes. Signals: components, hooks, routing, API-state handling, accessibility-relevant interactions.
6. **cross-service** — behavior that spans service boundaries via contract/event/integration. Signals: consumer-driven contracts, end-to-end flow of an affected path, another team's owned component.
7. **internal-refactor** — behavior is intended to be unchanged; structure changes. Signals: no criteria delta, but a regression risk to pin.

### Test-impact analysis (reference for step 3)

Compute blast radius from the code, never from a stored story↔test link, and re-derive it every change — so a change that touches a previous story's code is caught because it reaches that code, with no stored map to go stale. This, not an AC id, is what detects impact on a prior story's tests; capture-behavior-baseline then confirms which of them actually moved. Use the strongest signal available (preferred first):

1. **Coverage map** (`test → lines`) — map the diff's lines to the tests that execute them. The toolkit generates it by invoking your runner in coverage mode. It needs per-test coverage (test-impact / per-test-coverage tooling); where only aggregate line coverage exists, an uncovered changed line is a flagged gap, and the `test → line` link falls back to (2). Per-test coverage captures dynamic wiring (DI, reflection, event/queue hops) static analysis can't see — the strongest signal where the project has per-test-coverage tooling; treat it as the best-case path, not the default everywhere.
2. **Static reachability** (changed symbol → call/import graph) — deterministic for explicit calls and imports; the fallback where per-test coverage isn't available.
3. **Contract** — a `contract` test is implicated when a contract source it pins (`api-spec`/`event-schema`) is in the blast radius.
4. **System / e2e** — caught by flow participation: which flow a test exercises comes from a coverage run over the e2e suite if available, else from the test's location, its scenario/name, and the surfaces it sets up. A changed surface that any such flow touches pulls that test in.

**Widen, never silently miss.** Signal is weak when a surface has no per-test coverage, the change crosses a boundary static analysis can't follow (DI, reflection, events/queues, HTTP), or it touches framework wiring/config. Then widen the scope — to the enclosing module, the surface's own test file(s), and the e2e flows that touch it — and flag the gap. An over-run costs runtime; an under-run risks a false green, so bias to widen.

## Output

The Change Classification, consumed by `derive-validation-plan`, `capture-behavior-baseline`, and `run-execution`.

```
change-ref:        <diff|branch|PR>
change-types:      [ <taxonomy key> ... ]                 # one or more
blast-radius:
  changed:         [ <surfaces the diff changes> ]
  dependents:      [ <transitive callers / consumers> ]
  boundaries:      [ <contract / consumer boundaries crossed> ]
affected-tests:    [ { test-ref, type: unit|component|contract|integration|e2e, via: reachability|coverage|contract|flow } ]   # e2e = system-level
source-kinds:      [ { kind, location | proposed-addition | blocking } ]     # resolved via resolve-source-map
claim-authorities: [ { claim, authoritative-source, note } ]                 # contract change → decision
rule-coverage:     [ { change-type, rule | strategy-gap } ]
notes:             <refactor/baseline flags · multi-type interactions · missing-criteria note>
```
