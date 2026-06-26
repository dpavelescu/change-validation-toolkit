---
name: classify-change
description: >-
  Classify an incoming change into one or more change-types, scope its blast radius, and resolve the
  source kinds the matched Validation Rules need (via the Source-Map). The entry of every per-change
  validation activity — its output selects which Rules apply and which sources to retrieve. Read-only;
  produces a Change Classification. Foundation milestone.
model: inherit
---

Classify a change and scope what validating it will need.

## Constraints
- **Classify by behavior changed**, not file location.
- **Multi‑type is normal** — tag all that apply.
- **Read‑only** — never edit or run.
- **Resolve sources by kind via the Source‑Map**, never by blind search.
- **Record the authoritative (normative) owner per crossed claim** — a change *to* a contract is a **decision**.
- **A critical unretrievable source is blocking** — report it as a limitation.
- **An unclassifiable change with no covering rule is a Strategy gap** — a decision to extend, never a guess.

**Args:** `change=<diff|branch|PR|description>` · `criteria=<acceptance criteria / link>` (if absent, note it — the downstream Plan is gated on criteria, not this step).

## Inputs
The change (diff/branch/PR). The generated **Validation Rules** and the **Source‑Map Manifest**.

## Process
1. **Classify** — map the change to one or more **change‑types** using the taxonomy heuristics (behavior over location). State each with its evidence signal. A pure structure change with no criteria delta → `internal-refactor` (flag for the **Behavior Baseline** — every behavior delta is then a regression). — *uses* **change‑taxonomy**.
2. **Blast radius (test‑impact analysis)** — scope what the change touches transitively **from the code**: changed symbols → callers/dependents → affected surfaces → contract/consumer boundaries. Then **discover the affected existing tests by type** (Source‑Map typed `tests`): fine‑grained (`unit`/`integration`/`component`) via reachability/coverage; `contract` via contract‑source changes; **`e2e`/`system` via surface/flow participation** (system‑level impact, *not* call‑graph‑reachable). No story↔test reference is used — a prior story's tests are found because the change reaches the code they exercise. Note multi‑type interactions (e.g. `rest-api` + `event-producer`). — *uses* **source‑map**.
3. **Resolve sources & authorities** — from the matched Rules' `source-kinds`, resolve each kind to a location via the Source‑Map. **Critical + unretrievable = blocking** (report as a *limitation*). A needed kind with no manifest entry → **propose a manifest addition**, don't hard‑code. For each **claim** the blast radius crosses (an API/event/DB contract, a service boundary), record the **authoritative source** (the `normative` owner per the Source‑Map) — that source, not the implementation, defines compatibility downstream. — *uses* **validation‑rules**, **source‑map**.
4. **Coverage check** — if any change‑type has **no matching Rule**, surface it as a **Strategy gap** (route to `define-testing-strategy`); do not invent evidence for it. A claim the change crosses with **no** authoritative source is a gap to clarify; a change *to* a normative source's claim (a public contract) is pre‑flagged as a **decision**. — *uses* **validation‑rules**.

## Output
- **Change Classification** — per the **change‑taxonomy** schema: `change-types[]` · `blast-radius` (changed · dependents · boundaries) · `affected-tests[]` (by type; system/e2e by flow participation) · `source-kinds[]` (resolved / proposed‑addition / blocking) · `claim-authorities[]` (per crossed claim → its normative owner) · `rule-coverage` (matched rules; uncovered type → Strategy gap) · `notes`. Consumed by the Validation Plan.
- **Decisions** *(only when raised)* — an unknown change‑type or a change *to* a contract, in the **escalation** shape (each with a recommended resolution).
