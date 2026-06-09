> **Source:** [`context/workflow-execution.md`](https://github.com/chromelot/life-command-center/blob/main/context/workflow-execution.md) in the private workspace repo. Do not edit this mirror directly.

# Workflow Execution Protocol

Canonical rules for **all Tier 2 Cursor workflows**. Skills define *what* to present (output contracts); this file defines *how* to execute so output is **consistent week to week** and sections are **never skipped**.

## Core problem

Skills can be correct while execution drifts: vague summaries instead of contract tables, jumping phases before FIELD CHECK passes, batching questions, or advancing after tangents. The **ledger + output contract + Notion log** trio prevents that.

## Read order (workflow start)

1. `context/workflow-execution.md` (this file)
2. `context/systems/workflow-output-contracts.md` — table specs, section completion, anti-patterns
3. `context/systems/workflow-logs.md` — Notion schema + phase field writes
4. The workflow skill or `.cursor/rules/<workflow>.mdc`

## Determinism principles

1. **Skill = spec.** Present exactly the tables, headers, and questions defined for `current_step`. Do not improvise sections, metrics, or narrative structure.
2. **Finish before advancing.** A step is incomplete until its output contract is fully presented, required Aaron input is collected, and step-specific Notion fields are written (if any). Then `advance`. Never "move on and circle back."
3. **Gate before phase change.** Run `workflow-progress.mjs gate` and present the skill's FIELD CHECK table before the next major phase. Failed check = hard stop.
4. **Same inputs → same shape.** Use named pull scripts and skill-listed sources. Do not re-query ad-hoc or substitute prose for tables.
5. **One deliverable per turn.** One contract table (or one question) per assistant message unless Phase 0 silent pulls.

Full contract anatomy → `context/systems/workflow-output-contracts.md`.

## Mandatory behaviors (every workflow)

1. **Initialize ledger** at workflow start:
   ```
   node scripts/workflow-progress.mjs init --workflow <slug> [--week-of YYYY-MM-DD]
   ```

2. **Create Notion log** at step `1.0` (or first user-facing step after pull):
   ```
   node scripts/workflow-notion-log.mjs create --ledger output/workflow-progress/<file>.json
   ```

3. **Every turn:**
   ```
   node scripts/workflow-progress.mjs status --workflow <slug>
   ```
   Present **only** what the skill assigns to `current_step`.

4. **Phase banner:**
   ```
   **[<Workflow Name> · Phase X.Y — <Sub-step title>]**
   ```

5. **Output contract** — Copy table headers from skill verbatim. Fill every cell; `—` when data missing. See output-contracts doc.

6. **Section completion checklist** — Before `advance`, verify all items in `workflow-output-contracts.md` § Section completion checklist.

7. **Advance + sync** — Only after step complete:
   ```
   node scripts/workflow-progress.mjs advance --workflow <slug> --step <id>
   node scripts/workflow-notion-log.mjs sync --ledger <path>
   ```

8. **Phase-end writes** — Structured fields to Notion per `workflow-logs.md`.

9. **FIELD CHECK** — Present verification table; do not proceed on failure.

10. **Tangents** — Handle, re-run `status`, resume `current_step` (not next step).

11. **Session end** — State `current_step`, Notion URL, next sub-step + next table ID.

12. **Complete** — `workflow-notion-log complete` after final FIELD CHECK passes.

## Weekly plan — dates

```
node scripts/planning-dates.mjs
```
America/Chicago; do not trust stale ledger `week_of`.

## Weekly plan — sub-step order

| Order | Step ID | User-facing? |
|-------|---------|----------------|
| 1 | `pre-0` | Yes — monthly gate |
| 2 | `0` | Silent pulls |
| 3 | `0b` | Yes — Table 0b (data integrity) |
| 4 | `1.0` | Yes — create Notion log |
| 5–11 | `1.1`–`1.7` | Life review — one domain per step, contract tables per skill |
| 12 | `1.check` | FIELD CHECK — **gate before Phase 2** |
| 13 | `2.1`–`2.4` | Development tables per step |
| 14 | `2.check` | FIELD CHECK — **gate before commit** |
| 15 | `4` | Commit checklist + all FIELD CHECKs |

Ops → **Weekly Ops** workflow.

## Weekly ops — sub-step order

| Step | Contract |
|------|----------|
| `pre-0`, `0`, `0b` | Gate + pull + RED FLAGS table |
| `1.1`, `1.2` | Tables 1-A, 1-B |
| `2.1`, `2.2` | CS cadence, invoices (one customer/turn in 2.2) |
| `3.1`, `3.2` | Aaron sales, team oversight |
| `4.1`, `4.2` | Photographers (one/turn), staff table |
| `check` | FIELD CHECK — **gate before commit** |
| `commit` | Commit checklist |

## Session workflows

| Workflow | Steps | Contract doc |
|----------|-------|--------------|
| person-checkin | `0` → `1.0` → `1.1a`–`1.1c` → `2.0` | person-checkin.mdc |
| team-sync | `0` → `1.0` → `1.1` → `1.2` → `2.0` | team-sync.mdc |
| pd-cleanup | `0` → `1.0` → `1.1`–`4.1` → `5.0` | pd-cleanup SKILL |

## Monthly / quarterly

Same determinism rules. One sub-step / one contract block per turn. FIELD CHECK at part boundaries per skill.

## See also

- `context/systems/workflow-output-contracts.md` — table contracts (gold standard: weekly plan)
- `context/systems/workflow-logs.md` — Notion log map
- `scripts/lib/workflow-registry.mjs` — step lists
- `.cursor/rules/workflow-execution.mdc` — Cursor enforcement