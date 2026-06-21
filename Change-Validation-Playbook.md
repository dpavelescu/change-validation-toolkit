# Change‑Validation Toolkit — Playbook

> **Turn a change into the evidence needed to trust it** — derived from how your system is built, produced and checked against your real code, and self‑correcting, so a failing test becomes the system's next task, not yours.

This is the guide to **what the toolkit is for, what it does, and how to use it** — written for the people who adopt and run it. It is tool‑neutral; the agents and skills under `.github/` are the build that implements it (see [Under the hood](#under-the-hood) at the end).

---

## Why it exists

Every change has to be **trusted** before it ships, and today that trust is expensive and manual. Someone decides what to test, writes the tests, runs them, and — when something breaks — stops to debug. Two deeper problems hide behind that:

- **Tests drift from intent.** Over time a test ends up asserting *whatever the code does*, not *what the change was supposed to do*. When it goes red, it's edited until it's green again — quietly laundering a regression into "expected."
- **Brownfield changes break things silently.** A small change can alter behavior nobody thought to re‑check, in code that some *other*, unrelated story originally built.

The toolkit's bet is simple: **a change should carry its own evidence.** Starting from what the change is supposed to do, it works out what would prove that, produces the proof against your real system, and keeps the loop closed itself — so a red test is the next thing *it* works on, not a ticket back to you.

### Why you can trust the output

Four rules make the result trustworthy rather than merely automated. They're worth knowing, because they're *why* you can rely on what it produces:

- **Your acceptance criteria are the fixed point.** "Correct" is defined by what the change is meant to do — never by what the code happens to do.
- **The implementation is the only thing it rewrites.** Tests and fixtures bend around the criteria; the criteria never bend around the code.
- **Every test traces to a criterion.** A test exists to defend one specific expectation. It's a witness, never the source of truth.
- **You make decisions; you never debug.** It asks you a question only when the *criteria themselves* are genuinely unresolvable — it never hands you broken code to fix.

---

## What it does for you

Think of it as a set of capabilities, not a pile of files. Some you set up once; the rest run on every change.

### Set up once — about your system, not any one change

- **Captures your testing strategy.** What kinds of evidence make a change trustworthy *in your system* — and **if you don't have a written strategy, it authors one with you** from your architecture, asking one question at a time only where the answer genuinely isn't decided. It favours the **lowest test level that gives solid confidence** and pushes back on slow, brittle end‑to‑end tests where a focused unit or component test would do. The strategy stays human‑owned; you approve it.
- **Learns where truth lives.** A map of your sources — architecture, API specs, event schemas, tests, CI — and, crucially, **which source is authoritative for what** (the API spec defines the contract; the code doesn't). So when something has to be checked, it checks against the owner, not a guess.

### Every change — automatically

- **Finds the real blast radius.** It works out everything the change touches — *including which existing tests it affects, even tests written for old, unrelated work* — by analysing the code, with **no links for you to maintain** (more on this below).
- **Works out the evidence the change needs.** Two things, separately: that the **new behavior works** (your acceptance criteria) and that **nothing around it broke** (a regression guard over everything the change reaches).
- **Tells intended change from accidental breakage.** Before touching anything, it photographs current behavior. Afterward, a behavior that moved is either *justified* (a criterion moved with it) or a *regression* (nothing asked for it) — and it can tell which.
- **Writes and adjusts tests honestly.** Tests are authored from the criteria, never from the new code, and a test changes **only because the requirement behind it moved** — never to make a red result turn green.
- **Runs your own test suite — fast feedback first.** Not an invented harness — your suite — running the **cheap, local tests first** (unit, component) so problems surface early, before the slower CI‑level tests (integration, end‑to‑end), some of which can only run in CI anyway. It distinguishes a **real failure** (work to do) from a **broken harness** (its own gap to fix).
- **Drives correction to green — by handoff.** When a test fails it **diagnoses** and emits a structured **fix‑request** for whoever implements the code (a human or your impl agent), then re‑validates and iterates. It **never writes production code itself** and never hands a person broken code — only the tests, the evidence, and an honest diagnosis.
- **Keeps a durable audit trail.** When a change reaches green it records *what was validated, by what, and why* — criteria → witness → evidence, what behavior was preserved, the justified test changes, and any decisions or limitations — so a human review (or a compliance check) is about **behavior and decisions, not internals**. Evidence only; nothing is "done" without it.

> **The capability you should understand: finding affected tests without links.**
> When a change touches code that an old story's tests cover, those tests are found **because the change reaches that code** — discovered fresh every time from the code itself (call graph and/or coverage), never from a stored "this change relates to story X" reference. References rot; this can't, because there's nothing to keep. System‑level tests (end‑to‑end, integration) are caught too, by the *flows* they exercise — which is why the toolkit is told where each kind of test lives.

---

## How it works

**Three things you ever run** — everything else runs underneath them as a subagent:

| You run | When | It orchestrates underneath (subagents) |
|---|---|---|
| **`define-testing-strategy`** | once (and when architecture changes) | — *(authors the Strategy, generates the Rules)* |
| **`plan-validation`** | per change — to plan | `change-classifier` · `reconcile-criteria` · `validation-plan-reviewer` |
| **`drive-correction`** | per change — to execute & correct | `capture-baseline` · `implement-tests` · `run-validation` |

End‑to‑end: **set up once → plan a change → drive it to green.** You never invoke the inner agents directly — the two per‑change entry points orchestrate them. Underneath, the pipeline is *classify → plan → photograph behavior → write tests → run → correct*, with the **Execution Runner** as the shared engine that actually runs your suite.

### Scenarios it supports

These are the classes of change the toolkit handles end‑to‑end — each stated as a capability, then shown with a concrete example.

**① A new feature (new acceptance criteria).** It plans the required evidence, writes a witness for each criterion, and drives the unmet ones to green via fix‑requests — never editing a test to pass.
> *Example — "add an email field to signup":* classified as a REST change; the blast radius pulls in the endpoint, the shared `UserService`, and an old end‑to‑end test found *from the code*. The new criterion tests are red (feature not built) → fix‑request → you (or your impl agent) implement and re‑invoke → green.

**② A change that risks untouched‑but‑reachable code.** Its blast radius and behavior baseline catch a regression even in code some *other*, unrelated story built — with no links for you to maintain.
> *Example — a shared helper is tweaked:* nothing in the story asked for it, but an old order‑history endpoint now returns a different total. The delta maps to **no criterion → regression**, caught before anyone ships.

**③ A pure refactor (no behavior change intended).** Behavior‑preservation is the entire evidence: every touched behavior must match the pre‑change photograph; any difference is a regression.
> *Example:* an `extract method` cleanup — the baseline pins every reachable surface beforehand and confirms each is identical after.

**④ A fix that breaks a brittle test.** The loop tells a real regression from a test merely coupled to internal structure: if the baseline says behavior is **preserved**, the test is *brittle* and gets repaired (decoupled); if behavior **moved** with no criterion, it's a regression to fix. The baseline is the gate — a regression can never be relabelled "brittle."
> *Example:* a fix extracts a method; a unit test bound to the old internals goes red while the endpoint's behavior is unchanged → repaired, not laundered.

### When it comes to you — and when it doesn't

Every human touch is exactly one of two things — this is what keeps it autonomous without dumping work back on you:

- **A decision** — criteria ambiguous or contradictory, a public contract must change, something unsafe → a **clear question**. You answer a question; you never receive broken code.
- **A limitation** — it can't run a test, reproduce a failure, or build a fixture → the **toolkit's** gap to close, logged as such, never reframed as "a human handles this."

A failing test is **neither** — it's just the loop's next input. You hear about it only if it becomes a decision.

Both arrive in a **structured** shape (the **escalation** skill), not loose prose: a *decision* carries its question, the context that forces it, the owning authority, and what it blocks; a *limitation* carries the gap, what it blocked, and what would close it. So they're fast to review and they land in the Evidence Ledger.

---

## What it needs from you

- **Real acceptance criteria.** No criteria → you're still clarifying *what* to build (the companion work‑item toolkit's job); it says so rather than invent correctness.
- **A runnable suite and a seeded source map.** It runs *your* tests and resolves *your* sources from locations you provide — it never guesses or fakes, and reports a gap if it can't.

**Strongest fit:** brownfield systems where regressions hide and the same code is touched by many unrelated stories over time.

**What you get back, per change:** the evidence each requirement has (or a flagged gap), what ran and passed, what it **couldn't** do and why (its own gaps), and any **decisions** it needs. Nothing is "done" without evidence.

---

## Where it is today

**Specified end‑to‑end:** capturing the strategy, mapping sources and authority, classifying a change and its blast radius, planning the evidence, photographing behavior, running your suite, writing/adjusting tests honestly, **driving correction to green** via structured fix‑requests (implementation handed off, never written by the toolkit), and recording the durable **Evidence Ledger**.

**Next:** the parallel **`.claude/` build** — this is the Copilot‑first build; the model itself is now complete end‑to‑end.

---

## The toolkit — agents & skills (catalog)

This Playbook is the concept; the running build lives under `.github/`. Below is every **agent** — its purpose, arguments, the **skills** it uses, what it depends on, and what it produces — followed by a one‑line index of the skills.

**You run only three** (the entry points): `define-testing-strategy`, `plan-validation`, `drive-correction`. Everything else is a **subagent** one of those calls (shown as *Delegated by / Used by* below) — you don't invoke them directly in the normal flow, though each can be run standalone if you only want its output.

### Set up once

**`define-testing-strategy`** — Author or update the human‑owned **Testing Strategy** (authoring one from your architecture **if you don't have it**) and generate the derived **Validation Rules**. Run when architecture or technology patterns change, not per story.
- **Args:** `mode=author|update` *(inferred if omitted)* · `scope=change-types|all`
- **Uses skills:** `testing-strategy`, `validation-rules`, `change-taxonomy`, `source-map`
- **Needs:** your architecture sources (`architecture`, `api-spec`, `event-schema`, `data-model`, `coding-guidelines`, `ci-config`) via the Source‑Map
- **Produces:** the Testing Strategy + Validation Rules (a draft you approve)

### Plan a change — *advisory; nothing runs or is edited*

**`change-classifier`** — Classify a change into types, compute its **blast radius** (test‑impact analysis — including which existing tests it reaches), resolve the sources it needs, and record which source is **authoritative** for each crossed claim.
- **Args:** `change=<diff|branch|PR|description>` · `criteria=<acceptance criteria|link>` *(optional)*
- **Uses skills:** `change-taxonomy`, `validation-rules`, `source-map`
- **Needs:** the change · the Validation Rules · the Source‑Map
- **Produces:** change‑types · blast radius · affected tests (by type) · needed sources · claim authorities

**`reconcile-criteria`** — Give each acceptance criterion a **stable id for this change's run** (`new`/`unchanged`/`moved`/`retired` vs the existing suite). Read‑only on the story; ids are **per‑change** (stable across local↔CI, discarded after merge) — *not* a durable or cross‑change record.
- **Args:** `story=<link|file>` · `classification=<path>` *(affected tests)* · `criteria-ids=<path>` *(this change's run state; default `.validation/<change>/criteria.md`)*
- **Uses skills:** `criteria-identity`, `source-map`
- **Needs:** the story's ACs (Source‑Map kind `acceptance-criteria`) · the existing tests the change reaches (from the blast radius)
- **Produces:** the per‑change Criteria Identity + a **provisional** delta summary (new / moved / retired — the Baseline confirms moved/retired)
- **Delegated by:** `plan-validation`

**`plan-validation`** — *orchestrator* — Derive the **Validation Plan**: per‑AC required evidence and witness map, provisional test fates, the regression (behavior‑preservation) track, and the local/CI gates.
- **Args:** `change=<diff|branch|PR>` · `story=<link|file>` · `classification=<path>` *(reuse)*
- **Uses skills:** `validation-plan`, `change-taxonomy`
- **Delegates to:** `change-classifier`, `reconcile-criteria`, `validation-plan-reviewer`
- **Needs:** the change · the story · the Validation Rules · the Source‑Map
- **Produces:** the Validation Plan (a draft you approve) — or *Not ready* with a resumable agenda

**`validation-plan-reviewer`** — *gate lens* — Review the assembled plan before capture: coverage, fate justification, testability, blast‑radius regression, **test‑level discipline**, honesty, decisions. Read‑only.
- **Args:** none of its own (invoked by `plan-validation`)
- **Uses skills:** `validation-plan`, `criteria-identity`
- **Needs:** the assembled plan · the Criteria Identity
- **Produces:** findings + a recommendation (ready to capture / Not ready)
- **Delegated by:** `plan-validation`

### Execute a change — *Phase 3; runs your suite, drives correction*

**`capture-baseline`** — **Photograph current behavior** across the plan's surfaces before the change, then reconcile post‑change deltas into **justified** (a criterion moved) vs **regression** (none did).
- **Args:** `change` · `classification=<path>` · `plan=<path>` · `criteria-ids=<path>` · `baseline=<path>`
- **Uses skills:** `behavior-baseline`, `change-taxonomy`
- **Delegates to:** `run-validation`
- **Needs:** the plan's behavior‑preservation track · the classification · the criteria identity deltas · the pre‑change state
- **Produces:** the Behavior Baseline + a delta reconciliation

**`run-validation`** — Drive **your own test suite** over the blast‑radius slice (**cheapest/local first**), returning structured observations and a determinism verdict. The execution substrate; runs, never edits.
- **Args:** `change` · `classification=<path>` · `plan=<path>` · `gate=local|ci` · `run-record=<path>`
- **Uses skills:** `execution-runner`
- **Needs:** the blast radius · the plan's local/CI gates · the Source‑Map `build-commands`/`ci-config`/`tests`
- **Produces:** a run record + observations · loop‑input (clean fails) · limitations
- **Used by:** `capture-baseline`, `implement-tests`

**`implement-tests`** — Materialize and adjust the **witnessing tests** per the plan's fates — criterion witnesses from the criteria, regression witnesses from the baseline. The **independent test‑implementer**; never edits production code.
- **Args:** `change` · `plan=<path>` · `criteria-ids=<path>` · `baseline=<path>`
- **Uses skills:** `test-reconciliation`
- **Delegates to:** `run-validation`
- **Needs:** the Validation Plan · the Criteria Identity · the Behavior Baseline · the Source‑Map `tests`
- **Produces:** the materialized/adjusted tests + a reconciliation record

**`drive-correction`** — Drive a failing change **to green by handoff**: diagnose each failure into a structured **fix‑request** for whoever implements (a human or any implementation agent), re‑assess impact, re‑validate, and iterate. **Never writes production code.** Resumable — emits handoffs and pauses; re‑invoked after each external fix.
- **Args:** `change` · `plan=<path>` · `baseline=<path>` · `run-record=<path>` · `fix-requests=<path>` · `max-iterations=<n>`
- **Uses skills:** `correction-loop`
- **Delegates to:** `run-validation`, `change-classifier` *(re‑assess)*, `capture-baseline` *(sort)*, `implement-tests` *(test fixes)*
- **Needs:** the Validation Plan · the Behavior Baseline · the latest run record (loop‑input)
- **Produces:** one of — **green** (evidence) · **fix‑requests** (handoffs) · **re‑plan** (scope grew) · a **decision** · an **escalation** (no‑progress diagnosis)

**`record-evidence`** — On green, assemble the durable **Evidence Ledger** entry: *what was validated, by what, and why* (criteria → witness → evidence, behavior preserved, justified test changes, decisions, limitations). Records only real evidence; an **output, never read back**.
- **Args:** `change` · `plan=<path>` · `baseline=<path>` · `run-record=<path>` · `reconcile-record=<path>` · `ledger=<path>`
- **Uses skills:** `evidence-ledger`
- **Needs:** the Validation Plan · the Behavior Baseline reconciliation · the run records · the test‑reconciliation record · decisions/limitations raised
- **Produces:** the durable per‑change Evidence Ledger entry (verdict `green` only on complete evidence)
- **Delegated by:** `drive-correction` *(on green)*

### The skills (operational reference each agent uses)

| Skill | What it specifies |
|---|---|
| `testing-strategy` | authoring the architecture‑aware Strategy + the coverage checklist |
| `validation-rules` | the Rule schema + deriving Rules from the Strategy |
| `source-map` | source locations + **authority** (who owns which claim) + typed tests |
| `change-taxonomy` | the change‑types + blast‑radius / test‑impact analysis |
| `criteria-identity` | per‑change stable AC ids + new/unchanged/moved/retired classification |
| `validation-plan` | the plan schema + derivation + AC→witness mapping |
| `behavior-baseline` | the behavior snapshot + capture/reconcile + the honesty rule |
| `execution-runner` | the run record + resolve/run/observe + fail‑fast ordering |
| `test-reconciliation` | the fate→action mapping + criteria provenance + honesty lock |
| `correction-loop` | diagnose → fix‑request handoff → re‑assess → re‑validate; regression vs brittle |
| `evidence-ledger` | the durable audit trail: criteria → witness → evidence; output, never read back |
| `escalation` | the structured shape of a decision (a question) vs a limitation (a toolkit gap) |

**`source-map.manifest.md`** is the one file you fill in per project — where your sources live and what each is authoritative for. A `.claude/` build will follow the same shape once this stabilises.

A `.claude/` build will follow the same shape once this stabilises. If you're extending the toolkit, start in the skills — they're terse and self‑describing.
