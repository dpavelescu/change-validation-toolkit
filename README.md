# Change‑Validation Toolkit

**Turn a change into the evidence needed to trust it — derived from a testing strategy, executed local‑first then in CI, and self‑correcting — so a failing test becomes loop input, not a human handoff.**

Companion to the [work‑item‑preparation‑toolkit](../work-item-preparation-toolkit): that one clarifies *what* to build; this one validates *that a change is correct*. Same idiom — a tool‑neutral **Playbook** is canonical, **skills** are derived reference, **agents** orchestrate and apply lenses.

> **Status — Foundation + Phase 2 (Validation Plan), Copilot build.** Built: the **Testing Strategy**, the **derived Validation Rules**, the **Source‑Map Manifest**, **Change Classification**, the tool‑managed **Criteria Ledger**, and the per‑change **Validation Plan** (advisory — runs/edits nothing). The execution half (test reconciliation → local/CI auto‑fix → evidence) is specified in the Playbook and *forthcoming*. The `.claude/` build follows once this stabilizes.

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
                       Validation Plan → reconcile tests → local/CI auto-fix → evidence   (forthcoming)
```

Each arrow has a **minimum‑clarity gate**: if the source is too thin to derive honestly, the toolkit clarifies before proceeding rather than inventing.

---

## What's built (Foundation + Phase 2)

```
README.md
Change-Validation-Playbook.md       ← canonical, tool-neutral reference
source-map.manifest.md              ← fillable source-map instance (copy into your project)

.github/                            ← GitHub Copilot build
  agents/
    define-testing-strategy         author/update the Strategy + generate the Rules (human in loop)
    change-classifier               classify a change → types + blast radius + needed sources
    reconcile-criteria              maintain stable AC IDs in the Criteria Ledger (Phase 2)
    plan-validation                 derive the Validation Plan: AC→witness, fates, gates (Phase 2)
    validation-plan-reviewer        gate the plan: coverage + fate justification (Phase 2)
  skills/                           (derived, auto-loaded reference)
    testing-strategy                Strategy structure + coverage checklist
    validation-rules                Rule schema + derivation from the Strategy
    change-taxonomy                 the change-types + classification heuristics
    source-map                      manifest schema + deterministic discovery procedure
    criteria-ledger                 ledger schema + reconciliation/supersession (Phase 2)
    validation-plan                 plan schema + derivation + AC→witness mapping (Phase 2)
```

---

## The four foundation pieces

1. **Testing Strategy** — human‑owned, **architecture‑aware** source of truth for expected evidence, keyed by change‑type (REST API, SNS/SQS consumer, DB migration, React UI, cross‑service, refactor). Evidence, not tools.
2. **Validation Rules** — the thin, machine‑usable projection of the Strategy. **Generated from it, never hand‑edited beside it** (regenerate on change; version‑stamped).
3. **Source‑Map Manifest** — maps *source kinds → locations → which change‑types need them*, so agents **discover sources deterministically** instead of guessing. Critical + unretrievable = blocking.
4. **Change Classification** — classify a change into types and scope its **blast radius**, selecting which Rules apply and which sources to retrieve. The entry of every per‑change activity.

---

## The autonomy boundary (why no artificial handoffs)

Every human touch is sorted into one of two bins:

- **Decision (legitimate)** — ambiguous/contradictory criteria, an ownership boundary, an unsafe design choice, a public‑contract change → emit a **structured question**. A human answers a question, never receives broken code.
- **Limitation (illegitimate)** — can't run/reproduce/build a fixture, lost context, missing access → a **toolkit gap to close**, logged as such, never normalized into "a human handles this."

A limitation must never masquerade as human‑in‑the‑loop. That's what keeps it autonomous *over time*.

---

## Onboarding (Foundation)

1. **Copy the build** (`.github/agents/` + `.github/skills/`) into your project, and **copy `source-map.manifest.md`** to your repo root.
2. **Fill the Source‑Map** with your real source locations (architecture, specs, schemas, tests, CI config).
3. **Run `define-testing-strategy`** — it retrieves your architecture, drafts the Strategy per change‑type, clarifies the few genuinely‑open expectations, and **generates the Validation Rules**. Review and approve.
4. **Run `change-classifier`** on a change — it returns the change‑types, blast radius, and the resolved sources the next phase will need.
5. **Run `plan-validation`** on the change + its story — it reconciles the acceptance criteria into stable AC IDs (**Criteria Ledger**), then produces a reviewed **Validation Plan**: per‑AC required evidence, an AC→witness map, provisional test fates, and the local/CI gates. Review and approve.
6. Hand the plan to the **execution** phase (test reconciliation → local/CI auto‑fix) — *forthcoming*.

---

## Learn more

The [Change‑Validation Playbook](Change-Validation-Playbook.md) — the canonical reference: the operating principle, the full derivation chain, the artifacts, the escalation model, and the foundation in depth.
