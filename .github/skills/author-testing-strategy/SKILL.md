---
name: author-testing-strategy
description: >-
  The structure and coverage checklist for authoring an architecture-aware Testing Strategy — the
  human-owned source of truth for what validation evidence is expected, keyed by change-type. Used
  by define-testing-strategy while authoring/reviewing the Strategy and before generating Rules.
---

## Inputs

- **change-types** — the canonical change-type taxonomy from classify-change-type; the Strategy is keyed by these, one section per type.
- **existing Strategy** — the current Testing Strategy if one exists (update it); author in full when none exists (greenfield).
- **Source-Map** — the Source-Map Manifest from resolve-source-map; resolve each named source kind through it.
- **architecture facts** — the real system's properties (eventing semantics, service-owned schemas, no cross-service DB, CQRS, contract-first, regulated data, public contracts, cross-team ownership). The Strategy reflects this system, not a generic one.

## Procedure

1. **Seed each change-type section from the defaults.** For each change-type, start from the matching architecture-aware default below, then tailor it to this system — never accept a default blindly.
   - **rest-api** — controller behavior, service logic, error mapping, backward-compatible contract changes, OpenAPI/schema consistency.
   - **event-consumer** — event-contract compatibility, deserialization, idempotency, retry/DLQ behavior, ordering assumptions, version tolerance (old + new events).
   - **event-producer** — emitted-event schema validity, consumer compatibility, at-least-once semantics.
   - **db-migration** — backward compatibility, rollback/forward strategy, old-vs-new application-version coexistence.
   - **react-ui** — component behavior, user-visible flows, accessibility-sensitive interactions, mocked API states.
   - **cross-service** — contract verification, consumer-driven expectations, end-to-end flow of the affected path; surface ownership boundaries as decisions, never auto-change them.
   - **internal-refactor** — behavior preservation against a behavior baseline (capture-behavior-baseline).

2. **State each section as expected evidence, keyed by change-type.** Express the Strategy as expected evidence per change-type, not as a list of tools. For each change-type section state:
   - **What confidence matters** — the kinds of evidence that make a change of this type trustworthy here.
   - **Required evidence** — what must be validated (the seeds the derived Rules project from).
   - **Source kinds** — which sources (by kind, resolved via the Source-Map) are needed to validate this type.
   - **Test level + tier** — the lowest test level that gives solid confidence for each piece of evidence, and its support tier (① native · ② your-env · ③ externalized — see step 3).
   - **Risk modifiers** — when this type carries extra risk (public contract, regulated data, cross-team ownership), require stronger evidence.

3. **Place each evidence item in its support tier; drive cheap-first.** Assign every test category to one of four tiers and order each section cheap→expensive so failures surface early:
   - **① Native (first-class)** — `unit`, `component`. The toolkit specifies (and verifies), runs locally, and reads results end-to-end; fast, in-process, no external infra (test authoring is handed off). Gate these locally first.
   - **② With your environment** — `integration`, `contract`, `e2e`. The toolkit specifies and verifies them (authoring handed off) and runs them where your environment allows (locally if the infra is up, else CI); results come from your report. It uses your environment; it never stands one up.
   - **③ External — out of scope by default** — `performance`, `load`, `failure-resilience`, `security-scan`, `a11y`. Your pipeline/tools own these; the toolkit does not run them and does not sit in the path — no relay, no gate. Opt-in only: read your pipeline's result to record the trace and to flag an NFR that nothing validates — reading only. Otherwise mark them out of scope.
   - **④ Out of scope (declared)** — provisioning environments/harnesses, writing production code, manual/exploratory testing. The toolkit does not perform these. Record a human-only criterion as a manual attestation (admitted like a `runtime-monitor`, never faked); the testing itself stays yours.

