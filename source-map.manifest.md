# Source‑Map Manifest

*Fillable template. Copy into the consuming project (repo root) and fill the entries. Agents read this to discover the sources a change needs — deterministically, instead of guessing. Canonical schema lives in the **source‑map** skill; this file is the instance.*

## How agents use it

1. The classifier produces the change‑type(s) and blast radius.
2. The Validation Rules name the **source kinds** each applicable rule needs.
3. The agent resolves each kind to a **location** via the entries below, then retrieves it.
4. A **critical** source that can't be retrieved is **blocking** — treated like a missing one.
5. An agent that discovers a needed source not listed here **proposes an addition** (it does not silently invent a location).
6. To settle a **claim** (a contract, a boundary, the expected evidence), the agent consults the source `authoritative-for` it and treats a `normative` source as **binding** — never the implementation.

## Schema (one row per source)

| Field | Meaning |
|---|---|
| `kind` | one of the canonical kinds (see below); the key the Rules reference |
| `location` | path, glob, or URL where it lives |
| `retrieval` | how to get it: `repo` (path/glob) · `url` · `command:<cmd>` |
| `change-types` | which change‑types need it (`*` = all) |
| `criticality` | `critical` (missing → blocking) · `supporting` |
| `authority` | `normative` (defines correctness — binding) · `descriptive` (context) · `advisory` (recommends) |
| `authoritative-for` | the claim categories this source is the canonical truth for; `—` if none |
| `notes` | optional — version, owner, caveats |

**Canonical kinds:** `architecture` · `api-spec` · `event-schema` · `data-model` · `coding-guidelines` · `testing-strategy` · `acceptance-criteria` · `tests` · `build-commands` · `ci-config` · `runbook` · `observability`

**Claim categories** (what a source can be *authoritative‑for*): `correctness` (acceptance criteria) · `api-contract` · `event-contract` · `persistence-shape` · `service-boundaries` / `ownership` · `expected-evidence` · `build-and-run` · `conventions` · `operational-behavior`. The **implementation is never authoritative** for a claim a normative source owns.

## Entries — FILL THESE IN

> The rows below are illustrative examples for a Spring Boot + React + SNS/SQS shop. Replace with your project's real locations; delete what doesn't apply.

| kind | location | retrieval | change-types | criticality | authority | authoritative-for | notes |
|---|---|---|---|---|---|---|---|
| architecture | `docs/architecture/**` | repo | `*` | critical | normative | `service-boundaries`, `ownership` | C4 + ADRs; boundary/ownership claims bind |
| api-spec | `src/main/resources/openapi/*.yaml` | repo | `rest-api`, `cross-service` | critical | normative | `api-contract` | contract‑first; the spec defines the contract, not the controller |
| event-schema | `schemas/events/**/*.json` | repo | `event-consumer`, `cross-service` | critical | normative | `event-contract` | SNS/SQS payload contracts |
| data-model | `src/main/resources/db/migration/**` | repo | `db-migration` | critical | normative | `persistence-shape` | Flyway/Liquibase |
| coding-guidelines | `docs/engineering/guidelines.md` | repo | `*` | supporting | advisory | `conventions` | recommends, never binds |
| testing-strategy | `.github/skills/testing-strategy` + `TESTING-STRATEGY.md` | repo | `*` | critical | normative | `expected-evidence` | the human‑owned Strategy |
| acceptance-criteria | `<tracker link>` or `docs/stories/<id>.md` | url or repo | `*` | critical | normative | `correctness` | the story's ACs; source‑agnostic (link or in‑repo); read‑only — humans own the content, the Criteria Identity owns the ids |
| tests (unit) | `src/test/java/**/*Test.java` | repo | `*` | critical | descriptive | `—` | scope: symbol; **runs: local (fast, first)**; found by reachability/coverage |
| tests (component) | `**/*.spec.tsx`, `**/*.test.tsx` | repo | `react-ui` | critical | descriptive | `—` | scope: component; **runs: local (fast, first)**; found by reachability/coverage |
| tests (contract) | `src/test/**/contract/**`, `pacts/**` | repo | `rest-api`, `event-consumer`, `event-producer`, `cross-service` | critical | descriptive | `—` | scope: contract; **runs: local**; implicated by `api-spec`/`event-schema` changes |
| tests (integration) | `src/test/java/**/*IT.java`, `src/test/integration/**` | repo | `*` | critical | descriptive | `—` | scope: module/service; **runs: local-if-infra-available else CI**; reachability/coverage or surface participation |
| tests (e2e) | `e2e/**`, `cypress/**` | repo | `*` | supporting | descriptive | `—` | scope: flow/system; **runs: CI (cross-boundary/infra)**; implicated by any in‑scope surface in a covered flow — NOT call‑graph reachable |
| build-commands | `./mvnw test`, `npm test`, `pom.xml`, `package.json` | command:`./mvnw -q test` | `*` | critical | normative | `build-and-run` | install/build/test + selective‑run; defers to `ci-config` for parity |
| ci-config | `.github/workflows/*.yml` | repo | `*` | supporting | normative | `build-and-run` | parity authority: local must match what CI runs |
| runbook | `docs/runbooks/**` | repo | `cross-service`, `db-migration` | supporting | descriptive | `operational-behavior` | |
| observability | `<dashboards/alerts url>` | url | `cross-service` | supporting | descriptive | `operational-behavior` | for runtime‑witness items |

## Discovery rules

- **Resolve by kind, not by guess.** If a Rule needs kind `event-schema`, take the location from this manifest — do not search the repo blindly.
- **Resolve a claim by authority.** The `authoritative-for` source owns the claim; a `normative` source binds. The implementation is never the authority. No owner → a gap to clarify; two normative sources contradicting → a decision to escalate.
- **Critical + unretrievable = blocking.** Stop and report (a *limitation* per the escalation model), do not proceed on an assumed source.
- **Proposed additions are explicit.** When a needed kind/location is absent, surface it as a proposed manifest entry for human confirmation; never hard‑code a path inside an agent.
