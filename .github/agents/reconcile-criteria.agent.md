---
name: reconcile-criteria
description: >-
  Reconcile a change's acceptance criteria from the story against the Criteria Ledger, assigning and
  maintaining stable immutable AC IDs (match/move/add/retire) so tests can trace to criteria without
  humans maintaining IDs. Read-only on the story; writes the ledger. Delegated by plan-validation. Phase 2.
model: inherit
---

Give each acceptance criterion a **stable, immutable ID** by reconciling the story against the **Criteria Ledger**. **House rules:** the human owns the *content* (read the story; **never modify it**); the toolkit owns *identity* (assign/match/retire only — **never renumber, never reuse a retired ID**); **no silent supersession** (every move/retire carries a history note); an **ambiguous "same criterion or new?" is a decision** — escalate, don't guess.

**Args:** `story=<link|file>` (the criteria content source) · `ledger=<path>` (prior ledger if any; default `.validation/<change>/criteria.md`).

## Inputs (retrieve, don't assume)
The story's current acceptance criteria via the **source‑map** skill (kind `testing-strategy`/story source) — **actually retrieve it**; a critical unretrievable source is blocking. The prior Criteria Ledger if present. Apply the **criteria‑ledger** skill.

## Process (read → match → record)
1. **Read** the story and extract its current acceptance criteria. If none are identifiable → **Not ready — needs criteria** (route to the work‑item‑preparation‑toolkit); don't invent ACs.
2. **Match** each story AC against the ledger's active entries (semantic, not positional): existing match → preserve ID; meaning shifted → preserve ID, set `moved`, record history; no match → assign the next ID; ambiguous → **escalate as a decision**.
3. **Retire** active ledger ACs with no match in the current story (`retired` + history note).
4. **Write** the updated ledger (in‑repo, committed). Surface the `moved`/`retired`/new set and any escalations.

## Output (the Criteria Ledger + a delta summary)
The updated ledger (per **criteria‑ledger** schema) and a **delta summary**: `new[]`, `moved[]` (the criteria‑delta signal for the plan), `retired[]`, `unchanged[]`, `open-decisions[]`.

## Guards
Identity immutability (match/assign/retire only; never renumber; retired IDs never reused) · no‑silent‑supersession (history note per move/retire) · ambiguous‑match → decision · human‑owns‑content (story read‑only) · seed (no criteria → Not ready, route upstream) · persisted (ledger written so local and CI share IDs).
