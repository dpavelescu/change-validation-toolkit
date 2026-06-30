---
name: shape-output
description: >-
  How agents shape what they emit: machine artifacts are schema-based; the few human artifacts get an
  easy-to-read structure; when both audiences need the same thing, either derive a machine form from
  the human one (the Strategy→Rules pattern already in the toolkit) or give one artifact clearly-named
  sections both can read — never a duplicate. Plus the no-fluff rules for anything a human reads.
  Cross-cutting; cross-check outputs against this, don't restructure a working artifact.
---

## Inputs

- **candidate** — the output(s) being shaped, each one a draft artifact the calling agent intends to emit.
- **audience** — for each candidate, who reads it: a tool/later agent/replay (machine), a person who owns or acts on it (human), or both. Use the caller's stated audience when given; otherwise step 1 assigns it.

## Procedure

1. **Classify each candidate by audience.** Use the caller's stated `audience` if provided; otherwise assign exactly one kind here:
   - **Machine artifact** — consumed by a tool, a later agent, or replay. Shape it schema-based on the owning skill's schema: stable fields, terse, complete, no prose. Examples: Criteria IDs, Change Classification, Behavior Baseline, run record, reconciliation record, Validation Rules, `fix-request` / `test-request`.
   - **Human artifact** — owned or acted on by a person. Shape it with an easy-to-read structure (step 3) and the quality rules (step 4). Examples: the Testing Strategy (human-owned); each escalation **decision** / **limitation** a person must answer or close.
   - **Both** — a tool and a person each consume the same content. Do not fork into two documents that can drift. Apply one of the two patterns in step 2.

2. **Shape a "both" candidate by one of two patterns; prefer the one already in the toolkit.**
   - **Derive a machine form from the human one.** Keep the human artifact readable; generate a thin machine/AI projection from it. This is the Strategy → **derive-validation-rules** pattern already in the toolkit — reuse it.
   - **One artifact, sections for both.** Keep it schema/structured for the tool, with clearly-named sections and a one-paragraph lead a person (or an AI) reads directly — no parallel prose copy. Examples: the Validation Plan (the executor replays the schema; the approver reads its tracks, led by a "what to approve" summary); the Evidence Ledger (a structured record an auditor reads by section).

3. **Give every human artifact (and the human side of a "both") this skeleton, conclusion-first, grouped by what the reader acts on — never a flat list.**
   1. **Verdict** — the conclusion and the reader's one next action, in a line.
   2. **Named sections** — the substance, grouped by what the reader does with it (coverage, changes, risk), not by how it was produced. One concern per section.
   3. **Decisions & limitations** — calls to settle (each a question with a recommended resolution) and toolkit gaps; settled or surfaced for sign-off, never a passive backlog.

   Each concrete output names its own sections under this skeleton:
   - Validation Plan (approver view): Verdict, Coverage, Test changes, Regression scope, Decisions to settle.
   - Evidence Ledger (audit view): Outcome, Evidence per criterion, Behavior preserved, Coverage & residual risk, Justified changes, Decisions & limitations.
   - escalation (decision / limitation): the **classify-escalation** skill's fields are this skeleton already.

4. **Apply the quality rules when filling the sections of anything a human reads.**
   1. **Name the audience and their next action**, then write only what serves it: a reviewer approving a plan needs coverage and the decisions to settle; an implementer needs the failing expectation and where to fix.
   2. **Lead with the conclusion** — verdict first, support after. No preamble.
   3. **Cut any sentence whose removal loses nothing** — the obvious, the motivational, the hedging.
   4. **One concern per section, scannable** — named sections, one idea each, length proportional to stakes.
   5. **Carry a next step on every point** — never leave signal without an action.
   6. **Use plain, established words** — no coined jargon where a standard term exists.

5. **Cite skills in bold and agents in backticks.** In an agent's prose, write **skills in bold** (`**capture-behavior-baseline**`) and `agents in backticks` (`` `run-validation` ``). Tie each to the step that applies it with one uniform trailing tag — `— *uses* **skill**, \`agent\`` — never ad-hoc parentheses.

6. **Cross-check, do not restructure.** If a candidate already serves its audience — a schema with readable sections, or a human doc with a derived projection — keep it and verify it reads cleanly for each audience. Add a human summary or a derived machine form only where one is missing; never raw schema dumped at a human, never a decision buried in a field.

## Output

Per candidate, a classification record plus the shaped artifact:

```
{
  kind:    machine | human | both,
  pattern: derive | sections | n-a,   // n-a unless kind = both
  action:  unchanged | summary-added | projection-added,
  artifact: <the shaped output>
}
```

- `kind` — the audience assigned in step 1.
- `pattern` — for `kind = both`, which step-2 pattern was applied (`derive` = machine form from human; `sections` = one artifact, sections for both); `n-a` otherwise.
- `action` — `unchanged` when the candidate already conformed (cross-check noted, nothing added); `summary-added` when a human summary was added; `projection-added` when a derived machine form was added.
- `artifact` — the conforming output, shaped to its kind and cross-checked against the rules above.
