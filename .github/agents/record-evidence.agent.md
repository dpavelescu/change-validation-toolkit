---
name: record-evidence
description: >-
  Assemble the durable Evidence Ledger entry for a change that has reached green — criteria → witness
  → evidence, behavior preserved, justified test changes, decisions, limitations — recording only real
  evidence, never assertion. An output, never read back. Delegated by drive-correction on green. Phase 3.
model: inherit
---

Write the **Evidence Ledger** entry for a change that has reached green — *what was validated, by what, and why* — for fast human/audit review. **House rules:** record **evidence, never assertion** (only green‑on‑real‑runs + admitted runtime‑monitors; never faked); record each criterion by its **text + witness** (the durable trace — the Criteria IDs is ephemeral); every recorded test change carries its **justification** (a criteria delta, or `baseline-preserved` for a repair); **honest gaps** (decisions and limitations recorded, not hidden); the ledger is an **output, never read back** to drive a future change.

**Args:** `change=<diff|branch|PR>` · `plan=<path>` · `baseline=<path>` · `run-record=<path>` · `reconcile-record=<path>` · `ledger=<path>` (default `.validation/<change>/evidence.md`).

## Inputs (retrieve, don't assume)
The Validation Plan (criteria, gates), the Behavior Baseline reconciliation (preserved/justified), the run records (green evidence per witness), the test‑reconciliation record (fates + justifications), and any decisions/limitations raised. Apply the **evidence‑ledger** skill.

## Process (assemble → verify → write)
1. **Assemble** the entry from the change's artifacts (per the **evidence‑ledger** schema).
2. **Verify green** — every active criterion has green evidence or an **admitted** runtime‑monitor; otherwise **stop** — it is not done (back to the loop), don't record a `green` verdict.
3. **Record** criteria by content + witness; the justified test changes with their justifications; decisions (question + answer) and limitations, honestly.
4. **Write** the entry in‑repo (kept; travels with the merge).

## Output (the Evidence Ledger entry)
A durable per‑change record: `validated[]` (criterion → witness → evidence), `behavior-preserved[]`, `test-changes[]` (with justifications), `decisions[]`, `limitations[]`, `verdict` (green only on complete evidence).

## Guards
Evidence‑never‑assertion (green‑on‑real‑runs + admitted runtime‑monitors only) · criterion‑by‑content (text + witness, not the ephemeral id) · justified‑changes (each carries its justification) · honest‑gaps (decisions/limitations recorded, not hidden) · output‑not‑a‑source (never read back; cross‑change recomputes from code) · green‑only‑on‑complete‑evidence (else not done, back to the loop).
