---
name: source-map
description: >-
  The schema and discovery procedure for the Source-Map Manifest — how source kinds map to
  locations so agents retrieve the sources a change needs deterministically instead of guessing.
  Used whenever an agent must locate architecture, specs, schemas, tests, or CI config.
---

*Derived copy — canonical source is `Change-Validation-Playbook.md`; the fillable instance is `source-map.manifest.md`. If they disagree, the playbook wins.*

The Source‑Map makes source discovery **navigable and deterministic**. Validation Rules reference sources by **kind**; the manifest resolves a kind to a concrete **location** for this project. Agents never search blindly for a source that the map can resolve.

## Canonical kinds

`architecture` · `api-spec` · `event-schema` · `data-model` · `coding-guidelines` · `testing-strategy` · `tests` · `build-commands` · `ci-config` · `runbook` · `observability`

## Entry schema

| Field | Meaning |
|---|---|
| `kind` | canonical kind (the key Rules reference) |
| `location` | path, glob, or URL |
| `retrieval` | `repo` (path/glob) · `url` · `command:<cmd>` |
| `change-types` | which change‑types need it (`*` = all) |
| `criticality` | `critical` (missing → blocking) · `supporting` |
| `notes` | version, owner, caveats |

## Discovery procedure

1. From the classifier output, take the change‑type(s) and the `source-kinds` the matched Rules require.
2. For each needed kind, look up the manifest entries whose `change-types` include this change.
3. Retrieve via the entry's `retrieval` method.
4. **Critical + unretrievable = blocking.** Stop and report it as a *limitation* (per the escalation model) — never proceed on an assumed or invented source.
5. **Missing kind/location.** If a needed kind has no entry, surface a **proposed manifest addition** for human confirmation; do not hard‑code a path inside an agent.

## Guards

- **Resolve by kind, not by guess** — the whole point is determinism; a blind repo search defeats it.
- **Criticality is honored** — a critical source mirrors prepare‑work‑item's source guard: unretrievable is treated exactly like missing.
- **Agent‑extension is explicit** — agents propose additions; humans seed and confirm. The map stays trustworthy because nothing enters it silently.
- **`build-commands` is how the runner runs** — the (Phase 3) Execution Runner resolves its install/build/test invocations and selective‑run syntax from `build-commands`, cross‑checked against `ci-config` for local↔CI parity. It never guesses the test command any more than it guesses a file path; an unresolvable critical command is a blocking *limitation*.
