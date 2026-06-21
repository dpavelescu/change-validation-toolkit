---
name: testing-strategy
description: >-
  The structure and coverage checklist for authoring an architecture-aware Testing Strategy — the
  human-owned source of truth for what validation evidence is expected, keyed by change-type. Used
  by define-testing-strategy while authoring/reviewing the Strategy and before generating Rules.
---

The Testing Strategy is **human‑owned** and **architecture‑aware** — it reflects the real system, not a generic one. It is the source the per‑change validation plans are *derived from*; it is not invented per story. Keep it expressed as **expected evidence per change‑type**, not as a list of tools.

It is **authored in full when none exists** (greenfield) and updated as architecture moves — always the human‑owned **source of truth**, from which the thin AI‑facing **Validation Rules** are derived. The toolkit never produces only the derived layer: the AI context is a projection of a real, reviewable, human‑owned Strategy, never a free‑standing machine artifact.

## Structure (one section per change‑type)

For each change‑type in the **change‑taxonomy**, the Strategy states:

1. **What confidence matters** — the kinds of evidence that make a change of this type trustworthy here.
2. **Required evidence** — what must be validated (the seeds of the derived Rules).
3. **Source kinds** — which sources (by kind, resolved via the Source‑Map) are needed to validate this type.
4. **Test level + local/CI placement** — the **lowest test level** that gives solid confidence for each piece of evidence, and where it runs: fast, low‑level tests (unit, component, contract) run **locally first** for early failure; cross‑boundary / infra‑dependent tests (integration, e2e) that can't run on a dev machine belong to the **CI** gate. Order cheap→expensive so problems surface before any CI‑level run.
5. **Risk modifiers** — when this type carries extra risk (public contract, regulated data, cross‑team ownership) requiring stronger evidence.
6. **Lowest sufficient level (anti‑brittleness)** — specify evidence at the **lowest test category that proves it**; reserve integration/e2e for what *only* they can verify (real cross‑service flow, infra behavior). Higher‑level tests are slower and **brittle** — reaching for them when a unit/component test gives the same guarantee is a cost and a reliability risk to challenge, not a default.

## Coverage checklist (the Strategy is *complete enough* when…)

Assess against these — checking what's **missing** as much as present. Each is **Met / Deferred / Open**.

1. **Every change‑type covered** — each taxonomy type has a section, or is explicitly marked not‑applicable for this system.
2. **Architecture‑grounded** — the expectations reflect this system's real properties (eventing semantics, service‑owned schemas, no cross‑service DB, CQRS, contract‑first), not generic QA.
3. **Evidence, not tools** — each entry states *what confidence*, not just "write unit tests."
4. **Source kinds named** — each type names the sources it needs (so the Source‑Map can resolve them).
5. **Local/CI roles distinct (fail‑fast)** — fast low‑level tests run locally first; cross‑boundary/infra tests are the CI gate; the order is cheap→expensive so failures surface early.
6. **Test‑level discipline (anti‑brittleness)** — evidence is specified at the **lowest sufficient level**; integration/e2e are justified by what only they prove, not used by default; the brittleness/cost of over‑using high‑level tests is explicitly controlled.
7. **Risk modifiers stated** — public‑contract, regulated, and cross‑team cases call for stronger evidence.
8. **Non‑automatable evidence admitted** — where confidence can't be proven pre‑merge (load, real data, non‑determinism), the Strategy says so and names the runtime‑witness alternative — it does not pretend everything automates.
9. **Traceability expectation** — states that evidence traces back to acceptance criteria (the authoritative fixed point).

## Architecture‑aware defaults (starting expectations — tailor, don't accept blindly)

- **rest-api** — controller behavior, service logic, error mapping, **backward‑compatible contract changes**, OpenAPI/schema consistency.
- **event-consumer** — event‑contract compatibility, deserialization, **idempotency**, retry/DLQ behavior, ordering assumptions, **version tolerance** (old + new events).
- **event-producer** — emitted‑event schema validity, consumer compatibility, at‑least‑once semantics.
- **db-migration** — backward compatibility, rollback/forward strategy, **old‑vs‑new application‑version coexistence**.
- **react-ui** — component behavior, user‑visible flows, accessibility‑sensitive interactions, mocked API states.
- **cross-service** — contract verification, consumer‑driven expectations, end‑to‑end flow of the affected path; **ownership boundaries are decisions, not auto‑changes**.
- **internal-refactor** — behavior preservation against a behavior baseline.

## Gate

The Strategy is ready to generate Rules from when every checklist item is Met or explicitly Deferred/Out‑of‑scope. A type left **Open** is a gap that will surface later as an unclassifiable change with no rule — close it at the Strategy, not by guessing downstream. Internal **inconsistencies** (contradictory expectations, a source kind that won't resolve) are surfaced to the human the same way — **one at a time** — never silently resolved, and the AI‑facing Rules are never generated over an unresolved gap or inconsistency.
