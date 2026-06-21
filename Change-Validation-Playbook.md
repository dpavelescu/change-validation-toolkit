# Change‑Validation Playbook

**Turn a change into the evidence needed to trust it — derived from a testing strategy, executed local‑first then in CI, and self‑correcting — so a failing test becomes loop input, not a human handoff.**

This is the canonical, tool‑neutral reference. The Copilot / Claude builds are derived from it; if they disagree, this playbook wins.

> **Scope of this milestone — Foundation.** The four foundation pieces are specified in full here: the **Testing Strategy**, the **derived Validation Rules**, the **Source‑Map Manifest**, and **Change Classification**. The later layers (Validation Plan, test reconciliation, local/CI execution, auto‑fix) are described at the level needed to keep the model coherent and are marked *forthcoming*.

---

## The operating principle (the invariant everything obeys)

For any change:

- **Criteria are authoritative.** What "correct" means for the change is the fixed point.
- **Implementation is the only freely‑mutable element.** Tests, fixtures, harness are negotiable around the criteria; the criteria are not.
- **Tests are witnesses to criteria** — never the source of truth. A test exists to defend a criterion; it earns its place by tracing to one.
- **Humans decide, they don't debug.** A human is engaged only when the *criteria themselves* are unresolvable — never handed a broken implementation to fix.

Everything below is mechanism in service of this invariant.

---

## The derivation chain (the spine)

Validation is not a phase appended after implementation. It is a control loop, derived top‑down and executed bottom‑up. Each arrow is an activity; each arrow has a **minimum‑clarity gate** in front of it — if the source for an arrow is too thin to derive honestly, the toolkit clarifies before proceeding rather than inventing.

```
architecture + technology specifics
        │  (Testing Strategy: define-testing-strategy)
        ▼
testing strategy / baseline rules
        │  (regenerate, never hand-edit beside it)
        ▼
validation rules  (thin, machine-usable, per change-type)
        │  + the change + its criteria + discovered sources        ── forthcoming
        ▼
story-level validation plan  (AC → witness map, test fates)        ── forthcoming
        │
        ▼
tests (materialized / reconciled against the existing suite)        ── forthcoming
        │
        ▼
local execution → diagnose → fix → rerun                            ── forthcoming
        │
        ▼
CI execution (same plan, wider scope, constrained autonomy)         ── forthcoming
        │
        ▼
evidence ledger → human review (behavior + decisions, not internals)
```

**Foundation covers the top three boxes** plus the cross‑cutting **Source‑Map Manifest** that every box reads from.

---

## The artifacts

| Artifact | Lifetime | Owner | Foundation? |
|---|---|---|---|
| **Testing Strategy** | stable | human | ✅ |
| **Validation Rules** (thin op layer, per change‑type) | **generated from Strategy** | tool | ✅ |
| **Source‑Map Manifest** | stable, agent‑extendable | human‑seeded | ✅ |
| **Change Classification** | per change | tool | ✅ |
| Validation Plan | per change | tool (human‑reviewed if risky) | forthcoming |
| Characterization Baseline | per change (brownfield) | tool | forthcoming |
| Evidence Ledger | per change | tool | forthcoming |

**Hard rule:** *Validation Rules is regenerated from the Strategy, never hand‑maintained beside it.* Two hand‑edited documents drift, and the agent then enforces rules the strategy has abandoned.

---

## Foundation piece 1 — Testing Strategy

The Strategy is the **human‑owned source of truth** for what evidence the organization expects, **keyed by architecture and change‑type**. It is not a generic QA document and not invented per story; it is the thing story‑level plans are *derived from*.

A Strategy is **architecture‑aware**: it must reflect the real system, not a generic one. For this context that means Java/Spring Boot microservices, a React frontend, REST APIs, SNS/SQS eventing, service‑owned schemas (no cross‑service DB access), CQRS/query services, contract‑ or schema‑based integration, a CI/CD model, and regulated‑environment traceability.

It says, per change‑type, **what confidence matters** — e.g.:

- **REST API** → controller behavior, service logic, error mapping, backward‑compatible contract changes, OpenAPI/schema consistency.
- **SNS/SQS consumer** → event‑contract compatibility, deserialization, idempotency, retry/dead‑letter, ordering assumptions, version tolerance.
- **DB migration** → backward compatibility, rollback/forward strategy, old‑vs‑new application‑version compatibility.
- **React UI** → component behavior, user‑visible flows, accessibility‑sensitive interactions, mocked API states.
- **Cross‑service behavior** → contract verification, consumer‑driven expectations, end‑to‑end flow of the affected path.

