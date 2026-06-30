---
name: run-execution
description: >-
  The run-record schema and tier-preflight/run/observe procedure for the Execution Runner — the substrate
  that drives the project's own test/build suite over a change's blast radius to produce structured
  behavior observations and a flakiness check. Reuses the project's suite; runs, never edits.
  Used by run-validation; feeds the Behavior Baseline. Phase 3.
---

## Inputs

- **change-ref** — the change and the commit/state to run from.
- **scope** — the blast-radius slice to run: the minimal sufficient set, not the whole suite.
- **gate** — `local | ci`. Local-gate runs the fast low-level tests locally; ci-gate widens scope to the cross-boundary/infra tests in CI.
- **tier per category** — the Strategy tier (① native | ② your-env | ③ external) for each in-scope change category.
- **test-report location** — the machine-readable report to read, resolved via the Source-Map `test-report` (local file path, or CI artifact + format: `junit-xml|tap|json|native`).

## Procedure

1. Preflight each in-scope category by its tier; produce one disposition per category:
   - **① native** — run here.
   - **② your-env** — run where the environment allows; otherwise run at `gate=ci`.
   - **③ external** — out of scope by default (your pipeline owns it). Do not run it and do not relay. Optionally read its result for the audit trace, read-only.
   - A category that should run here but whose environment is absent → emit a **limitation** up front ("needs X, not available"); spend no effort on a test that can't execute.
2. Invoke the project's own test command/runner — the standard invocation devs and CI use — selecting the minimal slice where the runner supports selection. Use the project's runner; never catalog or guess commands and never invent a parallel harness. A runner you genuinely can't invoke → a **limitation**.
3. Order the slice fail-fast, cheapest first: run the `local` gate (fast low-level tests) before the `ci` gate (cross-boundary/infra tests), so a local failure short-circuits the CI-level run. Start scope minimal; **widen the slice when a clean fail or a flaky result lands in it** (the signal touched neighboring surfaces). A test that legitimately can't run locally (CI-only by placement) is deferred to its gate, not a limitation — a limitation is only a test that can't run where it's supposed to.
4. Run, then read the machine-readable test report (Source-Map `test-report`), not the console. Normalize it (JUnit XML / TAP / native JSON) into per-case `outcome` + the behavior it `evidenced`. Bind each report case to a surface by its `AC-N` tag and its `test-ref` (recorded in the plan/baseline), not by re-parsing the test name. Do not edit tests or implementation. No machine-readable report → a **limitation** (configure a reporter); never scrape stdout.
5. Sort every non-pass into exactly one bin: a **clean fail** (ran, asserted false) is a behavior signal fed back into the loop, never escalated to a human; a **can't-run** (runner won't invoke, infra absent, broken harness) is a **limitation**, never normalized as human-in-the-loop. A red test and a broken harness look identical at a glance — do not conflate them.
6. Check flakiness: re-run the slice (once by default; `N` per config). If any run disagrees → mark the surface `flaky`, quarantine it, and raise a **limitation**. Agreement is a smoke-check, not proof of determinism. Never pass flaky behavior to the baseline as truth — you cannot evidence what you cannot reproduce.
7. Never dispatch CI: local runs `gate=local`; the project's normal pipeline trigger (push/PR) invokes the runner at `gate=ci`. Read `ci-config` only as a parity reference. Read the same `test-report` in both gates — a local file locally, the CI run's artifact in CI — so local and CI evidence are directly comparable.
8. Record the run in-repo and commit it so CI replays the same scope and commands. The environments still differ; the scope and commands don't.

## Output

The run record:

```
run-ref:        <change-ref> @ <commit/state run from>
scope:          <blast-radius slice actually run — the minimal sufficient set, not the whole suite>
gate:           local | ci                 # local-gate run locally; ci-gate widens scope in CI
invocation:     [ <the project's own test command / runner used — its standard invocation, with a selector> ]
report:         <the test-report read — path (local) or CI artifact + format: junit-xml|tap|json|native>
environment:    local | ci

observations:                              # one per observed surface / test
  - surface-id:   <ties to a behavior-baseline surface, or a test ref>
    ran:          <command + selector that produced it>
    outcome:      pass | fail | error
    evidenced:    <the behavior the result evidences — assertion(s), output, status>
    determinism:  deterministic(<re-runs agreed>) | flaky(<how they disagreed>)

verdict:
  reproducible:   true | false
  recheck:        [ <clean fails — behavior signals for the baseline / loop> ]
  limitations:    [ <can't-run | runner-unavailable | env-absent | flaky-quarantined> ]
```
