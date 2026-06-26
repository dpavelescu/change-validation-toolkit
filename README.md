# Change‑Validation Toolkit

**Turn a change into the evidence needed to trust it.** The toolkit works out what would prove a change is correct, produces that evidence against your real code, runs your own test suite, and drives failures to green by handoff. It **writes no code** — it specifies what must be true, verifies it, runs, and records; authoring stays with you (or your coding agent).

New here? The **[Playbook](Change-Validation-Playbook.md)** is the human guide — what it's for, how it works end‑to‑end, and what's deliberately out of scope. This README gets you running.

---

## Quickstart

It ships as **two parallel builds — same model, each in its tool's idiom.** Pick yours:

| | **Claude Code** | **GitHub Copilot** |
|---|---|---|
| Copy into your project | `.claude/` | `.github/` |
| Contains | `commands/` · `agents/` · `skills/` | `agents/` · `skills/` |
| You invoke it as | slash commands — `/plan-validation` | the agent — select `plan-validation` in Copilot |

**1 · Install** — copy the build for your tool into your repo, plus the source‑map template at the root:
```bash
# Claude Code
cp -r .claude source-map.manifest.md /your/project/
# GitHub Copilot
cp -r .github source-map.manifest.md /your/project/
```

**2 · Fill the Source‑Map** (`source-map.manifest.md`) — where your real sources live and **which is authoritative for what** (architecture, API/event specs, tests, CI config). Declare external/cross‑repo sources; let search find the in‑repo rest.

**3 · Set up once — author your testing strategy**
- **Claude:** `/define-testing-strategy`  ·  **Copilot:** run the `define-testing-strategy` agent.
- It builds a human‑owned **Testing Strategy** from your architecture (asking one question at a time), then generates the machine‑facing **Validation Rules**. You approve it. Re‑run only when your architecture changes.

**4 · Per change — plan, then drive to green**
- **Plan** — `/plan-validation` *(Claude)* or the `plan-validation` agent *(Copilot)*, with the change + its story → a reviewed **Validation Plan** you approve.
- **Drive** — `/drive-correction` or the `drive-correction` agent → it pins current behavior, specifies the tests, runs your suite, and for each failure hands you a structured **fix‑request** and **pauses**. You (or your coding agent) apply it and re‑invoke. On green it writes the durable **Evidence Ledger**.

> **Where you're in the loop:** filling the source‑map, approving the Strategy and the Plan, answering **decisions**, and implementing code/tests (or delegating). **Where you're not:** deciding what to test, chasing which tests a change affects, or debugging a red test — that's the loop's job. By design it routes failures back through the loop and hands you decisions, not broken code. (Full walkthrough: **[Playbook → Running it end to end](Change-Validation-Playbook.md#running-it-end-to-end)**.)

---

## The basic flow

```
  SET UP ONCE                        PER CHANGE
  ───────────                        ──────────────────────────────────────────────
  define-testing-strategy   ──►      plan-validation   ──►   drive-correction   ──►  ✓ green
  → Testing Strategy + Rules         → Validation Plan       baseline · specify tests        │
    (you approve)                      (you approve)         · run your suite · correct      ▼
                                                             by handoff (writes no code)   Evidence Ledger
```

You run **three things**; everything else runs underneath as a subagent. See the Playbook for the scenarios it handles and the decision‑vs‑limitation model.

---

## What's in the box

Both builds carry the **same 9 agents + 12 skills**; the Claude build adds 3 slash `commands/` as the entry points.

```
Change-Validation-Playbook.md   ← the human guide — read this for the model
source-map.manifest.md          ← fillable: where your sources live + authority (copy to repo root)

agents/   (you run the first three; the rest are subagents)
  define-testing-strategy   author/update the Strategy + generate the Rules (human-owned)
  plan-validation           derive the Validation Plan: criteria ids, AC→test, dispositions, gates
  drive-correction          drive to green via test-/fix-request handoffs; re-assess + re-validate (writes no code)
  classify-change           classify a change → types + blast radius + needed sources
  review-plan               gate the plan: coverage + disposition justification
  capture-baseline          pin current behavior; sort post-change deltas → justified vs regression
  specify-tests             own what each test asserts (from criteria) + verify; authoring handed off
  run-validation            run the project's own suite over the blast radius → observations + flakiness
  record-evidence           on green, write the durable Evidence Ledger (criteria → test → evidence)

skills/   (auto-loaded reference each agent applies)
  testing-strategy · validation-rules · change-taxonomy · source-map · validation-plan ·
  behavior-baseline · execution-runner · test-reconciliation · correction-loop ·
  evidence-ledger · escalation · output-style
```

---

## The one principle

For any change: **criteria are authoritative**, the **implementation (code + tests) is the only freely‑mutable element**, **tests check the criteria — they never define them**, and **humans decide, they don't debug**. The toolkit writes no code; it specifies, verifies, runs, and records. The *why* — and the escalation model, the artifacts that count as audit evidence, and the foundation in depth — is in the **[Playbook](Change-Validation-Playbook.md)**.

**Status:** Foundation + Phase 2 + Phase 3, specified end‑to‑end, in both the `.github/` (Copilot) and `.claude/` (Claude Code) builds. Companion to the [work‑item‑preparation‑toolkit](../work-item-preparation-toolkit) — that one clarifies *what* to build; this one validates *that a change is correct*.
