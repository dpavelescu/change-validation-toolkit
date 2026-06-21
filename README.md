# Change‑Validation Toolkit

**Turn a change into the evidence needed to trust it — derived from a testing strategy, executed local‑first then in CI, and self‑correcting — so a failing test becomes loop input, not a human handoff.**

Companion to the [work‑item‑preparation‑toolkit](../work-item-preparation-toolkit): that one clarifies *what* to build; this one validates *that a change is correct*. The tool‑neutral **Playbook** is the human guide (purpose, capabilities, usage); the **skills** and **agents** are the self‑contained build that implements it.

> **Status — Foundation + Phase 2 + Phase 3 (most of it), Copilot build.** Built: the **Testing Strategy**, the **derived Validation Rules**, the **Source‑Map Manifest**, **Change Classification**, the tool‑managed **Criteria Identity**, the per‑change **Validation Plan**, the **Behavior Baseline** (the behavior snapshot that makes the auto‑fix honesty rule checkable), the **Execution Runner** (drives the project's *own* suite to observe behavior — the toolkit's first running piece), and **Test Reconciliation** (an *independent* test‑implementer that materializes witnesses from the criteria, never from the new impl). Still forthcoming: the auto‑fix loop and the Evidence Ledger. The `.claude/` build follows once this stabilizes.

---

## The idea in one principle

For any change: **criteria are authoritative**, the **implementation is the only freely‑mutable element**, **tests are witnesses to criteria** (never the source of truth), and **humans decide — they don't debug**. Testing is a control loop interleaved with implementation and CI, not a downstream phase. Everything in the toolkit is mechanism in service of that invariant.

---

## The derivation chain

```
architecture + technology  ──►  Testing Strategy  ──►  Validation Rules (derived, per change-type)
                                                              │
                              Source-Map Manifest ────────────┤  (resolves source kinds → locations)
                                                              ▼
                            Change Classification (types + blast radius)
                                                              │
                                                              ▼
            Criteria Identity → Validation Plan → Behavior Baseline → Test Reconciliation → [auto-fix loop → Evidence Ledger]
```

The **Execution Runner** is the shared substrate underneath: it runs the project's *own* suite to take the baseline's "before" snapshot and to gather evidence for test reconciliation. Bracketed stages are *forthcoming*.

Each arrow has a **minimum‑clarity gate**: if the source is too thin to derive honestly, the toolkit clarifies before proceeding rather than inventing.

---

## What's built (Foundation + Phase 2 + Phase 3)

```
README.md
Change-Validation-Playbook.md       ← the human guide (purpose, capabilities, usage)
source-map.manifest.md              ← fillable source-map instance (copy into your project)

.github/                            ← GitHub Copilot build
  agents/
    define-testing-strategy         author/update the Strategy + generate the Rules (human in loop)
    change-classifier               classify a change → types + blast radius + needed sources
    reconcile-criteria              give ACs stable per-change ids (local↔CI, not durable) (Phase 2)
    plan-validation                 derive the Validation Plan: AC→witness, fates, gates (Phase 2)
    validation-plan-reviewer        gate the plan: coverage + fate justification (Phase 2)
    capture-baseline           pin current behavior; sort post-change deltas → justified vs regression (Phase 3)
    run-validation                  drive the project's own suite over the blast radius → observations + determinism (Phase 3)
    implement-tests                 independent test-implementer: materialize witnesses from criteria, never the impl (Phase 3)
  skills/                           (derived, auto-loaded reference)
    testing-strategy                Strategy structure + coverage checklist
    validation-rules                Rule schema + derivation from the Strategy
    change-taxonomy                 the change-types + classification heuristics
    source-map                      manifest schema + deterministic discovery procedure
    criteria-identity                 per-change AC ids + new/unchanged/moved/retired reconciliation (Phase 2)
    validation-plan                 plan schema + derivation + AC→witness mapping (Phase 2)
    behavior-baseline       baseline schema + capture/reconcile + the honesty rule (Phase 3)
    execution-runner                run-record schema + resolve/run/observe + clean-fail vs can't-run (Phase 3)
    test-reconciliation             fate→action + criteria provenance + the honesty lock (Phase 3)
```

