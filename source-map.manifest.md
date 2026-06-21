# Source‑Map Manifest

*Fillable template. Copy into the consuming project (repo root) and fill the entries. Agents read this to discover the sources a change needs — deterministically, instead of guessing. Canonical schema lives in the **source‑map** skill; this file is the instance.*

## How agents use it

1. The classifier produces the change‑type(s) and blast radius.
2. The Validation Rules name the **source kinds** each applicable rule needs.
3. The agent resolves each kind to a **location** via the entries below, then retrieves it.
4. A **critical** source that can't be retrieved is **blocking** — treated like a missing one.
5. An agent that discovers a needed source not listed here **proposes an addition** (it does not silently invent a location).

## Schema (one row per source)

| Field | Meaning |
|---|---|
| `kind` | one of the canonical kinds (see below); the key the Rules reference |
| `location` | path, glob, or URL where it lives |
| `retrieval` | how to get it: `repo` (path/glob) · `url` · `command:<cmd>` |
| `change-types` | which change‑types need it (`*` = all) |
| `criticality` | `critical` (missing → blocking) · `supporting` |
| `notes` | optional — version, owner, caveats |

**Canonical kinds:** `architecture` · `api-spec` · `event-schema` · `data-model` · `coding-guidelines` · `testing-strategy` · `tests` · `ci-config` · `runbook` · `observability`

## Entries — FILL THESE IN

> The rows below are illustrative examples for a Spring Boot + React + SNS/SQS shop. Replace with your project's real locations; delete what doesn't apply.

| kind | location | retrieval | change-types | criticality | notes |
|---|---|---|---|---|---|
| architecture | `docs/architecture/**` | repo | `*` | critical | C4 + ADRs |
| api-spec | `src/main/resources/openapi/*.yaml` | repo | `rest-api`, `cross-service` | critical | contract‑first |
| event-schema | `schemas/events/**/*.json` | repo | `event-consumer`, `cross-service` | critical | SNS/SQS payload contracts |
| data-model | `src/main/resources/db/migration/**` | repo | `db-migration` | critical | Flyway/Liquibase |
| coding-guidelines | `docs/engineering/guidelines.md` | repo | `*` | supporting | |
| testing-strategy | `.github/skills/testing-strategy` + `TESTING-STRATEGY.md` | repo | `*` | critical | the human‑owned Strategy |
| tests | `src/test/**`, `**/*.spec.tsx` | repo | `*` | critical | existing suite (reconciliation target) |
| ci-config | `.github/workflows/*.yml` | repo | `*` | supporting | local↔CI parity reference |
| runbook | `docs/runbooks/**` | repo | `cross-service`, `db-migration` | supporting | |
| observability | `<dashboards/alerts url>` | url | `cross-service` | supporting | for runtime‑witness items |

## Discovery rules

- **Resolve by kind, not by guess.** If a Rule needs kind `event-schema`, take the location from this manifest — do not search the repo blindly.
- **Critical + unretrievable = blocking.** Stop and report (a *limitation* per the escalation model), do not proceed on an assumed source.
- **Proposed additions are explicit.** When a needed kind/location is absent, surface it as a proposed manifest entry for human confirmation; never hard‑code a path inside an agent.