See the **testing‑strategy** skill for the authoring structure and coverage checklist, and the **change‑taxonomy** skill for the canonical change‑types.

---

## Foundation piece 2 — Validation Rules (derived)

The Rules are a **thin, machine‑usable projection** of the Strategy — the agent's operational layer. Where the Strategy is prose for humans, the Rules are structured, per change‑type, and directly actionable: *given a change of type X touching Y, the required evidence is Z, the local gate is …, the CI gate is …*.

- **Derived, not authored.** Generated from the Strategy by `define-testing-strategy`; regenerated when the Strategy moves.
- **Per change‑type**, addressable by the classifier's output.
- **Names the sources** each rule needs (by kind), which the Source‑Map resolves to locations.

See the **validation‑rules** skill for the rule schema and the derivation procedure.

---

## Foundation piece 3 — Source‑Map Manifest

A first‑class artifact that maps **source kinds → where they live → which change‑types need them**, so agents **discover the sources they need deterministically** instead of guessing. It is the "considerate of sources" discipline made navigable.

Kinds include: architecture docs, API specs (OpenAPI), event schemas, coding/testing guidelines, the Testing Strategy itself, existing‑test locations, CI config, data models, runbooks/observability.

- **Human‑seeded, agent‑extendable.** A human supplies the initial map; agents may propose additions when they discover a needed source not yet listed.
- **Criticality‑aware.** A *critical* source that can't be retrieved is **blocking** — the same rule as a missing one (mirrors prepare‑work‑item's source guard).

See the **source‑map** skill for the manifest schema and the discovery procedure, and `source-map.manifest.md` for the fillable template.

---

## Foundation piece 4 — Change Classification

For an incoming change, classify it into one or more **change‑types** and compute its **blast radius** (what the diff touches transitively). This is the entry of every per‑change activity: it selects which Validation Rules apply and, via the Source‑Map, which sources to retrieve.

- A change may be **multiple types** (e.g. an endpoint that also emits an event).
- **Blast radius drives minimality** later — the smallest sufficient evidence set, not "run everything."
- Output is structured and feeds the (forthcoming) Validation Plan.

See the **change‑taxonomy** skill for the types and classification heuristics; `change-classifier` is the agent that produces this.

---

## The escalation model (decision vs limitation)

This is the cross‑cutting principle that keeps the toolkit autonomous **without artificial human‑in‑the‑loop**. Every time a human is engaged, the reason is sorted into exactly one bin:

- **Decision (legitimate).** The criteria are ambiguous or contradictory, an ownership boundary is crossed (another team's contract), a design choice is unsafe, a public contract must change. → Emit a **structured question**. The human answers a question; they never receive broken code.
- **Limitation (illegitimate).** The toolkit can't run a test, can't reproduce a failure, can't build a fixture, lost context between local and CI, lacks access. → This is a **toolkit gap to close**, logged as such — never normalized into "a human handles this case."

A limitation must never masquerade as human‑in‑the‑loop. This is what stops the system from quietly accreting handoffs that hide its own gaps.

---

## Minimum‑clarity gate

No derivation runs on a source too thin to derive honestly. If the criteria for a change don't exist, you are not in the validation loop yet — you are in clarification (see the companion **work‑item‑preparation‑toolkit**). Foundation's gate is narrower but the same in spirit: a Strategy with no rule for a change‑type, or a Source‑Map missing a critical source, blocks the dependent step until seeded.

---

## What's forthcoming (kept coherent, not yet built)

- **Validation Plan** — per‑change derivation from Rules + criteria: AC→witness map, test fates (keep/change/add/remove), local & CI gates.
- **Test reconciliation** against the existing brownfield suite, justified by a criteria delta against a **characterization baseline** — a test changes only because a criterion moved, never because it went red.
- **Local/CI execution & auto‑fix** — the self‑closing loop; CI runs the same plan with constrained autonomy and proposes fixes as commits, never silent edits to protected branches.
- **Evidence Ledger** — the audit trail of justified test changes that makes human review fast.
