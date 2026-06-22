---
name: escalation
description: >-
  The structured shape of the toolkit's two human-facing outputs — a decision (a legitimate judgment
  call, emitted as a question) and a limitation (a toolkit gap to close). Sorts every human touch into
  exactly one, so a limitation never masquerades as a decision. Applied wherever an agent escalates.
---

The toolkit engages a human in exactly two ways, and they must never be confused: a **decision** (a legitimate judgment call → a question, with a recommended answer, that the human settles) or a **limitation** (a gap in the *toolkit* → logged to close, never normalized as "a human handles this"). Both are **structured** so they are reviewable, reproducible, and recordable in the Evidence Ledger — not free prose.

## Decision (a legitimate human judgment)

A decision is raised only for what the toolkit cannot legitimately resolve itself — it first resolves what it can (consult the authoritative source, apply a stated default). What remains is genuine human judgment, surfaced **with a recommended resolution** so the human confirms or overrides a proposal rather than starting from a blank question. **Deriving the recommendation:** a contract / ownership decision proposes the **authoritative source's** position; a criteria ambiguity proposes the reading **most consistent with the other criteria** (and the safer option), marked as a best guess; where the toolkit genuinely has no basis, it says so rather than invent one. Decisions are **settled, not accumulated**: a *blocking* one (the plan can't be derived around it) makes the work **Not ready** until answered; a *confirmatory* one rides with the artifact for sign-off.

```
decision:
  id:        D-<n>
  type:      <hint for the reader, e.g. ambiguous-criteria · contradictory-criteria · un-observable-criterion
             · ownership-boundary · public-contract-change · unsafe-design — nothing branches on it>
  question:  <the single clear question the human settles>
  context:   <what forces the call — the conflicting criteria, the boundary crossed, …>
  recommend: <the toolkit's proposed answer + a one-line rationale; the human confirms or overrides>
  options:   [ <other candidate answers, if discrete> ]        # optional
  authority: <the source that owns the claim, if any (e.g. api-spec)>
  blocks:    <what stays Not-ready / un-validated until settled>  ·  blocking | confirmatory
```

A decision is **a question with a recommendation, never broken code**. The human decides; they never debug.

## Limitation (a toolkit gap to close)

```
limitation:
  id:        L-<n>
  type:      <hint for the reader, e.g. cant-run · runner-unavailable · env-absent · cant-reproduce
             · flaky-quarantined · lost-context · missing-access · critical-source-unretrievable — nothing branches on it>
  what:      <what the toolkit could not do>
  blocked:   <which evidence / step it blocked>
  needed:    <what would close the gap — the missing capability / access / fixture>
```

A limitation is **the toolkit's problem**, logged as such — **never** reframed as "a human handles this case." That reframing is the failure this structure exists to prevent.

## Guards

- **Exactly one bin** — every human touch is a `decision` *or* a `limitation`.
- **A limitation never masquerades as a decision** — "can't run / reproduce" is a gap to close, not a question.
- **A decision is a question** — the human settles it; never handed a broken artifact.
- **Propose, don't accumulate** — the toolkit resolves what it can, then recommends a resolution for what's left; decisions are settled (blocking → Not ready; confirmatory → at sign-off), never a growing backlog.
- **Structured, not prose** — both carry their fields so review and audit are fast.
- **A failing test is neither** — a clean fail is fed back into the loop, not an escalation.
