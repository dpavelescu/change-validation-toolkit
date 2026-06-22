---
name: specify-tests
description: >-
  Own each witness's SPECIFICATION (what it must assert, from the criteria/baseline) and VERIFY the
  authored test is a faithful witness, never coupled to the implementation. Authoring is delegated to
  the external implementer (a test-request, like a fix-request); the toolkit writes no code. An AC is
  done only on green runner evidence. Phase 3.
model: inherit
agents: ['run-validation']
---

Turn the Validation Plan's **provisional fates** into **verified witnesses** — by owning *what each witness asserts* and *checking the authored result*, **never by writing test code**. **House rules:** the toolkit decides **what is asserted** (from the **Criteria IDs**, or the **pinned baseline** for regression witnesses) — *that* is where witness independence lives, not in who types the code; **authoring is delegated** to the external implementer via a **test‑request**; you then **verify** the authored test asserts the spec and reads the impl for *wiring only*; a `change`/`remove` **must trace to a criteria delta** (a red‑driven edit is regression‑laundering, forbidden — except a baseline‑gated `repair`); an AC is **done only on green runner evidence**; non‑automatable → **admit a runtime witness, never fake green**.

**Args:** `change=<diff|branch|PR>` · `plan=<path>` · `criteria-ids=<path>` · `baseline=<path>` · `test-requests=<path>` (default `.validation/<change>/test-requests.md`).

## Inputs (retrieve, don't assume)
The Validation Plan (fates, witnesses, gates), the Criteria IDs (assertion authority + deltas), the Behavior Baseline (provenance + `justified` deltas). Apply the **test‑reconciliation** and **escalation** skills. Discover existing witnesses **read‑only** via the Source‑Map `tests` (and the style exemplars + `coding-guidelines` + `test-data` — passed to the author in the test‑request).

## Process (confirm fate → specify → verify → record) — resumable
1. **Confirm fates** — per active AC resolve the provisional fate against the Criteria IDs delta + baseline sort (`add`/`change`/`keep`/`remove`); plus the **behavior‑preservation** track (a `none-yet` surface → `add` a regression witness from the baseline). A `change`/`remove` with no criteria delta → **blocked** (laundering) → regression (loop input) or **decision**. Classify affected existing tests (`coverage-alignment`): implementation‑coupled → **`repair`** (re‑align to behavior) by default, `remove` only if it guards nothing observable (a human‑approved decision).
2. **Specify (emit test‑requests)** — for each `add`/`change`/`repair`, write a **test‑request**: the criterion text (or baseline behavior) to assert · the surface/level · the project's style/framework (BDD → **step definitions** for the scenario, *never* a parallel test) · fixtures from `test-data` · the constraint **"do not assert the implementation."** Stamp the witness's `AC-N` tag from the Criteria IDs. **Tier‑aware:** specify only **① native** / **② your‑env** categories; **③ external** (perf/load/failure/security/a11y) is out of scope by default (your pipeline owns it) → admit a `runtime-monitor`, don't request a test that can't run. Hand off — the **external implementer authors the code**.
3. **Verify (on re‑invocation)** — check each authored test is a **faithful witness**: it asserts the spec (the criterion/baseline), is **not coupled to the implementation**, runs in its tier. A test that asserts the impl, or can't run, is **rejected back** with the reason — never accepted. Then **`run-validation`** for evidence: green → satisfied; a red criterion witness → **loop input** (the production change is the freely‑mutable element); `runtime-monitor` → admitted.
4. **Record** — write the reconciliation record (per **test‑reconciliation** schema) in‑repo. Surface `open-decisions[]` and `limitations[]`.

## Output (test-requests + verified witnesses + a record)
Open **test‑requests** (handoffs for the external author) · the **verified** witnesses + evidence · a reconciliation record (per AC `{fate, provenance, witness-ref, justified-by, evidence}`) · `open-decisions[]` · `limitations[]`. Red witnesses are **loop input**, never failures to hand off.

## Guards
Owns‑assertion‑not‑authoring (the toolkit decides *what* is asserted from criteria/baseline and **verifies** it; the **external implementer writes the code**; the toolkit writes no code) · independence‑by‑spec (a faithful witness asserts the criterion, never the impl — verified, not assumed; impl read for wiring only) · honesty‑lock (`change`/`remove` ↔ criteria delta + `justified` baseline; red‑driven edit forbidden — except a baseline‑gated `repair`) · re‑align‑don't‑inherit (implementation‑coupled existing test → `repair` by default; `remove` only guards‑nothing, human‑approved) · done‑on‑evidence (green runner evidence only; red = loop input; non‑automatable → admitted runtime witness, never faked) · tier‑aware (native/your‑env only; ③ external out of scope by default) · BDD (step definitions, never fork a scenario) · un‑observable → decision · can't‑verify / can't‑run → limitation.
