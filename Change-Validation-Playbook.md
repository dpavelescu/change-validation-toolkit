# Change‑Validation Toolkit — Playbook

> **Turn a change into the evidence needed to trust it** — derived from your testing strategy, checked against your real code, and self‑correcting.

A guide to what the toolkit is for, what it does, and how to use it, for the people who adopt and run it. It is tool‑neutral; the agents and skills under `.github/` are the build that implements it, catalogued at the end.

---

## Why it exists

A passing test suite is meant to tell you a change is correct. Often it doesn't. A test that asserts whatever the code already does will pass whether or not the change is right — it confirms the implementation, not the intent. Tools that generate tests tend to make this worse, producing green tests that lock in current behavior. And a change can break code it never mentions, in areas an unrelated story built.

This toolkit validates a change against its acceptance criteria — what the change is meant to do — rather than against the code as written. Each test traces back to a criterion, and no test is edited only to make it pass, so a green result reflects the criteria being met rather than the code agreeing with itself.

The rest of the design follows from that. It works out what a change affects, records current behavior before anything is touched, runs your suite, and feeds failures back through a correction loop. It does not pad coverage with generated tests, it does not write the code — it specifies tests and verifies them, and the implementation is handed off — and it does not sit between you and your CI.

### Why you can trust the output

Four rules hold throughout:

- **The acceptance criteria are the fixed point.** Correct means what the change is meant to do, not what the code happens to do.
- **The toolkit specifies and verifies; it doesn't write code.** It decides what each test must assert and what the code must satisfy, then checks the result. The authoring is handed off; the criteria never bend to fit the code.
- **Every test traces to a criterion.** A test defends one expectation. It checks the criterion; it is never the source of truth.
- **You make decisions, not fixes.** It asks a question only when the criteria themselves are unresolvable, and never hands you broken code.

---

## What it does for you

Some capabilities you set up once; the rest run on every change.

### Set up once — about your system, not any one change

- **Captures your testing strategy.** What kinds of evidence make a change trustworthy in your system. If you don't have a written strategy, it authors one with you from your architecture, asking one question at a time where the answer isn't already decided. It favours the lowest test level that gives solid confidence, and pushes back on slow end‑to‑end tests where a unit or component test would do. The strategy stays yours; you approve it.
- **Maps where truth lives.** Your sources — architecture, API specs, event schemas, tests, CI — and which one is authoritative for what: the API spec defines the contract, not the code. When something needs checking, it checks against the owner rather than guessing.

### Every change

- **Finds the blast radius.** Everything the change touches, including which existing tests it reaches — even tests written for old, unrelated work — worked out from the code, with no links for you to maintain.
- **Works out the evidence the change needs.** Two things, separately: that the new behavior works (the acceptance criteria), and that nothing around it broke (a regression check over everything the change reaches).
- **Separates intended change from breakage.** It records current behavior before the change. Afterward, a behavior that moved is either justified (a criterion moved with it) or a regression that nothing asked for, and it can tell which.
- **Specifies and verifies the tests.** It decides what each test asserts — from the criteria, not the new code — hands the authoring to your implementer, and checks that the result tests the criterion rather than the implementation. A test changes only when the requirement behind it moves.
- **Runs your own suite, cheap tests first.** Your suite, not an invented harness — unit and component tests first so problems surface early, then the slower integration and end‑to‑end tests, some of which only run in CI. It tells a real failure from a broken harness.
- **Drives correction by handoff.** When a test fails it diagnoses the cause and writes a structured fix‑request for whoever implements the code, then re‑validates. It doesn't write the production code, and it doesn't hand a person broken code — only the tests, the evidence, and the diagnosis.
- **Records what was validated.** On green it records what was validated, by what, and why — criteria, tests, evidence, the behavior preserved, the test changes made, and any decisions or limitations — so review is about behavior and decisions rather than internals.

