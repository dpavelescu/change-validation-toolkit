---
name: source-map
description: >-
  The schema and discovery procedure for the Source-Map Manifest — how source kinds map to
  locations AND to the claims each source is authoritative for, so agents retrieve the sources a
  change needs map-driven (a lookup, not a blind search) and resolve a claim against its owner.
  Used whenever an agent must locate or establish authority over architecture, specs, schemas, tests, or CI config.
---

The Source‑Map makes source discovery **map‑driven** (a lookup, not a blind search) — and, more fundamentally, it records **authority**: *which source is the source of truth for which category of claim, and how binding that source is*. Validation Rules reference sources by **kind**; the manifest resolves a kind to a concrete **location** and declares what that source is **authoritative for**. Agents never search blindly for a source the map can resolve, and never treat a non‑authoritative source — least of all the implementation — as the truth for a claim some authority owns.

## Canonical kinds

`architecture` · `api-spec` · `event-schema` · `data-model` · `coding-guidelines` · `testing-strategy` · `acceptance-criteria` · `tests` · `test-data` · `test-report` · `ci-config` · `runbook` · `observability`

The Source‑Map maps **what the toolkit reads** to understand your system and discover sources. It does **not** hold *how to run* (test commands) or *what capabilities exist* — those are execution concerns, handled by invoking your **own test runner / pipeline** (see the **execution‑runner** skill), not catalogued here.

## Claims & authority

A source carries claims of one **authority class** — `normative` (defines correctness — **binding**), `descriptive` (describes the system — context), or `advisory` (recommends) — and is `authoritative-for` specific **claim categories**:

`correctness` (the acceptance criteria) · `api-contract` · `event-contract` · `persistence-shape` · `service-boundaries` / `ownership` · `expected-evidence` · `conventions` · `operational-behavior`

The map answers *"who owns this claim?"* — e.g. **correctness** (the acceptance criteria) is owned by `acceptance-criteria`, the **API contract** by `api-spec`, **service boundaries** by `architecture`, **expected evidence** by `testing-strategy`. The **implementation is never authoritative** for a claim a normative source owns — that is the toolkit's invariant restated at the source layer (tests evidence, code implements; neither defines truth). Authority is **per claim, not per source**: a source is trusted only for the categories it owns, never for an aside outside them (a story governs intent and acceptance criteria, not a backend remark dropped into it).

## Entry schema

| Field | Meaning |
|---|---|
| `kind` | canonical kind (the key Rules reference) |
| `location` | path, glob, or URL |
| `retrieval` | `repo` (path/glob) · `url` · `command:<cmd>` |
| `change-types` | which change‑types need it (`*` = all) |
| `criticality` | `critical` (missing → blocking) · `supporting` |
| `authority` | `normative` (defines correctness — binding) · `descriptive` (context) · `advisory` (recommends) |
| `authoritative-for` | the claim categories this source is the canonical truth for; `—` if purely descriptive/advisory |
| `notes` | version, owner, caveats |

## Discovery procedure

1. From the classifier output, take the change‑type(s) and the `source-kinds` the matched Rules require.
2. For each needed kind, look up the manifest entries whose `change-types` include this change.
3. Retrieve via the entry's `retrieval` method.
4. **Not in the map → repo‑search fallback.** If a needed kind has no entry (or the entry doesn't resolve), **fall back to a search scoped by that kind's conventions** — its usual filenames/globs and locations (e.g. `api-spec` → `**/openapi*.{yaml,json}`, `*swagger*`; `event-schema` → schema/avro/proto dirs; `tests` → the test roots; `ci-config` → `.github/workflows`, `*.gitlab-ci.yml`). **One confident match** → surface it as a **proposed manifest addition** for confirmation; **several or none** → surface a **gap** (ask), never silently hard‑code a path. The map is preferred; the search is the fallback, and bounded — **stop when no new candidate changes the resolved set**, not a blind or endless scan.
5. **Critical + still‑unfound = blocking.** If a *critical* source is neither in the map nor findable by the fallback search, stop and report it as a *limitation* (per the escalation model) — never proceed on an assumed or invented source.
6. **Resolve a claim by authority, not by guess.** For any claim — a contract, a boundary, the expected evidence — consult the source `authoritative-for` it and treat a `normative` source as **binding**, at its **latest approved, non‑superseded** version (a superseded source is not current authority). **Cite the claim's locus**, not just the source name — `api-spec: POST /devices → 409`, `ADR‑018 §ownership`, `tests: FooTest::bar`. The **implementation is never the authority**: if impl/tests disagree with a normative source, the impl is wrong, not the source. A claim with **no** authoritative source is a **gap** (clarify, don't invent — the minimum‑clarity gate); two **normative** sources contradicting on the same claim is a **decision** (escalate, never silently pick one).

## Guards

- **Map first, search as fallback** — prefer the declared source; scoped repo search complements a missing kind.
- **Authority over implementation** — a claim is resolved against its authoritative source.
- **No authority → gap; conflicting authorities → decision** — unowned claim clarified; contradicting normative sources escalate.
- **Descriptive/advisory are context** — they inform; they never bind.
- **Latest, non‑superseded** — resolve against the current version; a superseded source isn't authority, and a derived artifact (plan/baseline/criteria set) is stale if an upstream source changed after it was captured — re‑derive, don't trust blindly.
- **Cite the locus** — anchor a claim to its exact place (file/section/endpoint), not just the source name.
- **Criticality is honored** — a critical source unretrievable is treated as missing.
- **Agent‑extension is explicit** — agents propose additions; humans seed and confirm.
- **`tests` is typed (for discovery)** — one entry per test type, each with its location and scope.
- **`test-report` is what the runner reads** — results come from the machine‑readable test report.
