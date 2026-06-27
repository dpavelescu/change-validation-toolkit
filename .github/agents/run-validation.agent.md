---
name: run-validation
description: >-
  Drive the project's own test/build suite over a change's blast-radius slice to produce structured
  behavior observations and a flakiness check — the execution substrate the Behavior
  Baseline and the correction loop build on. Reuses the project's suite; runs, never edits;
  a clean fail is fed back into the loop, a can't-run is a limitation. Phase 3.
model: inherit
---

Drive **the project's own suite** over this change's blast‑radius slice and return **structured behavior observations** — what ran, what each result evidences, whether it reproduces.

## Constraints
- **Invoke the project's own test runner** — its standard command; never invent a parallel harness or guess commands.
- **Run the blast‑radius slice** — minimal on trustworthy signal, widen when it's weak; never the whole suite.
- **Observe only — never edit** tests or implementation.
- **A clean fail is fed back into the loop**, never a handoff.
- **A can't‑run is a limitation** (toolkit gap), never normalized as human‑in‑the‑loop.
- **Flaky → quarantine and a limitation**, never asserted as behavior.
- **Run by tier** — native here · your‑env where it allows · external out of scope by default (your pipeline owns it); an unavailable env that should run here is a **limitation up front**.
- **Read results from the machine‑readable report, not the console** — no report is a limitation.
- **Under `gate=ci`, run within the pipeline, never dispatch CI.**
- **Record the run in‑repo** so local and CI share identical commands (CI widens scope only).

**Args:** `change=<diff|branch|PR>` · `classification=<path>` (blast radius) · `plan=<path>` (the `local-gate`/`ci-gate` naming what to run) · `gate=local|ci` (default `local`) · `run-record=<path>` (default `.validation/<change>/run.md`).

## Inputs
The blast radius (which surfaces), the Validation Plan's `local-gate`/`ci-gate` (what evidence to run), the project's **own test runner** (its standard invocation), and the Source‑Map `tests` (selectors + tier) and `test-report` (results).

## Process
1. **Tier & preflight** — per in‑scope category take its **tier** (**① native** runs here · **② your‑env** runs where the environment allows, else CI · **③ external** → **out of scope by default**, your pipeline owns it, not run here; optionally read its result for the trace). A category that should run here but whose environment is **unavailable** → a **limitation up front** (no perfect unrunnable test). Then **invoke the project's own test runner** (its standard command, minimal slice via its selector); never catalog or guess commands — a runner you can't invoke is a **limitation**. — *uses* **execution‑runner**.
2. **Scope & order (fail‑fast)** to the blast radius and the `gate` — the slice via selective invocation (minimal on trustworthy signal, widen when it's weak), **cheapest first** (`local-gate` for `gate=local`, run **before** `ci-gate` for `gate=ci`; a local failure short‑circuits the CI‑level run). The slice runs **both** test duties: acceptance tests (the ACs) **and** behavior‑preservation (regression) tests across the blast radius. Never run the whole suite. A CI‑only test seen under `gate=local` is **deferred to `ci-gate`, not a `can't-run` limitation**. — *uses* **execution‑runner**.
3. **Run & observe** — execute; per surface record `outcome` + the behavior it `evidenced`. Sort each non‑pass: **clean fail → fed back into the loop**, **can't‑run → limitation**. Do not edit anything. — *uses* **execution‑runner**.
4. **Flakiness check** — re‑run the slice; disagreement → **quarantine** the surface + raise a **limitation** (never pass noise on as behavior). Agreement is a smoke‑check, not proof of determinism. — *uses* **execution‑runner**.
5. **Record** the run in‑repo (committed) for local↔CI parity; hand observations to `capture-baseline` (pin / re‑observe) or the correction loop. — *uses* `capture-baseline`.

## Output
- **Run record** — per the **execution‑runner** schema: `observations[]` (per surface — `outcome`, `evidenced` behavior, `determinism` re‑run result) · `recheck[]` (clean fails — behavior signals) · a flakiness result. The substrate `capture-baseline` calls to capture and re‑observe behavior.
- **Limitations** *(only when raised)* — can't‑run/build/unresolved or flaky‑quarantined surfaces (toolkit gaps), in the **escalation** shape.
