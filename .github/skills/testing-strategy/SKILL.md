---
name: testing-strategy
description: >-
  The structure and coverage checklist for authoring an architecture-aware Testing Strategy — the
  human-owned source of truth for what validation evidence is expected, keyed by change-type. Used
  by define-testing-strategy while authoring/reviewing the Strategy and before generating Rules.
---

The Testing Strategy is **human‑owned** and **architecture‑aware** — it reflects the real system, not a generic one. It is the source the per‑change validation plans are *derived from*; it is not invented per story. Keep it expressed as **expected evidence per change‑type**, not as a list of tools.

It is **authored in full when none exists** (greenfield) and updated as architecture moves — always the human‑owned **source of truth**, from which the thin AI‑facing **Validation Rules** are derived. The toolkit never produces only the derived layer: the AI context is a projection of a real, reviewable, human‑owned Strategy, never a free‑standing machine artifact.

**Human‑seeded, agent‑extendable.** The Strategy is generic per change‑type, but a specific story can reveal evidence the generic rule doesn't anticipate. When it does, the toolkit **surfaces a proposed Strategy addition** (a decision for the human to approve), exactly as the Source‑Map is agent‑extendable — so the Strategy *improves from real stories* instead of silently under‑testing them. The Strategy still wins; the toolkit only proposes.

## Structure (one section per change‑type)

For each change‑type in the **change‑taxonomy**, the Strategy states:

1. **What confidence matters** — the kinds of evidence that make a change of this type trustworthy here.
2. **Required evidence** — what must be validated (the seeds of the derived Rules).
3. **Source kinds** — which sources (by kind, resolved via the Source‑Map) are needed to validate this type.
4. **Test level + tier** — the **lowest test level** that gives solid confidence for each piece of evidence, and its **support tier** (① native · ② your‑env · ③ externalized — see *Test support tiers* below). Native tests run locally first; your‑env tests run where the environment allows (else CI); externalized ones integrate or admit a runtime monitor.
5. **Risk modifiers** — when this type carries extra risk (public contract, regulated data, cross‑team ownership) requiring stronger evidence.
6. **Lowest sufficient level (anti‑brittleness)** — specify evidence at the **lowest test category that proves it**; reserve integration/e2e for what *only* they can verify (real cross‑service flow, infra behavior). Higher‑level tests are slower and **brittle** — reaching for them when a unit/component test gives the same guarantee is a cost and a reliability risk to challenge, not a default.

## Test support tiers (what the toolkit does per category)

Every test category sits in one of four tiers; the toolkit's behaviour follows the tier, and the Strategy states which evidence is which:

- **① Native (first‑class)** — `unit`, `component`. The toolkit **specifies (and verifies), runs locally, and reads results** end‑to‑end — fast, in‑process, no external infra (authoring of the test handed off). Its strongest ground.
- **② With your environment** — `integration`, `contract`, `e2e`. The toolkit **specifies and verifies** them (authoring handed off) and **runs them where your environment allows** (locally if the infra is up, else **CI**); results come from your report. It *uses* your environment, never stands one up.
- **③ External — out of scope by default** — `performance`, `load`, `failure-resilience`, `security-scan`, `a11y`. Your own pipeline/tools own these; the toolkit **does not run them and does not sit in the path** — no relay, no gate. *Optionally* (opt‑in, for audit/completeness) it **reads** your pipeline's result to record the trace and to flag an NFR that **nothing** validates — *reading only*. If you don't want even that, these are simply out of scope and it says so. **Never a man‑in‑the‑middle.**
- **④ Out of scope (declared)** — provisioning environments/harnesses, writing production code, manual/exploratory testing. The toolkit doesn't perform these. (A criterion only a human can verify is still *recorded* as a manual attestation — admitted like a `runtime-monitor`, never faked — but the testing itself is yours.)

**One line:** the toolkit's real footprint is **①②** — it *runs* those. Everything else it leaves to you; for external NFRs it offers *opt‑in* bookkeeping (a coverage flag + an audit trace), never middleware.

**BDD is a spec style, not a tier.** A Gherkin scenario is a **specification = the acceptance criterion** (human‑owned), not a test. If your project uses BDD, the toolkit's test‑specification job is to **specify (and verify) the step definitions** (derived from the scenario, never the impl — authoring handed off) and run them through your **BDD tooling** (Cucumber/SpecFlow/Behave) — it **honors the practice; it does not fork the scenario into a separate xUnit test.** The scenario sits in the tier of whatever it exercises (a unit‑level scenario is native; an e2e scenario is your‑env). Only a BDD pipeline owned *entirely* by another process — scenarios, step defs, **and** execution — is integrate‑only (③).

## Coverage checklist (the Strategy is *complete enough* when…)

Assess against these — checking what's **missing** as much as present. Each is **Met / Deferred / Open**.

1. **Every change‑type covered** — each taxonomy type has a section, or is explicitly marked not‑applicable for this system.
2. **Architecture‑grounded** — the expectations reflect this system's real properties (eventing semantics, service‑owned schemas, no cross‑service DB, CQRS, contract‑first), not generic QA.
3. **Evidence, not tools** — each entry states *what confidence*, not just "write unit tests."
4. **Source kinds named** — each type names the sources it needs (so the Source‑Map can resolve them).
5. **Tier placement (fail‑fast)** — each evidence item names its **tier** (① native · ② your‑env · ③ externalized); native tests gate locally first, your‑env ones run where the environment allows, externalized ones are integrated or admitted as runtime monitors. Order cheap→expensive so failures surface early.
6. **Test‑level discipline (anti‑brittleness)** — evidence is specified at the **lowest sufficient level**; integration/e2e are justified by what only they prove, not used by default; the brittleness/cost of over‑using high‑level tests is explicitly controlled.
7. **Risk modifiers stated** — public‑contract, regulated, and cross‑team cases call for stronger evidence.
8. **Non‑functional & security per type** — security (authz, input validation, secrets), performance/latency budgets, accessibility, and reliability are named per change‑type, so cross‑cutting concerns aren't left implicit. Most are **tier ③ — out of scope for the toolkit by default** (your pipeline/tools own them); the toolkit's only role, *if you want it*, is to **flag an NFR that nothing covers** and record an external‑evidence trace — never to run or relay them. *(A story‑specific NFR is still an acceptance criterion; this covers the ones that apply to a whole change‑type.)*
9. **Non‑automatable evidence admitted** — where confidence can't be proven pre‑merge (load, real data, non‑determinism), the Strategy says so and names the runtime‑monitor alternative — it does not pretend everything automates.
10. **Traceability expectation** — states that evidence traces back to acceptance criteria (the authoritative fixed point).

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
