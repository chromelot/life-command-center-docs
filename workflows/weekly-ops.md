> **Source:** [`context/skills/weekly-ops/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-ops/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Weekly Ops — Chrome Lot Operations Review

This skill activates when Aaron says **"weekly ops"**, **"ops review"**, **"CL ops meeting"**, or **"operations meeting"**.

**Separate session** from Weekly Plan — typically a different day the same week.

**Target duration:** ~40 min · **Log:** Weekly Ops Meeting Log (dedicated Notion DB — not Weekly Meeting Log).

## Purpose

Current Chrome Lot operations: Pipedrive activity scorecard, CL currency, CS management, sales management, people/photographer review. Extracted from legacy weekly plan phases 2.5–2.8.

## Execution Protocol

1. **Date context:** `node scripts/planning-dates.mjs` — same planning week Monday as weekly plan.
2. **Init ledger:** `node scripts/workflow-progress.mjs init --workflow weekly-ops`
3. **Every turn:** `node scripts/workflow-progress.mjs status --workflow weekly-ops`
4. **Phase banner:** `**[Weekly Ops · Phase X — title]**`
5. **One sub-step per turn** · **One question per turn**
6. **Data pull (Phase 0):** `node scripts/weekly-ops-pull.mjs` (when implemented) or `scripts/weekly-data-pull.mjs` interim

## Phase map

| Phase | Content |
|-------|---------|
| `pre-0` | Confirm planning week; optional gate: Weekly Plan log exists for same `Week of {Monday}` |
| `0` | Ops data pull (Pipedrive, Knack, Hubstaff, Todoist overdue) |
| `1` | CL Operations — PD activity scorecard + CL currency check |
| `2` | CS Management — check-in cadence + late invoices |
| `3` | Sales Management — deal gaps, stale deals, team oversight |
| `4` | People Management — photographers + 1:1s + Hubstaff |
| `check` | Ops FIELD CHECK |
| `commit` | Weekly Ops Log entry + Team Activity Details + approved PD/Knack writes |

## Notion writes (Weekly Ops Log)

- Activity KPIs: `Aaron/Lexie/Tristen/Ran/Total Activities`
- `Ops Summary`, `Key Decisions`, `Action Items`
- `Active CL Sprint` snapshot, retire-a-slice commitment
- Team Activity Details (page blocks)

## Relationship to Weekly Plan

| Weekly Plan | Weekly Ops |
|-------------|------------|
| Life review + dev planning | CL ops execution |
| Weekly Meeting Log DB | Weekly Ops Log DB (separate) |
| `Ops Minutes` habit KPI (retrospective time) | PD activity scorecard (forward planning) |

Monthly plan reads **Weekly Ops Log** for team activity rollups (not Weekly Meeting Log).

## See also

- `../weekly-planning/SKILL.md` — life + dev (separate session)
- `../../work/chrome-lot/customer-service.md`, `../../work/chrome-lot/sales.md`, `../../work/chrome-lot/operations.md`
- `../../systems/pipedrive.md`, `../../systems/knack-fields.md`, `../../people/index.md`