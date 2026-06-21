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
- **Closes the loop** *(coming next)*. When a test fails, it diagnoses and fixes the *code* and re‑runs — instead of handing the failure back — and keeps a plain‑language record of what was checked and why.

> **The capability you should understand: finding affected tests without links.**
> When a change touches code that an old story's tests cover, those tests are found **because the change reaches that code** — discovered fresh every time from the code itself (call graph and/or coverage), never from a stored "this change relates to story X" reference. References rot; this can't, because there's nothing to keep. System‑level tests (end‑to‑end, integration) are caught too, by the *flows* they exercise — which is why the toolkit is told where each kind of test lives.

---

## How it works

A change flows through the toolkit roughly like this. You don't have to drive each step by hand once it's set up — but it helps to know what's happening.

1. **Classify the change** and find everything it touches (its blast radius), including the existing tests it reaches.
2. **Look up the rules** — for this kind of change, what evidence your strategy says matters.
3. **Pin the requirements** — give each acceptance criterion a stable handle so tests and evidence can point at it.
4. **Make a plan** — per requirement, what evidence is needed and which test provides it; plus a regression guard for everything touched that no requirement covers.
5. **Photograph today's behavior** so intended change can be told from accidental breakage.
6. **Run your own tests** to take that photograph and, after the change, to gather evidence.
7. **Write or adjust the tests** — from the requirements, never the new code.
8. **Close the loop** *(coming next)* — fix failing code, re‑run, and record what was checked.

### When it comes to you — and when it doesn't

This is the part worth internalising, because it's how the toolkit stays autonomous without quietly dumping work back on you. Every time a human is involved, it's exactly one of two things:

- **A decision (it needs your judgment).** The criteria are ambiguous or contradict each other; the change crosses into another team's contract; something is unsafe. → It asks you a **clear question**. You answer a question — you're never handed broken code.
- **A limitation (its own gap to close).** It can't run a test, reproduce a failure, or build a fixture. → That's the **toolkit's** problem to fix, logged as such. It is *never* quietly reframed as "a human will handle this case."

A failing test is **neither** of those — it's just the loop's next input. You hear about it only if it turns into a genuine decision.

---

## How to use it

### Getting started

1. **Add the build to your repo** and fill in the **source map** — point it at where your architecture, specs, schemas, tests, and CI config live.
2. **Run the strategy step.** It drafts (or updates) your testing strategy from your architecture, clarifying the few genuinely‑open expectations one question at a time, and you approve the result.
3. **Point it at a change** — a diff, branch, or PR — together with the story's acceptance criteria. It classifies, plans, and (in the execution phase) produces, runs, and self‑corrects the evidence.

### What it needs from you

- **Real acceptance criteria.** If a change has no criteria, you're not validating yet — you're still clarifying *what* to build (that's the companion work‑item‑preparation toolkit's job). It will say so rather than invent correctness.
- **A runnable suite.** It validates by running your tests; if it can't run them, that's a gap it reports, not something it papers over.
- **A seeded source map.** It resolves sources deterministically instead of guessing — but you provide the initial locations.

### Where it's the strongest fit

Brownfield systems where regressions hide, where the same code is touched by many unrelated stories over time, and where "did this change break something we forgot about?" is a real and recurring fear. That's exactly the case its blast‑radius and behavior‑baseline capabilities are built for.

### How to read what it gives you

For a change, it tells you: what evidence each requirement has (and any with none yet), what it ran and what passed, what it **couldn't** do and why (its own gaps), and any **decisions** it needs from you. Nothing is reported as "done" without evidence behind it.

---

## Example

*The following is one illustration on a specific stack — not a fixed menu. What counts as "the evidence a REST change needs" comes from **your** strategy, not from the toolkit.*