> **Finding affected tests without stored links.**
> When a change touches code an old story's tests cover, those tests are found because the change reaches that code — recomputed each time from the code itself, never from a stored "relates to story X" link. Nothing is stored, so nothing goes stale. The strongest signal it uses is a **coverage map** it generates by running your suite (which it runs anyway): because that's recorded from real execution, it catches even dynamic wiring — dependency injection, reflection, event/queue hops — that static analysis can't see. Where the signal is weak it **widens the scope and flags it** rather than risk a miss: an imperfect radius costs a little extra runtime, never a false green. System‑level tests are caught by the flows they exercise, which is why the toolkit is told where each kind of test lives.

---

## How it works

**Three things you ever run** — everything else runs underneath them as a subagent:

| You run | When | It orchestrates underneath (subagents) |
|---|---|---|
| **`define-testing-strategy`** | once (and when architecture changes) | — *(authors the Strategy, generates the Rules)* |
| **`plan-validation`** | per change — to plan | `classify-change` · `reconcile-criteria` · `review-plan` |
| **`drive-correction`** | per change — to execute & correct | `capture-baseline` · `specify-tests` · `run-validation` |

Set up once, plan a change, drive it to green. You don't invoke the inner agents directly; the two per‑change entry points orchestrate them. The pipeline underneath is *classify → plan → record behavior → specify tests → run → correct*, with the **Execution Runner** running your suite.

### Scenarios it supports

The classes of change it handles, each with an example.

**① A new feature (new acceptance criteria).** It plans the required evidence, specifies (and verifies) a test for each criterion — authoring handed off — and drives the unmet ones to green via fix‑requests — never editing a test to pass.
> *Example — "add an email field to signup":* classified as a REST change; the blast radius pulls in the endpoint, the shared `UserService`, and an old end‑to‑end test found *from the code*. The new acceptance tests are red (feature not built) → fix‑request → you (or your impl agent) implement and re‑invoke → green.

**② A change that risks untouched‑but‑reachable code.** Its blast radius and behavior baseline catch a regression even in code some *other*, unrelated story built — with no links for you to maintain.
> *Example — a shared helper is tweaked:* nothing in the story asked for it, but an old order‑history endpoint now returns a different total. The delta maps to **no criterion → regression**, caught before anyone ships.

**③ A pure refactor (no behavior change intended).** Behavior‑preservation is the entire evidence: every touched behavior must match the pre‑change snapshot; any difference is a regression.
> *Example:* an `extract method` cleanup — the baseline pins every reachable surface beforehand and confirms each is identical after.

**④ A fix that breaks a brittle test.** The loop tells a real regression from a test merely coupled to internal structure: if the baseline says behavior is **preserved**, the test is *brittle* and gets repaired (decoupled); if behavior **moved** with no criterion, it's a regression to fix. The baseline is the gate — a regression can never be relabelled "brittle."
> *Example:* a fix extracts a method; a unit test bound to the old internals goes red while the endpoint's behavior is unchanged → repaired, not masked.

### Starting conditions — what you bring to it

What already exists around a change varies by project. Here is how the toolkit handles three common cases.

**Ⓐ No testing strategy yet.** It won't validate against a guessed standard. It authors a strategy with you first (`define-testing-strategy`), derived from your architecture, asking one question at a time where the answer isn't already implied, then generates the machine‑facing Validation Rules. You own and approve it. This is one‑time setup, not repeated per change; until it exists there is nothing to derive evidence from, so it comes first.

**Ⓑ A testing strategy already exists.** It's treated as the **authoritative, human‑owned source**: the toolkit reads it (via the Source‑Map), derives and refreshes the Validation Rules from it, and **never edits it silently**. When a change needs evidence the strategy doesn't cover, that surfaces as a **Strategy gap** — a *proposed* addition for you to approve, not a quiet under‑test. An informal or prose strategy can be normalized into the structured form (still yours to approve).

