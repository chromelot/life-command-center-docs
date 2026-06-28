> **Source:** [`context/systems/workflow-output-contracts.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/workflow-output-contracts.md) in the private workspace repo. Do not edit this mirror directly.

# Workflow Output Contracts

**Skills are specs, not suggestions.** Tier 2 workflows must produce the **same shape of output week to week**. The weekly plan's numbered tables (fixed headers, named data sources, one sub-step per turn) are the gold standard. Vague prompts ("review fitness", "discuss sales") are not acceptable execution instructions.

## Why this exists

Without output contracts, two agents running the same workflow produce different sections, skip cells, batch questions, or advance before a phase is actually done. Contracts make output **auditable** and **resumable**.

## Three enforcement layers (all required)

| Layer | Mechanism | Blocks |
|-------|-----------|--------|
| **Step order** | `workflow-progress.mjs` ledger + `advance` | Skipping to step N+1 before N is marked complete |
| **Phase gates** | `gate --phase N` + FIELD CHECK tables in skill | Opening Phase 2 while Phase 1 fields are blank |
| **Output contract** | Per-step table/list spec in skill or rule | Advancing while required tables unanswered or improvised |

**Rule:** Do not call `advance` until the current step's output contract is **fully delivered** and Aaron has confirmed (or the step is silent pull-only).

## Anatomy of an output contract

Every user-facing sub-step should define:

1. **Table ID** — e.g. `1.2-A`, `2.1-F`, `check`
2. **Fixed column headers** — copy verbatim from skill; never rename or reorder
3. **Data source per cell** — script output path, MCP query, or Aaron input
4. **Missing data** — literal `—` in the cell; optional footnote only if skill requires reason
5. **Deliverable scope** — exactly which table(s) belong to this ledger step (see skill "Present only" map)
6. **Decisions** — at most one lettered option table per turn (never AskQuestion); Aaron replies with letter(s)

### Weekly plan reference

`context/skills/weekly-planning/SKILL.md` — each Phase 1.x / 2.x block lists **Present:** tables with full markdown table definitions. That level of specificity is the target for all cadence workflows.

### Weekly ops reference

`context/skills/weekly-ops/SKILL.md` — same pattern; ledger steps `1.1`–`1.3`, then `2.1`, `2.4`, `6.2`, `7.1`, `2.2`–`2.3`, `4.1`–`4.2`, `3.1`–`3.2`, `6.1` map to named tables (aligned to Dev Projects sub-items under **Create, and Audit Weekly Ops Plan**). Phase 1.3 = daily ops records + ops priority; Phase 2 = Hiring; Phase 3 = Photographer Management; Phase 4 = Social Media; Phase 5 = 1:1; Phase 6 = CS; Phase 7 = Finance (4.2 = QuickBooks placeholder); Phase 8 = Sales; Phase 9 = Post Production; department Health/Priority confirmed in each domain phase (no separate health phase).

## Section completion checklist (before every `advance`)

Agent must verify internally; show FIELD CHECK tables explicitly at `N.check` steps:

- [ ] Phase banner matches ledger `current_step`
- [ ] Only in-scope tables for this step were presented (no extra sections)
- [ ] Every contract table has all rows/columns filled (`—` allowed where specified)
- [ ] Named data sources were used (not re-queried ad-hoc unless skill allows)
- [ ] Required Aaron input collected for this step
- [ ] Step-specific Notion log fields written (if this step commits data)
- [ ] If next step crosses a phase boundary → `gate` run and PASS before proceeding

**Hard stop:** If any box is unchecked, **stay on the current step**. Do not say "we'll come back to this" and advance.

## FIELD CHECK gates

FIELD CHECK is not a summary — it is a **verification table** listing required Notion properties or decision outcomes with Pass/Fail/N/A columns.

| When | Action |
|------|--------|
| End of Phase 1 (weekly plan) | Present Table `1.check`; all rows must Pass or documented N/A before `gate --phase 1` |
| End of Phase 2 dev block | Present Table `2.check`; same before Phase 4 commit |
| Weekly ops `check` | Present ops FIELD CHECK table from skill before `commit` |
| Session workflow `2.0` | Present commit checklist table before `complete` |

If FIELD CHECK fails: list failing rows, fix or get Aaron approval for N/A with reason, re-run check. **Do not open the next phase.**

## One deliverable per turn

| Deliverable type | Per turn limit |
|------------------|----------------|
| Contract table (full) | 1 table, unless skill explicitly says "present tables X and Y together" |
| Lettered decision table | 1 table; Aaron replies `A` or `A, C` |
| Ledger step | 1 step |
| Silent data pull (Phase 0) | May batch pulls; no user questions |

Within a single ledger step that spans multiple tables (e.g. weekly plan `1.2` = tables `1.2-A` through `1.2-G`): **one table per turn**, in skill order, until all tables for that step are done — then one confirmation turn if needed — then `advance`.

## Anti-patterns (never)

- Prose summary instead of the contract table
- "Here's an overview of your week" without the specified tables
- Column headers invented or renamed
- Metrics not listed in the skill
- Bundling PHQ-2 + intentions + energy in one message
- Advancing after Aaron answered a tangent but not the step's required question
- Skipping `gate` because "we're close enough"
- Using prior-week memory instead of named pull output files
- **`Workout Active Minutes` = N/A** when Workouts DB has review-week entries — use `daily-health-sections.mjs` aggregate (`Workout logged min`)
- **`Behavioral Adjustments` blank** when a domain rated **Unhealthy** in that step — Table 1.4-G (and equivalents) required **only for Unhealthy domains**; Healthy = no adjustments
- **Advancing step `1.5` without fuel Stage 1 (+ Stage 2 when applicable) and recovery intentions when triggered**

### Weekly plan — Phase 1 hard stops (before `1.check`)

| Step | Rule |
|------|------|
| `1.3-B` | `Workout Active Minutes` = Workouts DB `Minutes` sum (script aggregate). Fail FIELD CHECK if sessions exist and field blank. |
| `1.4` | If `Sleep Health` = Unhealthy → Table **1.4-G** required; if Healthy → skip (no adjustments) |
| `1.5` | Fuel **Stage 1 + 2** + **1.5-E-b recovery intentions** when triggered; all in `Social Review` |
| `1.check` | `Workout Active Minutes` Fail if Workouts exist and blank; `Behavioral Adjustments` only Fail for **Unhealthy** domains that contributed none |

## Session workflows — contract index

| Workflow | Contract location | Key tables |
|----------|-------------------|------------|
| person-checkin | `.cursor/rules/person-checkin.mdc` | 1.1-A workload, 1.1-B talking points, 1.1-C delegation |
| team-sync | `.cursor/rules/team-sync.mdc` | 1.1-A summary, 1.1-B drift list, 1.2 per-item decision |
| pd-cleanup | `context/skills/pd-cleanup/SKILL.md` | 0-A counts, per-item review, 5.check commit |

Monthly/quarterly: contracts in each `SKILL.md` **Present exactly** map + phase table blocks (`1.check` / `12.check` monthly; `A.check` / `E.4` quarterly).

**Dev tracker rule (weekly plan Phase 2):**

- **2.1-A (review):** Queued CL/TG dev work = Dev Projects `This Week` during the week under review — not a reconstructed guess from log text. Log `Dev Projects Intended` is cross-check only.
- **2.1-C (accomplished):** Re-present the **same tree as 2.1-A** with ~~strikethrough~~ on Done items — no flat shipped table. Logged completions = Done + `This Week` + `last_edited` in review window (`weekly-habits`); do not cite Done pages without `This Week`.
- **2.1-D (dev health):** Agent summarizes output, time, and month-goal progress; Aaron rates Healthy/Unhealthy. No agent recommendations.
- **2.1-E (adjustments):** **Only if Unhealthy.** Aaron states commitments; agent captures verbatim — never proposes adjustments.
- **2.1-F/G/H (plan):** F = carryover; G = add from planning month?; H = `🌙 Month` linked CL/TG (`weekly-dev-review`). Personal → 2.2 via same month relation.
- **2.1-S / 2.2-D sync:** After each selection, run `scripts/sync-dev-projects-this-week.mjs --selected=<IDs>` — selected `This Week = true`; **all other open** records `This Week = false`; clear stale Done checks. Cumulative IDs through 2.2.
- **2.check:** One bulleted tree (CL/TG + Personal) must **exactly match** Notion `This Week` filtered view before Phase 4.
- **4b.1:** `--list-candidates` → Aaron confirms Week Tracker page (plans finish on different weekdays).
- **4b.2:** `--week-page-id` → Google Doc in `Plan Records/weekly/` + `Plan Doc URL` on week record + `Weekly Plan` section on Notion page (`weekly-plan-week-summary.mjs`). Suggested title maps ledger `week_of` (Monday) → `Week Starting M/D` (Sunday). **Print layout:** seven domain tables — *What happened last week* (brief) · *Intentions for next week* (goals + adjustments merged; major adjustments bold first). **Mid-session:** after each life/dev domain, agent runs `weekly-plan-section-preview.mjs --section <slug>` (same layout) before advancing; optional `--all` before 4b.2 write.
- **2.2-A/C:** Planning month Personal Dev Projects (`🌙 Month`) — no free-text monthly priorities.
- **Monthly plan 11b:** Link up to 5 root Dev Projects + open sub-items to planning month (`scripts/sync-dev-projects-month.mjs --include-descendants`). Priority Stack text deprecated.

## Adding contracts to a new workflow

1. Define ledger steps in `workflow-registry.mjs`
2. For each user-facing step, add a **Present:** block with markdown tables (headers + sources)
3. Add `N.check` FIELD CHECK before major phase transitions
4. Add phase→Notion field map in `workflow-logs.md`
5. Reference this file from the skill/rule Execution Protocol section

## See also

- `context/workflow-execution.md` — turn-by-turn protocol
- `context/systems/workflow-logs.md` — Notion field contracts
- `.cursor/rules/workflow-execution.mdc` — Cursor enforcement