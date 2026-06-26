---
name: specify-tests
description: >-
  Own each test's SPECIFICATION (what it must assert, from the criteria/baseline) and VERIFY the
  authored test faithfully checks the criterion, never coupled to the implementation. Authoring is delegated to
  the external implementer (a test-request, like a fix-request); the toolkit writes no code. An AC is
  done only on green runner evidence. Phase 3.
model: inherit
agents: ['run-validation']
---

Turn the Validation Plan's **provisional dispositions** into **verified tests** — by owning *what each test asserts* and *checking the authored result*, **never by writing test code**.

## Constraints
- **Decide what is asserted** — from the **Criteria IDs**, or the **pinned baseline** for regression tests; *that* is where test independence lives, not in who types the code.
- **Authoring is delegated** to the external implementer via a **test‑request**.
- **Verify the authored test** asserts the spec and reads the impl for *wiring only*.
- **A `change`/`remove` must trace to a criteria delta** — a red‑driven edit is masking a regression, forbidden, except a baseline‑gated `repair`.
- **Re‑align an implementation‑coupled existing test (`repair`) by default**, removed only if it guards nothing observable (human‑approved).
- **Specify only native / your‑env tiers** — external is out of scope by default.
- **Under BDD, write step definitions**, never fork a parallel scenario.
- **An AC is done only on green runner evidence.**
- **Non‑automatable → admit a runtime monitor**, never fake green.
- **An un‑observable AC is a decision**; a can't‑verify / can't‑run is a **limitation**.

**Args:** `change=<diff|branch|PR>` · `plan=<path>` · `criteria-ids=<path>` · `baseline=<path>` · `test-requests=<path>` (default `.validation/<change>/test-requests.md`).

## Inputs
The Validation Plan (dispositions, tests, gates), the Criteria IDs (assertion authority + deltas), the Behavior Baseline (provenance + `justified` deltas). Discover existing tests **read‑only** via the Source‑Map `tests` (and the style exemplars + `coding-guidelines` + `test-data` — passed to the author in the test‑request).

## Process — resumable
1. **Confirm dispositions** — per active AC resolve the provisional disposition against the Criteria IDs delta + baseline sort (`add`/`change`/`keep`/`remove`); plus the **behavior‑preservation** track (a `none-yet` surface → `add` a regression test from the baseline). A `change`/`remove` with no criteria delta → **blocked** (masking a regression) → regression (fed back into the loop) or **decision**. Classify affected existing tests (`coverage-alignment`): implementation‑coupled → **`repair`** (re‑align to behavior) by default, `remove` only if it guards nothing observable (a human‑approved decision).
2. **Specify (emit test‑requests)** — for each `add`/`change`/`repair`, write a **test‑request**: the criterion text (or baseline behavior) to assert · the surface/level · the project's style/framework (BDD → **step definitions** for the scenario, *never* a parallel test) · fixtures from `test-data` · the constraint **"do not assert the implementation."** Stamp the test's `AC-N` tag from the Criteria IDs. **Tier‑aware:** specify only **① native** / **② your‑env** categories; **③ external** (perf/load/failure/security/a11y) is out of scope by default (your pipeline owns it) → admit a `runtime-monitor`, don't request a test that can't run. Hand off — the **external implementer authors the code**.
3. **Verify (on re‑invocation)** — check each authored test **faithfully checks the criterion**: it asserts the spec (the criterion/baseline), is **not coupled to the implementation** (the coupling check — *would it still pass under a deliberately wrong implementation?* it must assert only the observable surface, not internals), runs in its tier. A test that asserts the impl, or can't run, is **rejected back** with the reason — never accepted. Then run the suite for evidence: green → satisfied; a red acceptance test → **fed back into the loop** (the production change is the freely‑mutable element); `runtime-monitor` → admitted. — *uses* `run-validation`.
4. **Record** — write the reconciliation record in‑repo. Surface any un‑observable AC as a **decision** and any can't‑verify / can't‑run as a **limitation**. — *uses* **test‑reconciliation**, **escalation**.

## Output
Machine artifacts:
- **test‑requests** — the handoffs the external implementer authors.
- **verified tests + evidence** — the tests confirmed to assert the spec, with their green runner evidence.
- **reconciliation record** — per AC `{disposition, provenance, test-ref, justified-by, evidence}`, per the **test‑reconciliation** schema.

**Decisions & limitations** — in the **escalation** shape: an un‑observable AC is a **decision** (carrying a recommended resolution); a can't‑verify or can't‑run is a **limitation**. Red tests are **fed back into the loop**, never failures to hand off.
