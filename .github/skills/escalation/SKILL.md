---
name: escalation
description: >-
  The structured shape of the toolkit's two human-facing outputs — a decision (a legitimate judgment
  call, emitted as a question) and a limitation (a toolkit gap to close). Sorts every human touch into
  exactly one, so a limitation never masquerades as a decision. Applied wherever an agent escalates.
---

The toolkit engages a human in exactly two ways, and they must never be confused: a **decision** (a legitimate judgment call → a question the human answers) or a **limitation** (a gap in the *toolkit* → logged to close, never normalized as "a human handles this"). Both are **structured** so they are reviewable, reproducible, and recordable in the Evidence Ledger — not free prose.

## Decision (a legitimate human judgment)

```
decision:
  id:        D-<n>
  type:      ambiguous-criteria | contradictory-criteria | un-observable-criterion
             | ownership-boundary | public-contract-change | unsafe-design
  question:  <the single clear question the human answers>
  context:   <what forces the call — the conflicting criteria, the boundary crossed, …>
  options:   [ <candidate answers, if discrete> ]        # optional
  authority: <the source that owns the claim, if any (e.g. api-spec)>
  blocks:    <what stays Not-ready / un-validated until answered>
```

A decision is **a question, never broken code**. The human answers; they never debug.

## Limitation (a toolkit gap to close)

```
limitation:
  id:        L-<n>
  type:      cant-run | runner-unavailable | env-absent | cant-reproduce
             | flaky-quarantined | lost-context | missing-access | critical-source-unretrievable
  what:      <what the toolkit could not do>
  blocked:   <which evidence / step it blocked>
  needed:    <what would close the gap — the missing capability / access / fixture>
```

A limitation is **the toolkit's problem**, logged as such — **never** reframed as "a human handles this case." That reframing is the failure this structure exists to prevent.

## Guards

- **Exactly one bin** — every human touch is a `decision` *or* a `limitation`.
- **A limitation never masquerades as a decision** — "can't run / reproduce" is a gap to close, not a question.
- **A decision is a question** — the human answers; never handed a broken artifact.
- **Structured, not prose** — both carry their fields so review and audit are fast.
- **A failing test is neither** — a clean fail is fed back into the loop, not an escalation.