> **A team adds an email field to the signup endpoint** (a Spring Boot + React + SNS/SQS system).
>
> 1. It's classified as a REST API change. The blast radius includes the signup endpoint, the shared `UserService` it calls, and — found from the code, not from any stored link — an **end‑to‑end signup test written months ago for a different story**.
> 2. The strategy says a REST change here needs: the endpoint behaves, errors map correctly, and the public contract stays backward‑compatible. Backward‑compatibility is checked against the **API spec** (the contract's owner), not the controller code.
> 3. The new acceptance criteria — "accepts a valid email," "rejects a malformed one" — each get a stable handle.
> 4. The plan: a test for each new criterion, *plus* a regression guard on `UserService` and that old e2e flow, which no new criterion covers.
> 5. Current behavior is photographed first. After the change, the e2e flow still passing is evidence the change didn't break it; if it had changed with no criterion asking for it, that's a **regression** — the loop's next task, not a question for you.
> 6. The new tests are written from the criteria. One fails because the code doesn't validate email yet → that's loop input; the code gets fixed and re‑run. None of it edits a test to force it green.

> **A pure refactor** (no behavior is supposed to change). There are no new criteria, so *every* behavior the change touches must stay identical. The behavior photograph **is** the evidence: any difference at all is a regression.

> **A caught regression.** A change tweaks a shared helper and, with nothing asking for it, an old order‑history endpoint starts returning a different total. No criterion justifies that delta → it's flagged as a regression and fixed, before anyone ships it.

---

## Where it is today

**Available and specified end‑to‑end through test reconciliation:** capturing the strategy, mapping sources and authority, classifying a change and its blast radius, planning the evidence, photographing behavior, running your suite, and writing/adjusting tests honestly.

**Coming next:** the self‑closing **auto‑fix loop** (diagnose a failure, fix the code, re‑run — proposing fixes as commits, never silent edits to protected branches) and the **audit trail** of what was checked and why, so a human review is fast.

---

## The toolkit — agents & skills (catalog)

This Playbook is the concept; the running build lives under `.github/`. Below is every **agent** you can run — its purpose, arguments, the **skills** it uses, what it depends on, and what it produces — followed by a one‑line index of the skills. Each agent is self‑contained; you run them in the order below.

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

**`reconcile-criteria`** — Give each acceptance criterion a **stable id** by reconciling the story's ACs against the Criteria Ledger (match / move / add / retire). Read‑only on the story.
- **Args:** `story=<link|file>` · `ledger=<path>` *(default `.validation/<change>/criteria.md`)*
- **Uses skills:** `criteria-ledger`, `source-map`
- **Needs:** the story's ACs (Source‑Map kind `acceptance-criteria`) · the prior ledger if any
- **Produces:** the updated Criteria Ledger + a delta summary (new / moved / retired)
- **Delegated by:** `plan-validation`

**`plan-validation`** — *orchestrator* — Derive the **Validation Plan**: per‑AC required evidence and witness map, provisional test fates, the regression (behavior‑preservation) track, and the local/CI gates.
- **Args:** `change=<diff|branch|PR>` · `story=<link|file>` · `classification=<path>` *(reuse)*
- **Uses skills:** `validation-plan`, `change-taxonomy`
- **Delegates to:** `change-classifier`, `reconcile-criteria`, `validation-plan-reviewer`
- **Needs:** the change · the story · the Validation Rules · the Source‑Map
- **Produces:** the Validation Plan (a draft you approve) — or *Not ready* with a resumable agenda

**`validation-plan-reviewer`** — *gate lens* — Review the assembled plan before capture: coverage, fate justification, testability, blast‑radius regression, **test‑level discipline**, honesty, decisions. Read‑only.
- **Args:** none of its own (invoked by `plan-validation`)
- **Uses skills:** `validation-plan`, `criteria-ledger`
- **Needs:** the assembled plan · the Criteria Ledger
- **Produces:** findings + a recommendation (ready to capture / Not ready)
- **Delegated by:** `plan-validation`

### Execute a change — *Phase 3; runs your suite (auto‑fix loop forthcoming)*

**`capture-baseline`** — **Photograph current behavior** across the plan's surfaces before the change, then reconcile post‑change deltas into **justified** (a criterion moved) vs **regression** (none did).
- **Args:** `change` · `classification=<path>` · `plan=<path>` · `ledger=<path>` · `baseline=<path>`
- **Uses skills:** `behavior-baseline`, `change-taxonomy`
- **Delegates to:** `run-validation`
- **Needs:** the plan's behavior‑preservation track · the classification · the ledger deltas · the pre‑change state
- **Produces:** the Behavior Baseline + a delta reconciliation

**`run-validation`** — Drive **your own test suite** over the blast‑radius slice (**cheapest/local first**), returning structured observations and a determinism verdict. The execution substrate; runs, never edits.
- **Args:** `change` · `classification=<path>` · `plan=<path>` · `gate=local|ci` · `run-record=<path>`
- **Uses skills:** `execution-runner`
- **Needs:** the blast radius · the plan's local/CI gates · the Source‑Map `build-commands`/`ci-config`/`tests`
- **Produces:** a run record + observations · loop‑input (clean fails) · limitations
- **Used by:** `capture-baseline`, `implement-tests`

**`implement-tests`** — Materialize and adjust the **witnessing tests** per the plan's fates — criterion witnesses from the criteria, regression witnesses from the baseline. The **independent test‑implementer**; never edits production code.
- **Args:** `change` · `plan=<path>` · `ledger=<path>` · `baseline=<path>`
- **Uses skills:** `test-reconciliation`
- **Delegates to:** `run-validation`
- **Needs:** the Validation Plan · the Criteria Ledger · the Behavior Baseline · the Source‑Map `tests`
- **Produces:** the materialized/adjusted tests + a reconciliation record

### The skills (operational reference each agent uses)

| Skill | What it specifies |
|---|---|
| `testing-strategy` | authoring the architecture‑aware Strategy + the coverage checklist |
| `validation-rules` | the Rule schema + deriving Rules from the Strategy |
| `source-map` | source locations + **authority** (who owns which claim) + typed tests |
| `change-taxonomy` | the change‑types + blast‑radius / test‑impact analysis |
| `criteria-ledger` | stable AC ids + the match/move/add/retire reconciliation |
| `validation-plan` | the plan schema + derivation + AC→witness mapping |
| `behavior-baseline` | the behavior snapshot + capture/reconcile + the honesty rule |
| `execution-runner` | the run record + resolve/run/observe + fail‑fast ordering |
| `test-reconciliation` | the fate→action mapping + criteria provenance + honesty lock |

**`source-map.manifest.md`** is the one file you fill in per project — where your sources live and what each is authoritative for. A `.claude/` build will follow the same shape once this stabilises.

A `.claude/` build will follow the same shape once this stabilises. If you're extending the toolkit, start in the skills — they're terse and self‑describing.
