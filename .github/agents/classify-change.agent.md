---
name: classify-change
description: >-
  Classify an incoming change into one or more change-types, scope its blast radius, and resolve the
  source kinds the matched Validation Rules need (via the Source-Map). The entry of every per-change
  validation activity — its output selects which Rules apply and which sources to retrieve. Read-only;
  produces a Change Classification. Foundation milestone. Not for deriving the Validation Plan — that
  is plan-validation — or for running or editing anything.
model: inherit
tools: ["read", "search"]
---

## Constraints
- **Classify by behavior changed**, not file location.
- **Multi-type is normal** — tag all that apply.
- **Read-only** — never edit or run.
- **Resolve sources by kind via the Source-Map**, never by blind search.
- **Record the authoritative (normative) owner per crossed claim** — a change *to* a contract is a **decision**.
- **A critical unretrievable source is blocking** — report it as a limitation.
- **An unclassifiable change with no covering rule is a Strategy gap** — a decision to extend, never a guess.

## Inputs
- **Args:** `change=<diff|branch|PR|description>` · `criteria=<acceptance criteria / link>` (if absent, note it — the downstream Plan is gated on criteria, not this step).
- The change (diff/branch/PR).
- The acceptance **criteria** (drives the criteria-delta classification in step 1).
- The generated **Validation Rules** and the **Source-Map Manifest**.

## Process
1. **Classify** — map the change to one or more **change-types** using the taxonomy heuristics (behavior over location). State each with its evidence signal. A pure structure change with no criteria delta (assessed from the **criteria** input) → `internal-refactor` (flag for the **capture-behavior-baseline** — every behavior delta is then a regression). — *uses* **classify-change-type**.
2. **Blast radius (test-impact analysis)** — scope what the change touches transitively **from the code**: changed symbols → callers/dependents → affected surfaces → contract/consumer boundaries. Then **discover the affected existing tests by type** (Source-Map typed `tests`): fine-grained (`unit`/`integration`/`component`) via reachability/coverage; `contract` via contract-source changes; **`e2e`/`system` via surface/flow participation** (system-level impact, *not* call-graph-reachable). No story↔test reference is used — a prior story's tests are found because the change reaches the code they exercise. Note multi-type interactions (e.g. `rest-api` + `event-producer`). — *uses* **resolve-source-map**.
3. **Resolve sources & authorities** — from the matched Rules' `source-kinds`, resolve each kind to a location via the Source-Map. **Critical + unretrievable = blocking** (report as a *limitation*). A needed kind with no manifest entry → **propose a manifest addition**, don't hard-code. For each **claim** the blast radius crosses (an API/event/DB contract, a service boundary), record the **authoritative source** (the `normative` owner per the Source-Map) — that source, not the implementation, defines compatibility downstream. — *uses* **resolve-source-map** (the **Validation Rules** are read as an input here, not generated).
4. **Coverage check** — if any change-type has **no matching Rule**, surface it as a **Strategy gap** (route to `define-testing-strategy`); do not invent evidence for it. A claim the change crosses with **no** authoritative source is a gap to clarify; a change *to* a normative source's claim (a public contract) is pre-flagged as a **decision**. (Coverage is checked against the **Validation Rules** input — an uncovered change-type → a Strategy gap.)

## Output format
- **Change Classification** — per the **classify-change-type** schema: `change-types[]` · `blast-radius` (changed · dependents · boundaries) · `affected-tests[]` (by type; system/e2e by flow participation) · `source-kinds[]` (resolved / proposed-addition / blocking) · `claim-authorities[]` (per crossed claim → its normative owner) · `rule-coverage` (matched rules; uncovered type → Strategy gap) · `notes`. Consumed by the Validation Plan.
- **Decisions** *(only when raised)* — an unknown change-type or a change *to* a contract, in the **classify-escalation** shape (each with a recommended resolution).
- **Limitations** *(only when raised)* — a critical unretrievable source, in the **classify-escalation** `limitation` shape.
