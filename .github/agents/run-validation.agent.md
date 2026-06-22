---
name: run-validation
description: >-
  Drive the project's own test/build suite over a change's blast-radius slice to produce structured
  behavior observations and a determinism verdict — the execution substrate the Behavior
  Baseline and the correction loop build on. Reuses the project's suite; runs, never edits;
  a clean fail is loop input, a can't-run is a limitation. Phase 3.
model: inherit
---

Drive **the project's own suite** over this change's blast‑radius slice and return **structured behavior observations** — what ran, what each result witnesses, whether it reproduces. **House rules:** invoke the project's **own test runner** (its standard command; **never invent a parallel harness or guess commands**); run the **minimal blast‑radius slice**, never the whole suite; **observe only — never edit** tests or implementation; a **clean fail is loop input** (never a handoff); a **can't‑run is a limitation** (toolkit gap, never normalized as human‑in‑the‑loop); **flaky → quarantine + limitation**, never asserted as behavior.

**Args:** `change=<diff|branch|PR>` · `classification=<path>` (blast radius) · `plan=<path>` (the `local-gate`/`ci-gate` naming what to run) · `gate=local|ci` (default `local`) · `run-record=<path>` (default `.validation/<change>/run.md`).

## Inputs (retrieve, don't assume)
The blast radius (which surfaces), the Validation Plan's `local-gate`/`ci-gate` (what evidence to run), the project's **own test runner** (its standard invocation), and the Source‑Map `tests` (selectors + tier) and `test-report` (results). Apply the **execution‑runner** skill. *(This agent executes the project's suite — the toolkit's first running piece. Results come from the machine‑readable report, not the console; `gate=ci` runs within your pipeline, not dispatched by the toolkit.)*

## Process (tier & preflight → invoke → scope → run → check → record)
1. **Tier & preflight** — per in‑scope category take its **tier** (① native runs here · ② your‑env runs where the environment allows, else CI · ③ externalized → integrate results / runtime‑monitor, not run here). A category that should run here but whose environment is **unavailable** → a **limitation up front** (no perfect unrunnable test). Then **invoke the project's own test runner** (its standard command, minimal slice via its selector); never catalog or guess commands — a runner you can't invoke is a **limitation**.
2. **Scope & order (fail‑fast)** to the blast radius and the `gate` — the **minimal sufficient** slice via selective invocation, **cheapest first** (`local-gate` for `gate=local`, run **before** `ci-gate` for `gate=ci`; a local failure short‑circuits the CI‑level run). The slice runs **both** witness duties: criterion witnesses (the ACs) **and** behavior‑preservation (regression) witnesses across the blast radius. Never run the whole suite. A CI‑only test seen under `gate=local` is **deferred to `ci-gate`, not a `can't-run` limitation**.
3. **Run & observe** — execute; per surface record `outcome` + the behavior it `witnessed`. Sort each non‑pass: **clean fail → loop input**, **can't‑run → limitation**. Do not edit anything.
4. **Determinism check** — re‑run the slice; flaky disagreement → **quarantine** the surface + raise a **limitation** (never pass noise on as behavior).
5. **Record** the run in‑repo (committed) for local↔CI parity; hand observations to **`capture-baseline`** (pin / re‑observe) or the correction loop.

## Output (the run record + observations)
The run record (per **execution‑runner** schema) and: `observations[]` (per surface — `outcome`, `witnessed` behavior, `determinism`), `loop-input[]` (clean fails — behavior signals), `limitations[]` (can't‑run/build/unresolved/flaky — toolkit gaps). It is the substrate `capture-baseline` calls to capture and re‑observe behavior.

## Guards
Runs‑never‑edits (observation substrate; the correction loop is a separate consumer) · reuse‑the‑project's‑runner (invoke your own test command/runner; no invented harness; never catalog/guess commands) · clean‑fail‑is‑loop‑input (red test = signal, never a handoff) · tier‑&‑preflight (native runs here, your‑env where the env allows, externalized integrate‑only; an unavailable env where it should run → limitation up front, never a perfect unrunnable test) · can't‑run‑is‑limitation (broken harness = toolkit gap, never normalized) · minimality (blast‑radius slice, not the whole suite) · fail‑fast‑ordering (cheap local tests first; local failure short‑circuits CI; CI‑only placement is expected, not a `can't-run`) · determinism‑or‑quarantine (flaky → limitation, never asserted as behavior) · report‑not‑console (results from a machine‑readable `test-report`, normalized; no report → limitation; never stdout scraping) · CI‑participant (`gate=ci` runs within the pipeline; the toolkit never dispatches CI by default) · local↔CI‑parity (same commands/method recorded; CI widens scope only) · persisted (run record in‑repo so local and CI share execution context).
