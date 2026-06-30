---
name: classify-escalation
description: >-
  The structured shape of the toolkit's two human-facing outputs — a decision (a legitimate judgment
  call, emitted as a question) and a limitation (a toolkit gap to close). Sorts every human touch into
  exactly one, so a limitation never masquerades as a decision. Applied wherever an agent escalates.
---

## Inputs

- **human-touch** — the point where the toolkit must engage a human: a judgment it cannot legitimately resolve itself, or a step it could not complete.
- **source/context** — what forces the touch: the conflicting or un-observable criteria, the boundary crossed, the authoritative source involved, or the capability/access/fixture that was missing.

## Procedure

1. **Resolve what you can first.** Before raising anything, consult the authoritative source and apply any stated default. Raise a human touch only for what remains.
2. **Sort the touch into exactly one bin** — a `decision` or a `limitation`, never both, never neither:
   - Emit a `decision` only when genuine human judgment remains after step 1 (a judgment call the toolkit cannot legitimately settle itself).
   - Emit a `limitation` when the *toolkit* could not do something (e.g. "can't run", "can't reproduce", "env absent"). Log it as the toolkit's gap; never reframe a gap as "a human handles this case".
   - A failing test is neither: feed a clean fail back into the correction loop (see `run-correction-loop`), do not escalate it.
   - Use the `type` enum hints below to label the bin for the reader; nothing branches on `type`.
3. **For a `decision`, derive the recommended resolution** so the human confirms or overrides a proposal rather than answering a blank question:
   - For a contract / ownership decision, propose the authoritative source's position.
   - For a criteria ambiguity, propose the reading most consistent with the other criteria (and the safer option), marked as a best guess.
   - Where the toolkit genuinely has no basis, state that rather than invent a recommendation.
4. **Settle decisions, don't accumulate them.** Mark a *blocking* decision (the plan can't be derived around it) as making the work Not ready until answered; mark a *confirmatory* decision to ride with the artifact for sign-off. A decision is a question the human settles — never a broken artifact to debug.
5. **Emit the structured record** for the touch using the matching schema below; never free prose, so the touch is reviewable, reproducible, and recordable in the Evidence Ledger (see `record-evidence-ledger`).

## Output

Emit exactly one record per human touch — a `decision` or a `limitation`.

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

```
limitation:
  id:        L-<n>
  type:      <hint for the reader, e.g. cant-run · runner-unavailable · env-absent · cant-reproduce
             · flaky-quarantined · lost-context · missing-access · critical-source-unretrievable — nothing branches on it>
  what:      <what the toolkit could not do>
  blocked:   <which evidence / step it blocked>
  needed:    <what would close the gap — the missing capability / access / fixture>
```
