---
name: execution-runner
description: >-
  The run-record schema and tier-preflight/run/observe procedure for the Execution Runner — the substrate
  that drives the project's own test/build suite over a change's blast radius to produce structured
  behavior observations and a determinism verdict. Reuses the project's suite; runs, never edits.
  Used by run-validation; feeds the Behavior Baseline. Phase 3.
---

The Execution Runner is the toolkit's **execution substrate** — the first piece that *runs* anything (everything before it proposes, never runs or edits). It drives **the project's own suite** by invoking your **own test runner** (the standard command devs and CI already use — never a parallel harness the toolkit invents) over the change's blast‑radius slice, and returns **structured observations**: what ran, what each result witnesses about behavior, and whether it reproduces. It **runs but never edits** — observation only; deciding what to *do* with a result belongs to the Behavior Baseline (pin/reconcile) and the correction loop. Per‑change, **recorded in‑repo and committed** so CI replays the same scope and commands (local↔CI parity).

**The execution split that defines the layer.** Every non‑pass result is sorted into exactly one bin — this is the operational form of the toolkit's "no artificial handoffs" stance, applied to execution outcomes:

- **Clean fail** — the test ran and asserted false → **loop input**, a behavior signal. Never escalated to a human.
- **Can't‑run** — couldn't run in the available environment (runner won't invoke, infra absent) → **limitation**, a toolkit gap. Never normalized as human‑in‑the‑loop.

A red test and a broken harness look identical at a glance; conflating them is how a toolkit quietly accretes handoffs that hide its own gaps. The runner refuses to — it sorts every non‑pass into **signal or gap**.

## Schema (the run record)

```
run-ref:        <change-ref> @ <commit/state run from>
scope:          <blast-radius slice actually run — the minimal sufficient set, not the whole suite>
gate:           local | ci                 # local-gate run locally; ci-gate widens scope in CI
invocation:     [ <the project's own test command / runner used — its standard invocation, with a selector> ]
report:         <the test-report read — path (local) or CI artifact + format: junit-xml|tap|json|native>
environment:    local | ci

observations:                              # one per observed surface / witnessing test
  - surface-id:   <ties to a behavior-baseline surface, or a test ref>
    ran:          <command + selector that produced it>
    outcome:      pass | fail | error
    witnessed:    <the behavior the result evidences — assertion(s), output, status>
    determinism:  deterministic(<re-runs agreed>) | flaky(<how they disagreed>)

verdict:
  reproducible:   true | false
  loop-input:     [ <clean fails — behavior signals for the baseline / loop> ]
  limitations:    [ <can't-run | runner-unavailable | env-absent | flaky-quarantined> ]
```

## Procedure (tier & preflight → invoke → scope → run → check → record)

1. **Tier & preflight** — for each in‑scope category take its **tier** (from the Strategy). **① native** → runs here; **② your‑env** → run where the environment allows (else CI); **③ external** → **out of scope by default** (your pipeline owns it) — the runner does **not** run it and does not relay; *optionally* read its result for the audit trace, reading only. A category that *should* run here but whose environment isn't available → a **limitation up front** ("needs X, not available"), never effort spent on a test that can't execute.
2. **Invoke the project's runner** — run via the project's **own test command / runner** (the standard invocation devs and CI use), selecting the minimal slice where the runner supports selection. The toolkit **uses your runner; it never catalogs or guesses commands.** A runner it genuinely can't invoke is a **limitation**.
3. **Scope & order (fail‑fast)** to the blast radius and the relevant gate — run the **minimal sufficient** slice via selective invocation, **cheapest first**: the `local-gate` (fast low‑level tests) locally **before** the `ci-gate` (cross‑boundary / infra tests), so problems surface early and a local failure short‑circuits the CI‑level run. Never "run everything." A test that legitimately can't run locally (CI‑only by placement) is **deferred to its gate, not a limitation** — a limitation is only a test that can't run *where it's supposed to*.
4. **Run & observe** — execute, then read the **machine‑readable test report** (Source‑Map `test-report`), **not** the console; normalize it (JUnit XML / TAP / native JSON) into per‑surface `outcome` + the behavior it `witnessed`. Sort each non‑pass: **clean fail → loop input**, **can't‑run → limitation**. No machine‑readable report → a **limitation** (configure a reporter), never stdout scraping. The runner **does not edit** tests or implementation.
5. **Determinism check** — re‑run the slice; agreement → `deterministic`; disagreement → `flaky`: **quarantine** the surface and raise a **limitation**. Flaky behavior is never passed to the baseline as truth — you cannot witness what you cannot reproduce.
6. **Record** the run in‑repo (committed) so CI replays the identical scope and commands — local↔CI parity is a recorded fact, not a hope.

## CI — a participant, not a trigger

The toolkit **does not dispatch CI.** Local runs `gate=local` (the fast tests); the project's **normal pipeline trigger** (push / PR) invokes the runner at `gate=ci` for the wider scope — the cross‑boundary / infra / e2e tests that can only run there. **Your pipeline owns how CI builds and tests**; the toolkit reads `ci-config` only as a parity *reference*, not as the source of execution. The runner taps the **same `test-report`** in both places — a local file locally, the CI run's artifact in CI — so local and CI evidence are directly comparable. The correction loop spans both: a red `ci-gate` is simply `loop-input` on its next pass. (Optionally a team may have the runner dispatch a workflow and wait, but that couples it to a provider; the default is *CI runs the toolkit*.)

## Guards

- **Runs, never edits** — the substrate observes; it does not touch tests or implementation.
- **Reuse the project's runner** — the toolkit invokes *your own* test command/runner; never a parallel harness.
- **Report, not console** — observations come from a machine‑readable `test-report`; no report → a *limitation*.
- **CI is a participant** — `gate=ci` runs *within* the pipeline; the toolkit never dispatches CI by default.
- **Clean fail ≠ can't‑run** — a red test is **loop input**; a broken harness is a **limitation**.
- **No handoff for a fail** — a failing test is loop input, never escalated to a human.
- **Minimality** — run the blast‑radius slice, not the whole suite.
- **Fail‑fast ordering** — cheapest/lowest‑level local tests run first; CI‑only placement is expected.
- **Determinism or quarantine** — flaky → limitation; never baselined.
- **Local↔CI parity** — same commands and method, recorded; CI widens scope only.
- **Persisted** — the run record lives in‑repo so local and CI share execution context.
