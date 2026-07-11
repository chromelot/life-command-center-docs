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
5. **One deliverable per turn.** One contract table (or one decision prompt) per assistant message unless Phase 0 silent pulls.

## Aaron input — lettered option tables (never AskQuestion)

**Do not use the AskQuestion tool** in Tier 2 workflows. When the Cursor UI renders AskQuestion, the agent response is often hidden or partially rendered.

When Aaron must choose:

1. Append a **Decision** table after the contract table(s) for the step.
2. Label options **A, B, C, …** (single-select) or note **multi-select — reply with letters** (e.g. `A, C, F`).
3. One decision table per turn; wait for Aaron's letter reply before `advance`.

Example:

| | Option |
|---|--------|
| **A** | Approve this write |
| **B** | Skip |
| **C** | Stop phase |

**Reply with:** letter(s) only (e.g. `B` or `A, C`).

**Dev planning:** Dev Projects `This Week` tracker is canonical state. Meeting logs are cross-check only. **Phase 2.1-A** presents that queue for the **week under review** (CL/TG `This Week` at review time); **2.1-G** writes the **upcoming** week’s queue. Work not in Dev Projects → create a record before committing the week.

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
| 13 | `2.1` | Dev Review — CL/TG last week + monthly selection |
| 14 | `2.2` | Systems/Workshop/Admin Review — deep-work=Systems; Workshop cap + Admin blocks; Todoist mirrors |
| 15 | `2.check` | FIELD CHECK — **gate before commit** |
| 16 | `4` | Commit checklist + all FIELD CHECKs |

Ops → **Weekly Ops** workflow.

## Weekly ops — sub-step order

| Step | Contract |
|------|----------|
| `pre-0`, `0`, `0b` | Gate + pull + RED FLAGS table |
| `1.1` | Work volume + busier + slower customer tables |
| `1.2` | PD activity scorecard |
| `1.3` | Work Management — daily records, backups, ops priority |
| `2.1` | Hiring — job posts, ad health, pipeline actions + Delegation & Hiring / Personnel dept health |
| `2.4` | Photographer Management — roster, perf behind, monitoring + dept health |
| `6.2` | Social Media — P&L, per-account posting, pipeline gaps/stale + dept health |
| `7.1` | 1:1 meetings — late 1:1 visits |
| `2.2` | CS check-in cadence + behind CS visits |
| `2.3` | HJD review + Customer Service dept health (one customer/turn when flagged) |
| `4.1` | Finance & Admin — invoices + Admin dept health (one customer/turn) |
| `4.2` | Finance & Admin — QuickBooks / quarterly estimates placeholder |
| `3.1`, `3.2` | Aaron sales (stale deals, overdue PD), team oversight + Account Management dept health |
| `6.1` | Post Production — QA / delivery backlog |
| `check` | FIELD CHECK — **gate before commit** |
| `commit` | Commit checklist |

## Session workflows

| Workflow | Steps | Contract doc |
|----------|-------|--------------|
| person-checkin | `0` → `1.0` → `1.1a`–`1.1c` → `2.0` | person-checkin.mdc |
| team-sync | `0` → `1.0` → `1.1` → `1.2` → `2.0` | team-sync.mdc |
| pd-cleanup | `0` → `1.0` → `1.1`–`4.1` → `5.0` | pd-cleanup SKILL |

## Monthly plan — sub-step order

| Step | Content |
|------|---------|
| `0` | Silent data pull |
| `1.0` | Create Notion log |
| `1.1`–`1.4` | Wellness → identity → quarterly gate → sustained unhealthy |
| `1.check` | Phase 1 FIELD CHECK — `gate --phase 2` |
| `2.1`–`11.2` | Domain phases per SKILL Present map |
| `12.0` | Commit — `gate --phase 12` then complete |

## Quarterly plan — sub-step order

| Step | Content |
|------|---------|
| `0` | Silent pre-flight |
| `A.0` | Create Notion log |
| `A.1`–`A.check` | Part A retrospective |
| `B.1`–`B.3` | People, environment, capacity — `gate --phase C` |
| `C.1`–`C.gate` | Personal, CL, TG strategy — `gate --phase D` |
| `D.1`–`D.check` | Tactical quarter — `gate --phase E` |
| `E.1`–`E.4` | System, calendar, context, commit + complete |

## See also

- `context/systems/workflow-output-contracts.md` — table contracts (gold standard: weekly plan)
- `context/systems/workflow-logs.md` — Notion log map
- `scripts/lib/workflow-registry.mjs` — step lists
- `.cursor/rules/workflow-execution.mdc` — Cursor enforcement