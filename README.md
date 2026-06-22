# Change‑Validation Toolkit

**Turn a change into the evidence needed to trust it — derived from a testing strategy, executed local‑first then in CI, and self‑correcting — so a failing test is fed back into the loop, not a human handoff.**

Companion to the [work‑item‑preparation‑toolkit](../work-item-preparation-toolkit): that one clarifies *what* to build; this one validates *that a change is correct*. The tool‑neutral **[Playbook](Change-Validation-Playbook.md)** is the human guide (purpose, value, usage); the **skills** and **agents** under `.github/` are the self‑contained build that implements it.

> **Status — Foundation + Phase 2 + Phase 3, Copilot build.** The model is **specified** end‑to‑end (the parallel `.claude/` build follows). The toolkit writes no code: it specifies what must be true and verifies it; authoring is handed off.

**The one principle:** for any change, **criteria are authoritative**, the **implementation (code + tests) is the only freely‑mutable element**, **tests check the criteria; they are never the source of truth**, and **humans decide — they don't debug**. Everything here is mechanism in service of that invariant. The *why*, the escalation model (decision vs limitation), and the foundation pieces are detailed in the **[Playbook](Change-Validation-Playbook.md)**.

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
        Criteria IDs → Validation Plan → Behavior Baseline → Test Reconciliation → Correction Loop → Evidence Ledger
```

The **Execution Runner** is the shared substrate underneath: it runs the project's *own* suite to take the baseline's "before" snapshot and to gather evidence. Each arrow has a **minimum‑clarity gate** — if the source is too thin to derive honestly, the toolkit clarifies before proceeding rather than inventing.

---

## What's built

```
README.md
Change-Validation-Playbook.md       ← the human guide (purpose, value, usage)
source-map.manifest.md              ← fillable source-map instance (copy into your project)

.github/                            ← GitHub Copilot build
  agents/
    define-testing-strategy         author/update the Strategy + generate the Rules (human in loop)
    classify-change                 classify a change → types + blast radius + needed sources
    reconcile-criteria              give ACs stable per-change ids (local↔CI, not durable) (Phase 2)
    plan-validation                 derive the Validation Plan: AC→test, dispositions, gates (Phase 2)
    review-plan                     gate the plan: coverage + disposition justification (Phase 2)
    capture-baseline                pin current behavior; sort post-change deltas → justified vs regression (Phase 3)
    run-validation                  drive the project's own suite over the blast radius → observations + determinism (Phase 3)
    specify-tests                   own what each test asserts (from criteria) + verify; authoring handed off (Phase 3)
    drive-correction                drive to green via test-request + fix-request handoffs (writes no code); re-assess + re-validate (Phase 3)
    record-evidence                 on green, write the durable Evidence Ledger (criteria → test → evidence) (Phase 3)
  skills/                           (derived, auto-loaded reference)
    testing-strategy                Strategy structure + coverage checklist
    validation-rules                Rule schema + derivation from the Strategy
    change-taxonomy                 the change-types + classification heuristics
    source-map                      manifest schema + map-driven discovery (+ repo-search fallback)
    criteria-ids                    per-change AC ids + new/unchanged/moved/retired reconciliation (Phase 2)
    validation-plan                 plan schema + derivation + AC→test mapping (Phase 2)
    behavior-baseline               baseline schema + capture/reconcile + the no-edit-to-pass rule (Phase 3)
    execution-runner                run-record schema + tier-preflight/run/observe + clean-fail vs can't-run (Phase 3)
    test-reconciliation             disposition→action + criteria provenance + the no-edit-to-pass rule (Phase 3)
    correction-loop                 test-request + fix-request handoffs → re-assess → re-validate (Phase 3)
    evidence-ledger                 durable audit trail: criteria → test → evidence; output, never read back (Phase 3)
    escalation                      the structured shape of a decision (a question) vs a limitation (a gap) (cross-cutting)
    output-style                    handoff artifact vs human report + the no-fluff rules for human-facing output (cross-cutting)
```

---

## Get started

1. **Copy the build** (`.github/agents/` + `.github/skills/`) into your project, and **copy `source-map.manifest.md`** to your repo root.
2. **Fill the Source‑Map** with your real source locations (architecture, specs, schemas, tests, CI config).
3. **Run `define-testing-strategy`** once — it authors the human‑owned Strategy from your architecture (or updates an existing one), one question at a time, then generates the derived Validation Rules. You own and approve the Strategy.
4. Then, per change, run only the two entry points — each orchestrates the inner agents as subagents:
   - **`plan-validation`** (change + story) → a reviewed **Validation Plan** (runs `classify-change`, `reconcile-criteria`, `review-plan`).
   - **`drive-correction`** → executes and drives to green (runs `capture-baseline`, `specify-tests`, `run-validation`). It emits structured handoffs for whoever implements, **pauses**, and is re‑invoked after each fix — never writing code, never handing you broken code. On green it writes the **Evidence Ledger**.

See the **[Playbook](Change-Validation-Playbook.md)** for how it works end‑to‑end, the scenarios it supports, and what is deliberately out of scope.
