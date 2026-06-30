---
name: resolve-source-map
description: >-
  The schema and discovery procedure for the Source-Map Manifest — how source kinds map to
  locations AND to the claims each source is authoritative for, so agents retrieve the sources a
  change needs map-driven (a lookup, not a blind search) and resolve a claim against its owner.
  Used whenever an agent must locate or establish authority over architecture, specs, schemas, tests, or CI config.
---

## Inputs

- **change-type(s)** — from the classifier output: the classified change-type(s) and the `source-kinds` the matched Validation Rules require.
- **source-kinds** — the canonical kinds each Rule references. Resolve every named kind; treat the canonical set as closed.

Canonical kinds:

`architecture` · `api-spec` · `event-schema` · `data-model` · `coding-guidelines` · `testing-strategy` · `acceptance-criteria` · `tests` · `test-data` · `test-report` · `ci-config` · `runbook` · `observability`

Map only **what the toolkit reads** to understand the system and discover sources. Do not catalogue *how to run* (test commands) or *what capabilities exist* — those are execution concerns; invoke the project's own test runner / pipeline (see the **run-execution** skill) for them.

Each source carries claims of one **authority class** — `normative` (defines correctness, binding), `descriptive` (describes the system, context), or `advisory` (recommends) — and is `authoritative-for` specific **claim categories**:

`correctness` (the acceptance criteria) · `api-contract` · `event-contract` · `persistence-shape` · `service-boundaries` / `ownership` · `expected-evidence` · `conventions` · `operational-behavior`

Resolve authority **per claim, not per source**: trust a source only for the categories it owns — `correctness` from `acceptance-criteria`, `api-contract` from `api-spec`, `service-boundaries` from `architecture`, `expected-evidence` from `testing-strategy`. Never trust an aside outside a source's owned categories (a story governs intent and acceptance criteria, not a backend remark dropped into it). Never treat the implementation as authoritative for a claim a normative source owns.

## Procedure

1. Take the change-type(s) and the required `source-kinds` from the classifier output.
2. For each needed kind, look up the manifest entries whose `change-types` include this change.
3. Retrieve via the entry's `retrieval` method.
4. Not in the map: fall back to a repo search scoped by that kind's conventions — its usual filenames/globs and locations (e.g. `api-spec` → `**/openapi*.{yaml,json}`, `*swagger*`; `event-schema` → schema/avro/proto dirs; `tests` → the test roots; `ci-config` → `.github/workflows`, `*.gitlab-ci.yml`). On **one confident match**, surface it as a **proposed manifest addition** for confirmation. On **several or none**, surface a **gap** (ask); never silently hard-code a path. Prefer the map; use the search only as fallback, and bound it — stop when no new candidate changes the resolved set, never a blind or endless scan.
5. If a **critical** source is neither in the map nor findable by the fallback, stop and report a **blocking limitation** (per the classify-escalation model). Never proceed on an assumed or invented source.
6. Resolve every claim against its `authoritative-for` source, never the implementation. Treat a `normative` source as binding at its **latest approved** version; an older, replaced source is not current authority, and a derived artifact (plan/baseline/criteria set) is stale if an upstream source changed after it was captured — re-derive, do not trust blindly. Cite the claim's **exact place**, not just the source name — `api-spec: POST /devices → 409`, `ADR-018 §ownership`, `tests: FooTest::bar`. If impl/tests disagree with a normative source, the impl is wrong. A claim with **no** authoritative source is a **gap** (clarify, do not invent). Two **normative** sources contradicting on the same claim is a **decision** (escalate, never silently pick one).

## Output

A resolved source set plus, per claim, a cited authority. Each manifest entry has this shape:

| Field | Meaning |
|---|---|
| `kind` | canonical kind (the key Rules reference) |
| `location` | path, glob, or URL |
| `retrieval` | `repo` (path/glob) · `url` · `command:<cmd>` |
| `change-types` | which change-types need it (`*` = all) |
| `criticality` | `critical` (missing → blocking) · `supporting` |
| `authority` | `normative` (defines correctness, binding) · `descriptive` (context) · `advisory` (recommends) |
| `authoritative-for` | the claim categories this source is the canonical truth for; `—` if purely descriptive/advisory |
| `notes` | version, owner, caveats |

Emit exactly one of these deterministic outcomes per resolution:

- **resolved source + citation** — the kind resolved to a location and the claim is cited at its exact place against the authoritative source.
- **proposed manifest addition** — one confident fallback match, surfaced for human confirmation before use.
- **gap** — a needed kind with no entry and no confident match, or a claim with no authoritative source; ask, do not invent.
- **decision** — two normative sources contradict on the same claim; escalate, never silently pick one.
- **blocking limitation** — a critical source is neither mapped nor findable; stop and report.
