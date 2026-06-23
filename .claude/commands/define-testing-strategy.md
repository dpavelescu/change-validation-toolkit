---
description: One-time setup — author the human-owned Testing Strategy from your architecture and generate the derived Validation Rules.
argument-hint: (optional) mode=author|update · scope=change-types|all
---

Run the **define-testing-strategy** agent (`.claude/agents/define-testing-strategy.md`) to set up validation for this project.

It retrieves your architecture sources (via the Source-Map), and — if you don't have a written testing strategy — **authors one with you**, one question at a time, then generates the machine-facing **Validation Rules**. You own and approve the Strategy.

Args (optional): $ARGUMENTS

Run once, and again when architecture or technology patterns change — not per story. Full model: `Change-Validation-Playbook.md`.
