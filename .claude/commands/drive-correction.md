---
description: Per change — execute the plan and drive the change to green by handoff (writes no code).
argument-hint: <diff|branch|PR> + path to the approved Validation Plan
---

Follow the **drive-correction** agent's procedure (`.claude/agents/drive-correction.md`) to drive this change to green:

$ARGUMENTS

Run **from this (main) session** so you can call the inner agents as subagents:
- first run: delegate **`capture-baseline`** (pin current behavior) and **`specify-tests`** (own what each test asserts + emit `test-request`s), then **`run-validation`** (run your own suite);
- for each failure: sort regression vs. brittle (via the baseline), emit a structured **`fix-request`** for whoever implements, and **pause** — re-invoke after the fix;
- on green: delegate **`record-evidence`** to write the **Evidence Ledger**.

The toolkit **writes no code** — authoring (tests and production) is handed off; it specifies, verifies, runs, and records. It hands you **decisions**, never broken code. Full model: `Change-Validation-Playbook.md`.
