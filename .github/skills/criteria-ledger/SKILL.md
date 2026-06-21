---
name: criteria-ledger
description: >-
  The schema and reconciliation (supersession) procedure for the Criteria Ledger — the tool-managed
  record that gives each acceptance criterion a stable, immutable ID. Humans own the prose; the
  toolkit owns identity. Used by reconcile-criteria and read by plan-validation. Phase 2.
---

The Criteria Ledger makes acceptance‑criterion identity **stable without human effort**. The human owns the *content* (prose ACs in the story); the toolkit owns *identity* (the `AC‑N` IDs). The story stays the source of truth for what the criteria say; the ledger is the source of truth for which criterion is which. It is per‑change, **persisted in‑repo, committed, and travels with the change**.

## Entry schema (one per acceptance criterion)

```
ac-id:       AC-<n>            # tool-assigned, IMMUTABLE; never renumbered, retired IDs never reused
text:        <current criterion, normalized from the story>
status:      active | moved | retired
source-ref:  <where in the story/tracker it came from>
witnesses:   [ <test-ref | "runtime-monitor" | "none-yet"> ]   # populated by the Validation Plan
history:     [ { event: assigned|moved|retired, note, prior-text? } ]
```

## Reconciliation procedure (story ACs → ledger)

Run on every change. **Read the story; never modify it.** For each AC currently in the story, match against the ledger's active entries:

| Situation | Action | ID outcome |
|---|---|---|
| matches an existing active AC (even reworded) | match | **same ID preserved** |
| matches but meaning shifted | match + record | same ID, `status: moved`, history note |
| no match — genuinely new | assign next ID | new `active` entry |
| an active ledger AC has no match in the story | retire | `status: retired`, history note |
| ambiguous — same criterion reworded, or a new one? | **escalate as a decision** | human answers; no silent guess |

The **`moved` flag is the criteria‑delta signal** the Validation Plan and (later) test reconciliation depend on — it marks "this criterion's witness may legitimately change."

## Guards

- **Identity immutability** — only match / assign / retire. Never renumber an AC; never reuse a retired ID.
- **No silent supersession** — every `moved` or `retired` carries a history note; nothing changes meaning invisibly.
- **Ambiguous match → decision** — a borderline "same or new?" is escalated as a structured question, never resolved by guess (per the escalation model).
- **Human owns content** — the story is read‑only here; the ledger never edits the criteria, only tracks their identity.
- **Persisted** — the ledger is written to the repo so local and CI see identical IDs.