**Ⓒ The story comes with test specifications (e.g. BDD) — optional.** A behavioral spec that accompanies the change description is treated as a **specification of intended behavior: it _complements_ the acceptance criteria, it never bypasses them.** For **BDD/Gherkin**, the scenario *is* the criterion — the plan cross‑checks scenarios against criteria (a criterion with **no** scenario is a coverage gap; a scenario with **no** criterion is an orphan → a candidate criterion or a decision), and the toolkit **specifies the step definitions** (authoring handed off to your implementer) to run through *your* BDD runner, never re‑authoring the scenario as a separate test. Other spec styles (example tables, contract specs) follow the same rule. If **no** specs accompany the story, the criteria come from its acceptance criteria alone. Either way the **Validation Plan owns coverage** (criteria‑driven); the specs are inputs and tests, never the definition of what's covered.

### When it comes to you — and when it doesn't

When it needs you, it's one of two things:

- **A decision** — criteria ambiguous or contradictory, a public contract must change, something unsafe → a clear question with a recommended answer. You decide; you don't receive broken code.
- **A limitation** — it can't run a test, reproduce a failure, or build a fixture → a gap for the toolkit to close, logged as such rather than reframed as "a human handles this."

A failing test is neither — it's the loop's next step, and you hear about it only if it turns into a decision.

Both come in the **escalation** skill's structured shape: a decision carries its question, the context that forces it, a recommended resolution, the owning authority, and what it blocks; a limitation carries the gap, what it blocked, and what would close it. Both land in the Evidence Ledger.

---

## What it needs from you

- **Real acceptance criteria.** No criteria → you're still clarifying *what* to build (the companion work‑item toolkit's job); it says so rather than invent correctness.
- **A runnable suite and a seeded source map.** It runs *your* tests and resolves *your* sources from locations you provide — it never guesses or fakes, and reports a gap if it can't.

**Strongest fit:** brownfield systems where regressions hide and the same code is touched by many unrelated stories over time.

**What you get back, per change:** the evidence each requirement has (or a flagged gap), what ran and passed, what it **couldn't** do and why (its own gaps), and any **decisions** it needs. Nothing is "done" without evidence.

---

## Artifacts — what's human-readable, and what counts as evidence

Most of what the toolkit writes is operational: machine-readable records its agents pass along and replay — the change classification, criteria ids, behavior baseline, run records, and the test- and fix-requests. They underlie the evidence but aren't the record you would hand to a reviewer.

Four artifacts are human-readable documentation, and these are what serve as **process evidence**, including in regulated or audited environments:

- **Testing Strategy** — the human-owned document of what evidence makes a change trustworthy in your system. A controlled document you author and approve.
- **Validation Plan** (approved) — for one change, the evidence it requires, the test dispositions, and your approval. The record of what was planned and signed off.
- **Evidence Ledger** — the durable, per-change audit trail: each criterion, the test that checked it, the evidence observed, the behavior preserved, the test changes and their justification, and any decisions or limitations. The primary record that a change was validated, by what, and why.
- **Decisions & limitations** — the judgment calls made (the question, context, resolution, and who owns it) and the gaps the toolkit hit. An auditable trail of where a human decided and where the toolkit fell short.

The Evidence Ledger references the operational records rather than copying them, so the audit trail stays readable while the detail stays reproducible. Whether these satisfy a particular standard is your determination; the toolkit's job is to produce them completely and to flag anything missing rather than imply coverage it doesn't have.

---

## How tests run — and what's in scope

What the toolkit does itself depends on the test category. Each sits in one of four tiers:

| Tier | Categories | What the toolkit does |
|---|---|---|
| **① Native** | unit · component | Specifies and verifies the tests, runs them, and reads results — locally and fast. Authoring is handed off to your implementer. |
| **② With your environment** | integration · contract · end-to-end | Specifies and verifies, then runs them where your environment allows — locally if the infra is up, else in CI. It uses your environment; it doesn't stand one up. |
| **③ External** | performance · load · failure/resilience · security-scan · accessibility | Out of scope by default — your pipeline owns these. Opt-in, it reads your result to record an audit trace and flag an NFR nothing covers; it reads only, and never gates. |
| **④ Out of scope** | provisioning environments, production code, manual/exploratory | Left to you and your team. |

