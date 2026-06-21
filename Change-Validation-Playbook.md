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
        │  + the change + its criteria + discovered sources
        ▼
story-level validation plan  (AC → witness map, test fates)
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

**Foundation** covers the top three boxes and the cross‑cutting **Source‑Map Manifest** that every box reads from; **Phase 2** adds the **Criteria Ledger** and the story‑level **Validation Plan**. From *tests* down is still forthcoming.

---

## The artifacts

| Artifact | Lifetime | Owner | Foundation? |
|---|---|---|---|
| **Testing Strategy** | stable | human | ✅ |
| **Validation Rules** (thin op layer, per change‑type) | **generated from Strategy** | tool | ✅ |
| **Source‑Map Manifest** | stable, agent‑extendable | human‑seeded | ✅ |
| **Change Classification** | per change | tool | ✅ |
| **Criteria Ledger** (AC identity, tool‑managed) | per change, persisted | tool | ✅ (Phase 2) |
| **Validation Plan** | per change | tool (human‑reviewed if risky) | ✅ (Phase 2) |
| **Characterization Baseline** | per change (brownfield) | tool | ✅ (Phase 3 — specified; capture via the runner) |
| **Execution Runner** (run record) | per change / run | tool | ✅ (Phase 3 — specified; first piece that runs) |
| **Test Reconciliation** (witnesses + record) | per change | tool (independent test‑implementer) | ✅ (Phase 3 — specified) |
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

## Criteria identity — the ledger

The criteria are the authoritative fixed point, so tests must trace to them by a **stable identifier** (`AC‑1`, `AC‑2`…). But **humans do not maintain those IDs** — hand‑maintaining immutable IDs across reworded, reordered, split, or merged stories is fragile and would silently lie the moment it slipped.

Instead, **content and identity are split:**

- **The human owns content** — writes acceptance criteria in prose, in the story (tracker or in‑repo), edits freely, never types an ID.
- **The toolkit owns identity** — assigns and maintains the IDs in a tool‑managed **Criteria Ledger** (in‑repo, committed, travels with the change). The story stays the source of truth for *what the criteria say*; the ledger is the source of truth for *which criterion is which*.

On every change the toolkit reconciles the story's current ACs against the ledger — the **same supersession machinery as the work‑item‑preparation‑toolkit's answer ledger**:

| Situation | Action | ID outcome |
|---|---|---|
| matches an existing AC (even reworded) | match | **same ID preserved** |
| reworded enough to change meaning | match + flag | same ID, **marked "moved"** |
| genuinely new | assign | new ID |
| removed from story | retire | ID retired |
| ambiguous (same criterion reworded, or new?) | **escalate as a decision** | human answers a question |

Two consequences make this more than bookkeeping:

- **The ID lifecycle *is* the criteria‑delta detector.** "Did this criterion move?" — the signal test reconciliation depends on — falls out of the match step for free.
- **Humans never write the test tag either.** When the toolkit generates or updates a test, it stamps the `AC‑N` tag from the ledger. IDs are immutable not by discipline but because only the tool assigns them, and it only matches / adds / retires — never renumbers.

(Annotation convention for tests — native tag `@Tag("AC‑N")` / `[AC‑N]`, extracted by one `AC‑[0-9]+` regex — is specified with the forthcoming execution layer; the human is not in that loop.)

## Minimum‑clarity gate

No derivation runs on a source too thin to derive honestly. If the criteria for a change don't exist, you are not in the validation loop yet — you are in clarification (see the companion **work‑item‑preparation‑toolkit**). Foundation's gate is narrower but the same in spirit: a Strategy with no rule for a change‑type, or a Source‑Map missing a critical source, blocks the dependent step until seeded.

---

## Phase 3 — Characterization Baseline (the honesty rule, made checkable)

