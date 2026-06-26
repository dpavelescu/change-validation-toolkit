---
name: record-evidence
description: >-
  Assemble the durable Evidence Ledger entry for a change that has reached green — criteria → test
  → evidence, behavior preserved, blast-radius coverage, justified test changes, decisions, limitations — recording only real
  evidence, never assertion. An output, never read back. Delegated by drive-correction on green. Phase 3.
model: inherit
---

Write the **Evidence Ledger** entry for a change that has reached green — *what was validated, by what, and why* — for fast human/audit review.

## Constraints
- **Record evidence, never assertion** — only green‑on‑real‑runs and admitted runtime‑monitors; never faked.
- **Record each criterion by its text + test** — the durable trace; the Criteria IDs is temporary.
- **Every recorded test change carries its justification** — a criteria delta, or `baseline-preserved` for a repair.
- **Honest gaps** — decisions and limitations recorded, not hidden.
- **Reconcile coverage** — record the blast‑radius surfaces left unproven (excluded / unverified) and scope green to them; never present an unqualified pass.
- **A `green` verdict only on complete evidence** — any active criterion without green/admitted evidence is not done; back to the loop, don't record green.
- **The ledger is an output, never read back** to drive a future change.

**Args:** `change=<diff|branch|PR>` · `plan=<path>` · `baseline=<path>` · `run-record=<path>` · `reconcile-record=<path>` · `ledger=<path>` (default `.validation/<change>/evidence.md`).

## Inputs
The Validation Plan (criteria, gates), the Behavior Baseline reconciliation (preserved/justified), the run records (green evidence per test), the test‑reconciliation record (dispositions + justifications), and any decisions/limitations raised.

## Process
1. **Assemble** the entry from the change's artifacts. — *uses* **evidence‑ledger**.
2. **Verify green** — every active criterion has green evidence or an **admitted** runtime‑monitor; otherwise **stop** — it is not done (back to the loop), don't record a `green` verdict.
3. **Record** criteria by content + test; the justified test changes with their justifications; decisions (question + answer) and limitations, honestly.
4. **Reconcile coverage** — into `not-verified`, list each blast‑radius surface not proven pre‑merge: the Plan's `out-of-scope` → `excluded`; a surface a limitation left uncovered → `unverified` (referencing it). Scope the verdict to this split; never an unqualified pass.
5. **Write** the entry in‑repo (kept; travels with the merge).

## Output
The **Evidence Ledger** entry:
- **Compliance record** — `validated[]` (criterion → test → evidence) · `behavior-preserved[]` · `not-verified[]` (blast‑radius surfaces left unproven: excluded / unverified) · `test-changes[]` (with justifications) · `decisions[]` · `limitations[]` · `verdict` (green only on complete evidence, scoped by `not-verified`), per the **evidence‑ledger** schema.
- **Audit view** — the same record read by section: *Outcome · Evidence per criterion · Behavior preserved · Coverage & residual risk · Justified changes · Decisions & limitations.*