In practice it works in tiers ① and ②, leaving the rest to you; for external NFRs it can record an opt-in audit trace. It doesn't insert itself into a flow you already run.

A few practical points:
- **Your frameworks, your runner.** The test-request specifies the idiomatic style for your stack; your implementer authors it; it runs via your own test command, not an invented harness. Results come from your machine-readable report (JUnit XML, etc.), not the console.
- **BDD.** A Gherkin scenario is the acceptance criterion. The toolkit specifies the step definitions and runs them through your BDD tooling rather than forking a separate test. Coverage stays defined by the criteria and cross-checked against your scenarios, so a criterion with no scenario, or a scenario with no criterion, is surfaced.
- **CI is a participant, not a trigger.** Locally it runs the local gate; your normal push/PR pipeline runs the CI gate. It reads the same report in both, so local and CI evidence are comparable.
- **Environment checked first.** If a category's environment isn't available where it should run, that's flagged as a limitation up front rather than producing a test that can't execute.

---

## Where it is today

**Specified end‑to‑end:** capturing the strategy, mapping sources and authority, classifying a change and its blast radius, planning the evidence, recording behavior, running your suite, specifying and verifying tests, driving correction to green via structured fix‑requests (implementation handed off), and recording the Evidence Ledger.

**Next:** the parallel `.claude/` build. This is the Copilot‑first build; the model is specified end‑to‑end.

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

**`classify-change`** — Classify a change into types, compute its **blast radius** (test‑impact analysis — including which existing tests it reaches), resolve the sources it needs, and record which source is **authoritative** for each crossed claim.
- **Args:** `change=<diff|branch|PR|description>` · `criteria=<acceptance criteria|link>` *(optional)*
- **Uses skills:** `change-taxonomy`, `validation-rules`, `source-map`
- **Needs:** the change · the Validation Rules · the Source‑Map
- **Produces:** change‑types · blast radius · affected tests (by type) · needed sources · claim authorities