4. **Treat BDD as a spec style, not a tier.** Treat a Gherkin scenario as a specification equal to the acceptance criterion (human-owned), not a test. When the project uses BDD, specify (and verify) the step definitions derived from the scenario (never from the implementation; authoring handed off) and run them through your BDD tooling (Cucumber/SpecFlow/Behave). Do not fork a scenario into a separate xUnit test. Place the scenario in the tier of whatever it exercises (a unit-level scenario is native; an e2e scenario is your-env). Classify as integrate-only (③) only a BDD pipeline owned entirely by another process — scenarios, step defs, and execution.

5. **Specify each evidence item at the lowest test category that proves it.** Reserve integration/e2e for what only they can verify (real cross-service flow, infra behavior). Challenge any higher-level test that gives the same guarantee as a unit/component test — higher-level tests are slower and brittle, a cost and a reliability risk, not a default.

6. **Run the coverage checklist; mark each item Met / Deferred / Open / N-A.** The list is a baseline, not a ceiling: add any coverage concern specific to this system's architecture or domain even if unlisted; mark a listed item N-A only when it genuinely does not apply (never pad). Check what is missing as much as what is present.
   1. **Every change-type covered** — each taxonomy type has a section or is explicitly marked not-applicable for this system.
   2. **Architecture-grounded** — expectations reflect this system's real properties (eventing semantics, service-owned schemas, no cross-service DB, CQRS, contract-first), not generic QA.
   3. **Evidence, not tools** — each entry states what confidence, not just "write unit tests."
   4. **Source kinds named** — each type names the sources it needs (so the Source-Map can resolve them).
   5. **Tier placement (fail-fast)** — each evidence item names its tier (① native · ② your-env · ③ externalized); native tests gate locally first, your-env ones run where the environment allows, externalized ones are integrated or admitted as runtime monitors. Order cheap→expensive.
   6. **Test-level discipline (avoids brittle tests)** — evidence is specified at the lowest sufficient level; integration/e2e are justified by what only they prove; the brittleness/cost of over-using high-level tests is explicitly controlled.
   7. **Risk modifiers stated** — public-contract, regulated, and cross-team cases call for stronger evidence.
   8. **Non-functional & security per type** — name security (authz, input validation, secrets), performance/latency budgets, accessibility, and reliability per change-type. Most are tier ③ — out of scope for the toolkit by default (your pipeline/tools own them); the toolkit's only role, if you want it, is to flag an NFR that nothing covers and record an external-evidence trace — never to run or relay them.
   9. **Non-automatable evidence admitted** — where confidence cannot be proven pre-merge (load, real data, non-determinism), say so and name the runtime-monitor alternative; do not pretend everything automates.
   10. **Traceability expectation** — state that evidence traces back to acceptance criteria (the authoritative fixed point).

7. **Surface a proposed Strategy addition as a human decision; never apply it silently.** When a specific story reveals evidence the generic change-type rule does not anticipate, surface a proposed Strategy addition for the human to approve (as resolve-source-map surfaces source-map extensions). The Strategy is human-owned: the toolkit proposes, the human decides.

8. **Run the Gate readiness check (final step).** The Strategy is ready to generate Rules from only when every checklist item is Met or explicitly Deferred/Out-of-scope. A type left Open is a gap — close it at the Strategy, not by guessing downstream. Surface internal inconsistencies (contradictory expectations, a source kind that will not resolve) to the human one at a time; never resolve them silently. Do not generate the AI-facing Validation Rules (derive-validation-rules) over any unresolved Open gap or inconsistency.

## Output

- **Strategy document** — one section per change-type, each stating what-confidence-matters, required evidence, source kinds, test level + tier, and risk modifiers. The human-owned source of truth from which derive-validation-rules projects the thin machine-usable Rules; never a free-standing machine artifact.
- **Coverage-checklist status** — each checklist item marked Met / Deferred / Open / N-A, including added system-specific concerns.
- **Surfaced decisions and inconsistencies** — proposed Strategy additions awaiting human approval and any internal inconsistencies surfaced one at a time, with the Gate readiness verdict (ready to generate Rules, or blocked on an Open gap/inconsistency).
