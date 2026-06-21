---
name: source-map
description: >-
  The schema and discovery procedure for the Source-Map Manifest — how source kinds map to
  locations AND to the claims each source is authoritative for, so agents retrieve the sources a
  change needs deterministically and resolve a claim against its owner instead of guessing.
  Used whenever an agent must locate or establish authority over architecture, specs, schemas, tests, or CI config.
---

*Derived copy — canonical source is `Change-Validation-Playbook.md`; the fillable instance is `source-map.manifest.md`. If they disagree, the playbook wins.*

The Source‑Map makes source discovery **navigable and deterministic** — and, more fundamentally, it records **authority**: *which source is the source of truth for which category of claim, and how binding that source is*. Validation Rules reference sources by **kind**; the manifest resolves a kind to a concrete **location** and declares what that source is **authoritative for**. Agents never search blindly for a source the map can resolve, and never treat a non‑authoritative source — least of all the implementation — as the truth for a claim some authority owns.

## Canonical kinds

`architecture` · `api-spec` · `event-schema` · `data-model` · `coding-guidelines` · `testing-strategy` · `acceptance-criteria` · `tests` · `build-commands` · `ci-config` · `runbook` · `observability`

## Claims & authority

A source carries claims of one **authority class** — `normative` (defines correctness — **binding**), `descriptive` (describes the system — context), or `advisory` (recommends) — and is `authoritative-for` specific **claim categories**:

`correctness` (the acceptance criteria) · `api-contract` · `event-contract` · `persistence-shape` · `service-boundaries` / `ownership` · `expected-evidence` · `build-and-run` · `conventions` · `operational-behavior`

The map answers *"who owns this claim?"* — e.g. **correctness** (the acceptance criteria) is owned by `acceptance-criteria`, the **API contract** by `api-spec`, **service boundaries** by `architecture`, **expected evidence** by `testing-strategy`. The **implementation is never authoritative** for a claim a normative source owns — that is the toolkit's invariant restated at the source layer (tests witness, code implements; neither defines truth).

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
4. **Critical + unretrievable = blocking.** Stop and report it as a *limitation* (per the escalation model) — never proceed on an assumed or invented source.
5. **Missing kind/location.** If a needed kind has no entry, surface a **proposed manifest addition** for human confirmation; do not hard‑code a path inside an agent.
6. **Resolve a claim by authority, not by guess.** For any claim — a contract, a boundary, the expected evidence — consult the source `authoritative-for` it and treat a `normative` source as **binding**. The **implementation is never the authority**: if impl/tests disagree with a normative source, the impl is wrong, not the source. A claim with **no** authoritative source is a **gap** (clarify, don't invent — the minimum‑clarity gate); two **normative** sources contradicting on the same claim is a **decision** (escalate, never silently pick one).

## Guards

- **Resolve by kind, not by guess** — the whole point is determinism; a blind repo search defeats it.
- **Authority over implementation** — a claim is resolved against its authoritative source; the implementation is **never** authoritative for a claim a normative source owns (the invariant, at the source layer).
- **No authority → gap; conflicting authorities → decision** — an unowned claim is clarified, not invented; two normative sources that contradict on a claim escalate as a decision.
- **Descriptive/advisory are context** — they inform; they never bind. Don't let a runbook or a guideline overrule a normative contract.
- **Criticality is honored** — a critical source mirrors prepare‑work‑item's source guard: unretrievable is treated exactly like missing.
- **Agent‑extension is explicit** — agents propose additions; humans seed and confirm. The map stays trustworthy because nothing enters it silently.
- **`build-commands` is how the runner runs** — the (Phase 3) Execution Runner resolves its install/build/test invocations and selective‑run syntax from `build-commands`, cross‑checked against `ci-config` for local↔CI parity. It never guesses the test command any more than it guesses a file path; an unresolvable critical command is a blocking *limitation*.