**`reconcile-criteria`** — Give each acceptance criterion a **stable id for this change's run** (`new`/`unchanged`/`moved`/`retired` vs the existing suite). Read‑only on the story; ids are **per‑change** (stable across local↔CI, discarded after merge) — *not* a durable or cross‑change record.
- **Args:** `story=<link|file>` · `classification=<path>` *(affected tests)* · `criteria-ids=<path>` *(this change's run state; default `.validation/<change>/criteria.md`)*
- **Uses skills:** `criteria-ids`, `source-map`
- **Needs:** the story's ACs (Source‑Map kind `acceptance-criteria`) · the existing tests the change reaches (from the blast radius)
- **Produces:** the per‑change Criteria IDs + a **provisional** delta summary (new / moved / retired — the Baseline confirms moved/retired)
- **Delegated by:** `plan-validation`

**`plan-validation`** — *orchestrator* — Derive the **Validation Plan**: per‑AC required evidence and test map, provisional test dispositions, the regression (behavior‑preservation) track, and the local/CI gates.
- **Args:** `change=<diff|branch|PR>` · `story=<link|file>` · `classification=<path>` *(reuse)*
- **Uses skills:** `validation-plan`, `change-taxonomy`
- **Delegates to:** `classify-change`, `reconcile-criteria`, `review-plan`
- **Needs:** the change · the story · the Validation Rules · the Source‑Map
- **Produces:** the Validation Plan (a draft you approve) — or *Not ready* with a resumable agenda

**`review-plan`** — *gate lens* — Review the assembled plan before capture: coverage, disposition justification, testability, blast‑radius regression, **test‑level discipline**, honesty, decisions. Read‑only.
- **Args:** none of its own (invoked by `plan-validation`)
- **Uses skills:** `validation-plan`, `criteria-ids`
- **Needs:** the assembled plan · the Criteria IDs
- **Produces:** findings + a recommendation (ready to capture / Not ready)
- **Delegated by:** `plan-validation`

### Execute a change — *Phase 3; runs your suite, drives correction*

**`capture-baseline`** — **Record current behavior** across the plan's surfaces before the change, then reconcile post‑change deltas into **justified** (a criterion moved) vs **regression** (none did).
- **Args:** `change` · `classification=<path>` · `plan=<path>` · `criteria-ids=<path>` · `baseline=<path>`
- **Uses skills:** `behavior-baseline`, `change-taxonomy`
- **Delegates to:** `run-validation`
- **Needs:** the plan's behavior‑preservation track · the classification · the criteria IDs deltas · the pre‑change state
- **Produces:** the Behavior Baseline + a delta reconciliation

**`run-validation`** — Drive **your own test suite** over the blast‑radius slice (**cheapest/local first**), returning structured observations and a determinism verdict. The execution substrate; runs, never edits.
- **Args:** `change` · `classification=<path>` · `plan=<path>` · `gate=local|ci` · `run-record=<path>`
- **Uses skills:** `execution-runner`
- **Needs:** the blast radius · the plan's local/CI gates · your **own test runner** · the Source‑Map `tests` + `test-report`
- **Produces:** a run record + observations · recheck list (clean fails) · limitations
- **Used by:** `capture-baseline`, `specify-tests`

**`specify-tests`** — **Own what each test asserts** (from the criteria/baseline) and **verify** the authored result faithfully checks the criterion — never coupled to the implementation. Emits a **`test-request`** for the external implementer to author; **writes no code**. An AC is done only on green evidence.
- **Args:** `change` · `plan=<path>` · `criteria-ids=<path>` · `baseline=<path>` · `test-requests=<path>`
- **Uses skills:** `test-reconciliation`, `escalation`
- **Delegates to:** `run-validation`
- **Needs:** the Validation Plan · the Criteria IDs · the Behavior Baseline · the Source‑Map `tests`
- **Produces:** `test-request`s (handoffs) · verified tests + evidence · a reconciliation record

**`drive-correction`** — Drive a failing change **to green by handoff**: diagnose each failure into a structured **fix‑request** for whoever implements (a human or any implementation agent), re‑assess impact, re‑validate, and iterate. **Never writes production code.** Resumable — emits handoffs and pauses; re‑invoked after each external fix.
- **Args:** `change` · `plan=<path>` · `baseline=<path>` · `run-record=<path>` · `fix-requests=<path>` · `max-iterations=<n>`
- **Uses skills:** `correction-loop`
- **Delegates to:** `run-validation`, `classify-change` *(re‑assess)*, `capture-baseline` *(sort)*, `specify-tests` *(test fixes)*
- **Needs:** the Validation Plan · the Behavior Baseline · the latest run record (the recheck list)
- **Produces:** one of — **green** (evidence) · **fix‑requests** (handoffs) · **re‑plan** (scope grew) · a **decision** · an **escalation** (no‑progress diagnosis)

**`record-evidence`** — On green, assemble the durable **Evidence Ledger** entry: *what was validated, by what, and why* (criteria → test → evidence, behavior preserved, justified test changes, decisions, limitations). Records only real evidence; an **output, never read back**.
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
| `criteria-ids` | per‑change stable AC ids + new/unchanged/moved/retired classification |
| `validation-plan` | the plan schema + derivation + AC→test mapping |
| `behavior-baseline` | the behavior snapshot + capture/reconcile + the no-edit-to-pass rule |
| `execution-runner` | the run record + resolve/run/observe + fail‑fast ordering |
| `test-reconciliation` | the disposition→action mapping + criteria provenance + the no-edit-to-pass rule |
| `correction-loop` | diagnose → fix‑request handoff → re‑assess → re‑validate; regression vs brittle |
| `evidence-ledger` | the durable audit trail: criteria → test → evidence; output, never read back |
| `escalation` | the structured shape of a decision (a question) vs a limitation (a toolkit gap) |

**`source-map.manifest.md`** is the one file you fill in per project — where your sources live and what each is authoritative for. A `.claude/` build will follow the same shape once this stabilises.

If you're extending the toolkit, start in the skills — they're terse and self‑describing.
