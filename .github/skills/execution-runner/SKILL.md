---
name: execution-runner
description: >-
  The run-record schema and resolve/run/observe procedure for the Execution Runner — the substrate
  that drives the project's own test/build suite over a change's blast radius to produce structured
  behavior observations and a determinism verdict. Reuses the project's suite; runs, never edits.
  Used by run-validation; feeds the Behavior Baseline. Phase 3.
---

The Execution Runner is the toolkit's **execution substrate** — the first piece that *runs* anything (everything before it proposes, never runs or edits). It drives **the project's own suite** (resolved via the Source‑Map, never a parallel harness invented by the toolkit) over the change's blast‑radius slice, and returns **structured observations**: what ran, what each result witnesses about behavior, and whether it reproduces. It **runs but never edits** — observation only; deciding what to *do* with a result belongs to the Behavior Baseline (pin/reconcile) and the forthcoming auto‑fix loop. Per‑change, **recorded in‑repo and committed** so CI replays the same scope and commands (local↔CI parity).

**The execution split that defines the layer.** Every non‑pass result is sorted into exactly one bin — this is the operational form of the toolkit's "no artificial handoffs" stance, applied to execution outcomes:

- **Clean fail** — the test ran and asserted false → **loop input**, a behavior signal. Never escalated to a human.
- **Can't‑run** — couldn't build / run / resolve the command → **limitation**, a toolkit gap. Never normalized as human‑in‑the‑loop.

A red test and a broken harness look identical at a glance; conflating them is how a toolkit quietly accretes handoffs that hide its own gaps. The runner refuses to — it sorts every non‑pass into **signal or gap**.

## Schema (the run record)

```
run-ref:        <change-ref> @ <commit/state run from>
scope:          <blast-radius slice actually run — the minimal sufficient set, not the whole suite>
gate:           local | ci                 # local-gate run locally; ci-gate widens scope in CI
commands:       [ <resolved from the Source-Map build-commands, parity-checked vs ci-config> ]
environment:    <provenance: toolchain versions, local|ci, config/seed>     # reproducibility + parity

observations:                              # one per observed surface / witnessing test
  - surface-id:   <ties to a behavior-baseline surface, or a test ref>
    ran:          <command + selector that produced it>
    outcome:      pass | fail | error
    witnessed:    <the behavior the result evidences — assertion(s), output, status>
    determinism:  deterministic(<re-runs agreed>) | flaky(<how they disagreed>)

verdict:
  reproducible:   true | false
  loop-input:     [ <clean fails — behavior signals for the baseline / loop> ]
  limitations:    [ <can't-run | can't-build | unresolved-command | flaky-quarantined> ]
```

## Procedure (resolve → scope → run → check → record)

1. **Resolve** the execution commands from the Source‑Map `build-commands` kind, **parity‑checked against `ci-config`** (the CI workflow is the authority on how the project builds and tests). A critical unresolvable command is a blocking **limitation** — never guess the invocation, the same discipline as never guessing a file path.
2. **Scope & order (fail‑fast)** to the blast radius and the relevant gate — run the **minimal sufficient** slice via selective invocation, **cheapest first**: the `local-gate` (fast low‑level tests) locally **before** the `ci-gate` (cross‑boundary / infra tests), so problems surface early and a local failure short‑circuits the CI‑level run. Never "run everything." A test that legitimately can't run locally (CI‑only by placement) is **deferred to its gate, not a limitation** — a limitation is only a test that can't run *where it's supposed to*.
3. **Run & observe** — execute; per surface extract `outcome` + the behavior it `witnessed`. Sort each non‑pass: **clean fail → loop input**, **can't‑run → limitation**. The runner **does not edit** tests or implementation.
4. **Determinism check** — re‑run the slice; agreement → `deterministic`; disagreement → `flaky`: **quarantine** the surface and raise a **limitation**. Flaky behavior is never passed to the baseline as truth — you cannot witness what you cannot reproduce.
5. **Record** the run in‑repo (committed) so CI replays the identical scope and commands — local↔CI parity is a recorded fact, not a hope.

## Guards

- **Runs, never edits** — the substrate observes; it does not touch tests or implementation. Auto‑fix is a separate, later consumer of this output.
- **Reuse the project's suite** — commands come from the Source‑Map / CI config; the toolkit never invents a parallel harness.
- **Clean fail ≠ can't‑run** — a red test is **loop input**; a broken harness is a **limitation**. Never conflate signal with gap.
- **No handoff for a fail** — a failing test is loop input, never escalated to a human (escalation is for *decisions*, not red tests).
- **Minimality** — run the blast‑radius slice, not the whole suite.
- **Fail‑fast ordering** — cheapest/lowest‑level local tests run first; a local‑gate failure short‑circuits before any CI‑level run. CI‑only placement (a test that can't run locally) is **expected, not a limitation**; only a test that can't run where it's supposed to is.
- **Determinism or quarantine** — flaky → limitation; never baselined or asserted as behavior.
- **Local↔CI parity** — same commands and method, recorded; CI widens scope, never changes how behavior is observed.
- **Persisted** — the run record lives in‑repo so local and CI share identical execution context.