The invariant says *tests are witnesses to criteria* and a test may change **only because a criterion moved**. Phase 2 enforces that at the level of *wording* (the Criteria Ledger's `moved` flag) and *intent* (the plan reviewer rejects any `change`/`remove` fate not tracing to a criteria delta). Neither has looked at **behavior**. The Characterization Baseline closes that gap: it pins **current observable behavior** of the change's blast‑radius surfaces *before* the change, so that after the change every **behavior delta** sorts into exactly one of:

- **Justified** — the delta maps to a criteria delta (an AC `moved`, new, or `retired`). The behavior changed because a criterion moved; the witnessing test legitimately changes. This **confirms** the Validation Plan's provisional `change`/`add`/`remove` fate — now against *fact*, not just intent.
- **Regression** — the behavior changed but **no criterion moved**. This is **loop input** — it feeds the (forthcoming) auto‑fix loop or surfaces as a finding — **never a human handoff**. (One exception: a delta crossing a public contract or ownership boundary is a legitimate **decision** → structured question.)

This is the **auto‑fix honesty rule** made checkable: *a test goes green by moving with a criterion, never by being edited because it went red.* An `internal-refactor` — which carries no criteria delta by definition — makes the rule sharpest: every behavior delta is, necessarily, a regression, so the baseline *is* a refactor's primary evidence.

It reuses machinery already in the toolkit rather than inventing parallel concepts: the **blast radius** (from Change Classification) scopes what to pin — minimal, smallest sufficient set; the **Source‑Map** resolves where surfaces, tests, and contracts live; the **match / justified‑move / regression** sort is the same supersession idiom as the Criteria Ledger, run over *behaviors* instead of *criterion text*.

**Lifetime & honesty.** The baseline is per‑change, tool‑owned, captured against the **pre‑change** state, and **persisted in‑repo and committed** so local and CI reconcile against identical pinned behavior. Once captured it is **immutable** — you never widen the baseline after the fact to make a regression look "expected" (the behavior analogue of *no silent supersession*).

**The execution boundary.** Pinning behavior requires **running current code** — the toolkit's first execution touch. So the piece splits: **planning the baseline** (which surfaces, what is observable, how to capture) is advisory and lands now, like the rest of the toolkit; **capture and reconcile** delegate to the **Execution Runner** (below). Capture over **non‑deterministic** behavior is refused, not faked — a flaky surface is quarantined and raised as a **limitation** (you cannot pin what you cannot reproduce), never silently baselined into noise.

See the **characterization‑baseline** skill for the schema and the capture/reconcile procedure; `characterize-baseline` is the agent that produces it.

---

## Phase 3 — Execution Runner (the first thing that runs)

Up to here every artifact is advisory — it proposes, it never runs or edits. The Characterization Baseline needs *observed* behavior, so something must finally execute. That is the **Execution Runner**: the substrate that drives **the project's own suite** — resolved through the Source‑Map (the `build-commands` kind, parity‑checked against `ci-config`), never a harness the toolkit invents — over the change's blast‑radius slice, and returns **structured observations**: what ran, what each result witnesses about behavior, and whether it reproduces. It **runs but does not edit** — observation only; what to *do* with a result belongs to the baseline's reconciliation and the forthcoming auto‑fix loop.

One distinction defines the layer, and it is the operational form of the whole "no artificial handoffs" stance applied to execution outcomes:

- A **clean fail** — the test ran and asserted false — is **loop input**, a behavior signal. It is never escalated to a human. *(This is the playbook's founding line — "a failing test becomes loop input, not a human handoff" — made literal.)*
- A **can't‑run** — couldn't build, run, or resolve the command — is a **limitation**, a toolkit gap logged as such. It is never normalized into "a human handles this case."

A red test and a broken harness look identical at a glance; conflating them is exactly how a toolkit quietly accretes handoffs that hide its own gaps. The runner refuses to — it sorts every non‑pass into **signal or gap**.

Two more disciplines keep it honest. **Minimality** — it runs the blast‑radius slice via selective invocation, never the whole suite. **Determinism or quarantine** — it re‑runs; a flaky surface is quarantined and raised as a limitation, never passed to the baseline as truth, because you cannot witness what you cannot reproduce. And because the run record is **persisted in‑repo and committed**, CI replays the identical scope and commands — local↔CI parity becomes a recorded fact, not a hope (closing the playbook's standing worry about "lost context between local and CI").

See the **execution‑runner** skill for the run‑record schema and the resolve/run/observe procedure; `run-validation` is the agent that drives it, and `characterize-baseline` calls it to pin and re‑observe behavior.

---

## Phase 3 — Test Reconciliation (witnesses, authored honestly)

The Validation Plan proposes **fates** (`add`/`change`/`keep`/`remove`) for each AC's witness, but provisionally. Test Reconciliation makes them real: it **materializes new tests and adjusts existing ones** so every active criterion has a witness that traces to it, then hands them to the Runner for evidence. It is the step that finally writes test code — and *who* writes it, and *from what*, is load‑bearing.

**An independent witness.** Tests are authored by a dedicated test‑implementer (`implement-tests`), **never by the producer of the change**. The invariant holds that implementation is the only freely‑mutable element and tests are witnesses to criteria; if the same will writes both, the witness collapses into the thing it judges and silently becomes a witness to the *implementation* instead of the *criterion*. So the test‑implementer's authority for *what to assert* is the **Criteria Ledger** — or, for a refactor's characterization tests, the **pinned baseline** — and **never the new implementation**. (It may read the impl for mechanical wiring — how to invoke a surface — but not to decide what is correct; that read is flagged.)

**The honesty lock.** A test `change`/`remove` is honest only when three independent keys agree: the **Ledger** carries a criteria delta (`moved`/`retired`) to justify it, the **Baseline** classifies the behavior delta as `justified` rather than `regression`, and the **Runner** *observes* the evidence rather than it being asserted. A test edited **because it went red, with no criteria delta**, is regression‑laundering and is forbidden — the same guard the plan reviewer applies to fates, now enforced on real test edits. If behavior changed without a criterion moving, that is a regression (loop input) or a criteria gap (decision) — never a quiet test edit.

**Done is evidence, not assertion.** An AC's witness is satisfied only on **green evidence from the Runner**. A red witness with no implementation yet is not a failure to hand off — it is **loop input** for the implementer, whose production change is the freely‑mutable element. A non‑automatable AC is **admitted** as a runtime witness, never faked green. This is why a witness cannot be declared done without running it, and why the test must be implemented to close the loop: the evidence *is* the test, run.

See the **test‑reconciliation** skill for the fate→action mapping and the honesty lock; `implement-tests` is the test‑implementer that performs it and calls `run-validation` for evidence.

---

## What's forthcoming (kept coherent, not yet built)

- **Auto‑fix loop (local/CI)** — the self‑closing loop *on top of* the Execution Runner: a clean fail is diagnosed, a fix proposed as a commit and re‑run — never a silent edit to a protected branch; CI runs the same plan with constrained autonomy.
- **Evidence Ledger** — the audit trail of justified test changes that makes human review fast.