---

## The four foundation pieces

1. **Testing Strategy** — human‑owned, **architecture‑aware** source of truth for expected evidence, keyed by change‑type (REST API, SNS/SQS consumer, DB migration, React UI, cross‑service, refactor). Evidence, not tools.
2. **Validation Rules** — the thin, machine‑usable projection of the Strategy. **Generated from it, never hand‑edited beside it** (regenerate on change; version‑stamped).
3. **Source‑Map Manifest** — maps *source kinds → locations → what each is **authoritative for** → which change‑types need them*, so agents **discover sources deterministically** and **resolve a claim against its owner** instead of guessing. Records authority (`normative`/`descriptive`/`advisory`) so the API contract, event contract, boundaries, and expected evidence each have a declared source of truth — and the implementation is never it. Critical + unretrievable = blocking.
4. **Change Classification** — classify a change into types and scope its **blast radius**, selecting which Rules apply and which sources to retrieve. The entry of every per‑change activity.

---

## The autonomy boundary (why no artificial handoffs)

Every human touch is sorted into one of two bins:

- **Decision (legitimate)** — ambiguous/contradictory criteria, an ownership boundary, an unsafe design choice, a public‑contract change → emit a **structured question**. A human answers a question, never receives broken code.
- **Limitation (illegitimate)** — can't run/reproduce/build a fixture, lost context, missing access → a **toolkit gap to close**, logged as such, never normalized into "a human handles this."

A limitation must never masquerade as human‑in‑the‑loop. That's what keeps it autonomous *over time*.

---

## Onboarding (Foundation → Phase 3)

1. **Copy the build** (`.github/agents/` + `.github/skills/`) into your project, and **copy `source-map.manifest.md`** to your repo root.
2. **Fill the Source‑Map** with your real source locations (architecture, specs, schemas, tests, CI config).
3. **Run `define-testing-strategy`** — it retrieves your architecture and **authors the full human‑owned Strategy from scratch if you don't have one** (or updates an existing one), per change‑type, clarifying the genuinely‑open expectations **one question at a time** and surfacing any gap/inconsistency for your decision. Only then does it **generate the derived Validation Rules** (the thin AI‑facing layer). You own and approve the Strategy; the Rules are its projection.
4. **Run `change-classifier`** on a change — it returns the change‑types, blast radius, and the resolved sources the next phase will need.
5. **Run `plan-validation`** on the change + its story — it reconciles the acceptance criteria into stable AC IDs (**Criteria Identity**), then produces a reviewed **Validation Plan**: per‑AC required evidence, an AC→witness map, provisional test fates, and the local/CI gates. Review and approve.
6. Run `capture-baseline` on the change — it scopes the blast‑radius surfaces to pin and, via `run-validation` (which drives your *own* suite), captures current behavior, then sorts each post‑change delta into **justified** (a criterion moved) vs **regression** (none did), confirming the plan's provisional fates against fact. A clean fail is loop input; a can't‑run is a logged toolkit gap — never a handoff.
7. Run `implement-tests` — the **independent test‑implementer** materializes/adjusts each AC's witness from the **criteria** (or the baseline, for regression witnesses), never from the new impl, and runs it via `run-validation`. An AC is done only on green evidence; a red witness is loop input for the implementer; a `change`/`remove` not tracing to a criteria delta is blocked as regression‑laundering.
8. Hand the witnesses + evidence to the **auto‑fix loop** (diagnose red → propose a fix as a commit → re‑run) — *forthcoming*.

---

## Learn more

The [Change‑Validation Playbook](Change-Validation-Playbook.md) — the human guide to the toolkit: the operating principle, the full derivation chain, the artifacts, the escalation model, and the foundation in depth.
