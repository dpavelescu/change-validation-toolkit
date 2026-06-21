# STATUS — context handoff

*Living handoff so any session (or person) can resume with full context. Read this + the [Change‑Validation Playbook](Change-Validation-Playbook.md) and you're caught up. Update it when state changes.*

Last updated: 2026‑06‑21.

---

## What this repo is

An AI‑driven **change‑validation** toolkit: for any change, derive what evidence is needed to trust it, from a testing strategy grounded in the architecture, and (eventually) execute that validation local‑first then in CI with self‑correction — so a failing test is **loop input, not a human handoff**. Companion to the sibling `work-item-preparation-toolkit` (that clarifies *what* to build; this validates *that a change is correct*).

Built **Copilot‑first** (`.github/` only); the `.claude/` build follows once the design stabilizes.

## The operating principle (the invariant)

- **Criteria are authoritative** — what "correct" means for the change is the fixed point.
- **Implementation is the only freely‑mutable element**; tests are **witnesses** to criteria, never the source of truth.
- **Humans decide, they don't debug** — engaged only when the *criteria themselves* are unresolvable.

## The derivation chain

`architecture → testing strategy → validation rules → change classification → criteria ledger → validation plan → [tests → local/CI execution + auto‑fix → evidence]`
(bracketed = forthcoming). Each arrow has a **minimum‑clarity gate**: too thin to derive honestly → clarify, don't invent.

---

## Where we are

### ✅ Built — Foundation (milestone 1)
- **Testing Strategy** — human‑owned, architecture‑aware, evidence‑per‑change‑type. → `define-testing-strategy` agent + `testing-strategy` skill.
- **Validation Rules** — thin machine‑usable projection, **generated from the Strategy, never hand‑edited** (version‑stamped). → `validation-rules` skill.
- **Source‑Map Manifest** — source kinds → locations → which change‑types need them; deterministic discovery; critical+unretrievable = blocking. → `source-map.manifest.md` + `source-map` skill.
- **Change Classification** — change‑types + blast radius + resolved sources. → `change-classifier` agent + `change-taxonomy` skill.

### ✅ Built — Phase 2 (Validation Plan), advisory (runs/edits nothing)
- **Criteria Ledger** — tool‑managed stable AC IDs; human owns prose, tool owns identity; reconciliation = match/move/add/retire (supersession), ambiguous → decision. → `reconcile-criteria` agent + `criteria-ledger` skill.
- **Validation Plan** — per‑AC required evidence, AC→witness map, **provisional** test fates (justified by criteria deltas), local/CI gates. → `plan-validation` agent (orchestrator) + `validation-plan` skill.
- **Plan gate** — coverage + **criteria‑delta justification** of every change/remove fate (regression‑laundering guard). → `validation-plan-reviewer` agent.

### ⏳ Forthcoming — Phase 3 (Execution), NOT built
Test reconciliation against the real brownfield suite, **characterization baseline**, local/CI auto‑fix loop, Evidence Ledger.

---

## Decisions locked (with rationale)

1. **Copilot build first**, stabilize, then port to `.claude/`.
2. **First milestone = Foundation only**; Phase 2 added next (both advisory). Execution deferred.
3. **Criterion identity model** — human owns content (prose ACs in the story), toolkit owns identity (stable immutable `AC‑N` in the in‑repo **Criteria Ledger**). Humans never type IDs; the tool only matches/adds/retires, never renumbers. The ID lifecycle *is* the criteria‑delta detector. (Playbook → "Criteria identity — the ledger".)
4. **Story input is source‑agnostic** — a tracker link or in‑repo file, retrieved like `prepare-work-item` retrieves linked sources. Not tied to where stories live.
5. **Ledger + plan live in‑repo**, committed, travel with the change (so local and CI share identical IDs/context).
6. **Escalation = decision vs limitation.** A *decision* (ambiguous/contradictory criteria, ownership boundary, unsafe/public‑contract change) → structured question. A *limitation* (can't run/reproduce/build/lost‑context) → a toolkit gap to close, never normalized as human‑in‑the‑loop.
7. **No artificial handoffs** — a limitation must never masquerade as human‑in‑the‑loop.

---

## Open items / next decisions

- **Phase 3, first real gap:** the **characterization baseline** — how to pin current brownfield behavior so test changes sort into *intended* (criteria moved) vs *accidental* (regression). This is where autonomy and the **auto‑fix honesty rule** (a test changes only because a criterion moved, never because it went red) actually bite.
- **AC‑tag annotation convention** (native `@Tag("AC‑N")` / `[AC‑N]`, one `AC‑[0-9]+` regex) is specified to land **with execution**; humans stay out of that loop (tool stamps the tag from the ledger).
- **Copilot subagent tool** must be enabled for `plan-validation`'s delegation (`agents:` frontmatter) — confirm before leaning on it; add reviewer lenses similarly.
- **Push/auth:** commits land locally; pushing from the sandboxed shell fails (no credentials there). Push from your own terminal, or wire SSH/gh. Remote: `https://github.com/dpavelescu/change-validation-toolkit`.

---

## How to resume context in a fresh session

1. Read this file + the [Playbook](Change-Validation-Playbook.md) (canonical) + [README](README.md).
2. Skim `.github/agents/` and `.github/skills/` — they're terse and self‑describing.
3. Memory is path‑keyed and does **not** follow a folder move; this in‑repo file is the durable handoff. After settling the final folder location, seed project memory with the locked decisions if desired.
