---
name: correction-loop
description: >-
  The loop that drives a failing change to green — diagnose, emit a structured fix-request to whoever
  implements (agent-agnostic), re-assess impact, re-validate — WITHOUT the toolkit writing production
  code. The Behavior Baseline gates regression vs brittle. Used by drive-correction. Phase 3.
---

The correction loop closes the validation loop **without the toolkit touching production code**. When validation is red, it **diagnoses, emits a structured `fix-request`, and pauses** — whoever drives implementation (a human, or any implementation agent) applies the fix and **re-invokes** the loop. The toolkit owns tests, evidence, diagnosis, and the handoff; **implementation is out of scope**. It is per‑change, resumable, and runs your suite via the Execution Runner.

**One boundary, restated.** A red *criterion* test — a feature not built yet, or a regression — is **never fixed by editing the test**; it's a `fix-request` for the code. A red test whose *behavior is preserved* is a **brittle** test, repaired (decoupled) — but even that authoring is a handoff. **The toolkit writes no code at all**: it emits two kinds of handoff to the external implementer — a **`test-request`** (author/adjust a witness, from `specify-tests`) and a **`fix-request`** (author production code) — and *verifies* the result. It owns *what* must be true and *whether the result proves it*; the code (tests and production) is authored outside.

## The fix‑request (the handoff — agent‑agnostic)

```
fix-request:
  id:            FR-<n>
  trigger:       failing-criterion(AC-N) | regression(surface-id)
  expected:      <the criterion text, or the pinned baseline behavior>
  observed:      <what the Runner saw — assertion / output / status>
  witness:       <the test that defends it: file::case>   # DO NOT edit — it is the witness
  scope:         <where to fix — surface/code, from the blast radius>
  constraints:   [ honor <contract> (its authoritative source) · criteria are fixed · don't edit the witness ]
  status:        open        # resolved when re-validation passes
```

Bounded and coherent: *what's wrong · the witness · what you may not break · where to fix.* The loop writes it and stops; it does not call any particular implementer.

## Loop procedure (resumable — one pass per invocation)

1. **Validate** — `run-validation` over the affected slice (fail‑fast: local first). **Green & complete → done** (evidence → the Evidence Ledger).
2. **Re‑assess (on re‑invocation after a fix)** — the diff moved, so impact moved: incrementally update the blast radius / affected tests over the fix's delta (not a full re‑plan). A material scope change (new surfaces/criteria) → **return `re-plan`** (re‑run `plan-validation`, then resume); the loop never re‑plans itself.
3. **Sort each clean fail by the Behavior Baseline** — map the test to the surface its witness defends, then read the baseline (the honesty gate):
   - behavior **moved**, no criterion owns it → **regression**
   - behavior **moved**, a criterion owns it → **justified**
   - behavior **preserved**, test red → **brittle** (it asserted internal structure the fix changed)
4. **Route:**
   - **regression** / **failing‑criterion** → emit a **`fix-request`** (code fix, external).
   - **justified** → `change` the witness (a criteria delta justifies it); **brittle** → `repair` it (decouple, re‑align to assert the behavior, gated by the baseline's `preserved` verdict) — or, if it guards **nothing observable**, a **`remove` recommendation** (a human‑approved `decision`, never a silent delete). Both via `specify-tests`, then re‑validate — never to force green.
5. **Hand off & pause** — write the open `fix-request`s + the re‑assessment; **return control**. A human or any implementation agent applies them and re‑invokes.
6. **Terminate / escalate** — needs a criterion or contract to change → **decision** (structured question); **no progress** after `max-iterations` (a test stays red across re‑invocations, or fixes oscillate) → **escalate a diagnosis** (a limitation), never loop silently.

## Guards

- **Never writes production code** — the loop emits a `fix-request` and is re‑invoked after the external fix; implementation is out of scope.
- **Baseline is the gate** — `regression` vs `brittle` is decided by *behavior preservation*, never by the red result; a real regression can't be relabelled "brittle."
- **Witness never edited to pass** — test changes go through `specify-tests`, justified by a criteria delta (`justified`) or behavior preservation (`brittle`).
- **Re‑assess on every fix** — a fix ripples; impact is re‑evaluated incrementally (minimal), and a material scope change re‑plans.
- **No silent loop** — every pass ends in green, a handoff, a re‑plan, a decision, or an escalation; bounded by `max-iterations`.
- **Decision vs limitation** — criteria/contract change → decision; can't‑resolve / oscillation → escalate a diagnosis, never broken code to a human.
