---
description: Per change — derive the reviewed Validation Plan (what evidence proves this change is correct).
argument-hint: <diff|branch|PR> + the story / acceptance criteria
---

Follow the **plan-validation** agent's procedure (`.claude/agents/plan-validation.md`) to derive a **Validation Plan** for:

$ARGUMENTS

Run **from this (main) session** so you can call the inner agents as subagents — Claude subagents don't nest:
- delegate **`classify-change`** (types + blast radius + sources), then
- assign the **Criteria IDs** and derive the plan yourself (apply the **validation-plan**, **change-taxonomy**, **source-map** skills), then
- delegate **`review-plan`** to gate it.

Produce a **reviewed Validation Plan** (a draft for you to approve) — or **Not ready** with a resumable agenda. Decisions/limitations follow the **escalation** skill. Full model: `Change-Validation-Playbook.md`.
