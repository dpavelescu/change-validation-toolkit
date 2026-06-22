---
name: output-style
description: >-
  How agents shape what they emit — the split between a handoff artifact (a skill's schema, read by a
  tool or a later agent) and a human report (readable prose over it), and the best-practice rules for
  the human one: name the audience and their next action, lead with the conclusion, relevant prose
  only, structured to scan, every point actionable. Cross-cutting; applies to every agent's Output.
---

Every output an agent emits is one of two kinds. **Name which, and shape it to that** — most "poorly specified output" is the two kinds blurred together.

## The two kinds

- **Handoff artifact** — consumed by a tool, a later agent, or replay (the Criteria IDs, run record, Behavior Baseline, the Validation Plan schema, the reconciliation record, a `fix-request` / `test-request`). It follows its **skill's schema exactly**: stable field names, complete, terse, parseable. Not prose.
- **Human report** — read by a person who will **decide, approve, answer, audit, or implement**. Prose over the artifact, held to the rules below.

Some outputs are **both** — the Validation Plan is a schema a tool replays *and* a thing a human approves. Then emit both layers: the schema for the tool, a short readable summary for the person. **Never hand a person raw schema; never bury a decision in a YAML field.**

## Rules for a human report

1. **Name the audience and their next action.** State who reads it and what they do with it, and write only what serves that — a reviewer approving a plan needs coverage and open decisions; an implementer needs the failing expectation and where to fix. Different reader → different report.
2. **Lead with the conclusion.** The decision, verdict, or finding first; the support after. No restating the task, no preamble.
3. **Relevant prose only — no fluff.** Full sentences where the reasoning carries weight; cut the obvious, the motivational, the hedging. If removing a sentence loses nothing, remove it.
4. **Structured to scan.** Named sections or short lists, one idea each. Length is proportional to stakes — a routine pass is a line; a decision gets its full context.
5. **Every point carries its next step.** A finding states what it means *and* what to do — signal is never left without an action.
6. **Plain, established words.** The vocabulary a practitioner already knows; no coined jargon where a standard term exists.

## Guards

- **Name the kind** — each output is declared handoff-artifact, human-report, or both.
- **Schema for tools, prose for people** — never raw schema at a human; never a decision buried in a field.
- **Audience-first** — content is what the named reader needs to act, nothing more.
- **No fluff** — every sentence serves the intent.
- **Signal carries its action** — no finding without a next step.
