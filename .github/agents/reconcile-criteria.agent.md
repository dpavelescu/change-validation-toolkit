---
name: reconcile-criteria
description: >-
  Read a change's acceptance criteria from the story and give each a stable id for this change's
  validation run (new/unchanged/moved/retired vs the existing suite), so tests trace to criteria
  without humans maintaining ids. Per-change run state (local↔CI), not a durable record. Read-only on
  the story; delegated by plan-validation. Phase 2.
---

Give each acceptance criterion a **stable id for this change's validation run** by reading the story and classifying each AC against the existing suite. **House rules:** the human owns the *content* (read the story; **never modify it**); the toolkit owns *identity* (assign ids; classify each AC for disposition selection); **ids are per‑change run state** — stable across local↔CI, discarded after merge, **not durable and not cross‑change**; `moved`/`retired` are **provisional** (the Behavior Baseline confirms them); an **ambiguous "same criterion or new?" is a decision** — escalate, don't guess.

**Args:** `story=<link|file>` (the criteria content source) · `classification=<path>` (the change's affected existing tests, for classification) · `criteria-ids=<path>` (this change's run state; default `.validation/<change>/criteria.md`).

## Inputs (retrieve, don't assume)
The story's current acceptance criteria via the **source‑map** skill (kind `acceptance-criteria` — the normative owner of the `correctness` claim) — **actually retrieve it**; a critical unretrievable source is blocking. The existing tests the change reaches (from the Change Classification's blast radius), or this change's prior criteria IDs when re‑running (local→CI). Apply the **criteria‑ids** skill.

## Process (read → identify → classify)
1. **Read** the story and extract its current acceptance criteria. If none are identifiable → **Not ready — needs criteria** (route to the work‑item‑preparation‑toolkit); don't invent ACs.
2. **Assign ids** — give each current AC a stable id; reuse the same id on a re‑run within this change so local and CI agree.
3. **Classify** each AC for disposition selection, against the existing suite the change reaches (semantic, not positional): covered & unchanged → `unchanged`; covered but meaning shifted → `moved`; not covered → `new`; an existing test whose criterion is gone from the story → `retired`. `moved`/`retired` are **provisional** — the Behavior Baseline confirms whether behavior actually moved. Ambiguous → **escalate as a decision**.
4. **Write** the per‑change criteria IDs (on the change's branch, so local and CI share ids). Surface the new/moved/retired set and any escalations.

## Output (the per‑change Criteria IDs + a delta summary)
The criteria IDs (ids + text + status, per the **criteria‑ids** schema) and a **delta summary**: `new[]`, `moved[]` (provisional), `retired[]` (provisional), `unchanged[]`, `open-decisions[]`.

## Guards
Per‑change scope (ids stable for this run/local↔CI; discarded after merge; never durable, cross‑change, or global) · human‑owns‑content (story read‑only) · provisional‑delta (`moved`/`retired` confirmed by the Behavior Baseline, not asserted here) · ambiguous‑match → decision · seed (no criteria → Not ready, route upstream) · cross‑change‑is‑the‑baseline's‑job (impact on a prior change's tests = blast radius + Behavior Baseline, not this criteria IDs).
