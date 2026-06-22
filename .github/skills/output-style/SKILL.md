---
name: output-style
description: >-
  How agents shape what they emit: machine artifacts are schema-based; the few human artifacts get an
  easy-to-read structure; when both audiences need the same thing, either derive a machine form from
  the human one (the Strategy→Rules pattern already in the toolkit) or give one artifact clearly-named
  sections both can read — never a duplicate. Plus the no-fluff rules for anything a human reads.
  Cross-cutting; cross-check outputs against this, don't restructure a working artifact.
---

Be precise about **who reads each output**. Most "poorly specified output" is a machine schema and a human summary blurred into one vague blob. Classify each output as one of three kinds and shape it to that — and **don't add a layer the current model doesn't already need**.

## Three kinds — classify each output

1. **Machine artifact** — consumed by a tool, a later agent, or replay. **Schema-based** (the owning skill's schema): stable fields, terse, complete, no prose. *Criteria IDs · Change Classification · Behavior Baseline · run record · reconciliation record · Validation Rules · `fix-request` / `test-request`.*
2. **Human artifact** — owned or acted on by a person. **Easy-to-read structure** (named sections, plain prose, the rules below). Few of these: *the Testing Strategy (human-owned) · each escalation **decision** / **limitation** a person must answer or close.*
3. **Both audiences** — don't fork into two documents that can drift. Two patterns; prefer the one already in the model:
   - **Derive a machine form from the human one.** The human artifact stays readable; a thin machine/AI projection is generated from it. *This is the **Strategy → Validation Rules** pattern already in the toolkit — reuse it, don't reinvent.*
   - **One artifact, sections for both.** Keep it schema/structured for the tool, with **clearly-named sections and a one-paragraph lead** a person (or an AI) reads directly — no parallel prose copy. *The **Validation Plan** (the executor replays the schema; the approver reads its tracks, led by a "what to approve" summary) · the **Evidence Ledger** (a structured record an auditor reads by section).*

**Cross-check, don't restructure.** If an artifact already serves both — a schema with readable sections, or a human doc with a derived projection — keep it; just verify it reads cleanly for each audience. Only add a human summary or a derived form where one is genuinely missing.

## Rules for anything a human reads (kind 2, and the human side of kind 3)

1. **Name the audience and their next action** — write only what serves it: a reviewer approving a plan needs coverage and open decisions; an implementer needs the failing expectation and where to fix.
2. **Lead with the conclusion** — the decision, verdict, or finding first; the support after. No preamble.
3. **Relevant prose only** — cut the obvious, the motivational, the hedging; if removing a sentence loses nothing, remove it.
4. **Structured to scan** — named sections or short lists, one idea each; length proportional to stakes.
5. **Every point carries its next step** — signal is never left without an action.
6. **Plain, established words** — no coined jargon where a standard term exists.

## Guards

- **Classify the output** — machine artifact (schema), human artifact (readable), or both.
- **Both ≠ duplicated** — derive a machine form from the human one, or give one artifact sections for both; never two copies to drift.
- **Reuse the model** — Strategy→Rules is the derivation template; cross-check existing artifacts, don't add layers the model already covers.
- **Schema for tools, readable for people** — never raw schema dumped at a human, never a decision buried in a field.
- **No fluff** — every sentence serves the named reader.
