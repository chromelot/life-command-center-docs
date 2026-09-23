> **Source:** [`context/skills/weekly-planning/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-planning/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

﻿---
updated: 2026-09-15
status: active
tags: [skill, weekly-planning, procedure]
---

# Weekly Planning — SKILL

## Table of contents

- [Trigger](#trigger)
- [Inputs](#inputs)
- [Execution Protocol (mandatory — read `context/workflow-execution.md` + `context/systems/workflow-output-contracts.md`)](#execution-protocol-mandatory-read-contextworkflow-executionmd-contextsystemsworkflow-output-contractsmd)
- [Interaction Style](#interaction-style)
- [Required Notion fields — index](#required-notion-fields-index)
- [Procedure](#procedure)
- [Pre-Phase 0: Monthly Plan Gate (mandatory — runs before everything else)](#pre-phase-0-monthly-plan-gate-mandatory-runs-before-everything-else)
- [Phase 0a: Confirm Review + Planning Weeks (~1 min)](#phase-0a-confirm-review-planning-weeks-1-min)
- [Phase 0: Data Pull (silent, before conversation)](#phase-0-data-pull-silent-before-conversation)
  - [Wellness pulls (run first)](#wellness-pulls-run-first)
  - [Domain register pull (canonical board for `1.R` + `3.R`)](#domain-register-pull-canonical-board-for-1r-3r)
  - [Social pulls (with wellness; used in Phase 1.5)](#social-pulls-with-wellness-used-in-phase-15)
  - [Development pulls (after wellness/social; silent until Phase 2+)](#development-pulls-after-wellnesssocial-silent-until-phase-2)
- [Phase 0b: Data Integrity Gate (~3 min)](#phase-0b-data-integrity-gate-3-min)
- [Phase 1: Life Review (~28 min)](#phase-1-life-review-28-min)
  - [1.0 Create Weekly Log Entry](#10-create-weekly-log-entry)
  - [1.1 Values Context (~1 min) — **first table to Aaron**](#11-values-context-1-min-first-table-to-aaron)
  - [1.2 Mind — Review · Mood · Rate · Intentions (~6 min)](#12-mind-review-mood-rate-intentions-6-min)
  - [1.3 Fitness — Review · Rate · Intentions (~5 min)](#13-fitness-review-rate-intentions-5-min)
  - [1.3b Health & Care — Review · Rate · Intentions (~2 min)](#13b-health-and-care-review-rate-intentions-2-min)
  - [1.4 Sleep and Schedule — Review · Rate · Intentions (~5 min)](#14-sleep-and-schedule-review-rate-intentions-5-min)
  - [1.5 Social — Review · Rate · Intentions (~5 min)](#15-social-review-rate-intentions-5-min)
  - [1.6 Parenting — Review · Rate · Intentions (~4 min)](#16-parenting-review-rate-intentions-4-min)
  - [1.7 Personal Enjoyment (~2 min)](#17-personal-enjoyment-2-min)
  - [1.8 Money & Admin — Review · Rate · Intentions (~2 min)](#18-money-and-admin-review-rate-intentions-2-min)
  - [`1.R` — Personal Repair & Debt *(REQUIRED — closes Phase 1)*](#1r-personal-repair-and-debt-required-closes-phase-1)
- [Phase 2: Development (domain-first, ~18 min)](#phase-2-development-domain-first-18-min)
  - [Per-domain loop — run `2.TG` → `2.CL` → `2.SY`](#per-domain-loop-run-2tg-2cl-2sy)
  - [`2.H` — Dev health review (one turn, after all three domains)](#2h-dev-health-review-one-turn-after-all-three-domains)
  - [`2.WA` — Workshop + Admin (lighter tail)](#2wa-workshop-admin-lighter-tail)
  - [`2.sync` — Commit the combined slate (run once)](#2sync-commit-the-combined-slate-run-once)
- [Phase 3: Operations (~10 min)](#phase-3-operations-10-min)
  - [`3.R` — Work Repair & Debt *(REQUIRED — closes Phase 3)*](#3r-work-repair-and-debt-required-closes-phase-3)
- [Phase 4: Commit (~5 min)](#phase-4-commit-5-min)
- [Cross-Cutting Rules](#cross-cutting-rules)
- [Outputs](#outputs)
- [Failure modes & graceful degradation](#failure-modes-and-graceful-degradation)
- [See also](#see-also)

---


> **🚫 STORAGE IS ZPT/D1 — NOT NOTION.** Planning now runs in the **Zero Pit Stop app** (in-app weekly wizard); the system of record is **Cloudflare D1 `meeting_logs`** (see `context/systems/weekly-plan-app.md` + `.cursor/rules/zpt-d1-sor.mdc`). **Every Notion write-step in this SKILL is RETIRED — do not execute it:** no Weekly Meeting Log DB writes, no `personal_notion_*` calls, no Week Tracker page append, no `weekly-plan-week-summary.mjs` PDF/Plan-Doc upload, no Values-DB Notion updates. This file survives **only** as the reference for the *review/data-pull logic and per-domain question flow*. When you run the plan, capture results in the ZPT app (it dual-writes to D1); the Cursor dual-write scripts key the same `wlog_<planning_period_id>` D1 record. **If you're about to touch Notion, stop — you're following stale instructions.**

<a id="trigger"></a>
## Trigger

This skill activates when Aaron says "weekly plan", "weekly meeting", "plan this week", "sprint planning", or "Monday review". Target duration: ~60 minutes (Phase 1 life ~30 min incl. the personal repair · Phase 2 dev domain-first ~18 min · Phase 3 operations + work repair ~10 min · Phase 4 commit ~5 min). **CL operations** are stewarded in this session via domain ratings and **`work_repair_domain`** — not a separate weekly-ops session. Department reviews live at `#/departments` in ZPT when needed.

<a id="inputs"></a>
## Inputs

Load via the router. Read these before starting:

- `context/systems/cadences.md` — cadence health check, Phase 0 schedule
- `context/systems/capacity-rules.md` — limits, overcommitment triggers, intervention protocol
- `context/systems/notion-databases.md` — every DB ID referenced below (Tasks, Weekly Meeting Log, source DBs, Values, Workouts, etc.)
- `context/systems/knack-fields.md` — Customer + Photographer field references
- `context/systems/hubstaff.md` — member IDs, weekly-report tool
- `context/systems/health-data.md` — MCP architecture, expected fields
- `context/self/values.md` — six categories and current Health statuses (Phase 1.1 context; per-domain ratings in Phase 1 + 2.H/2.WA-H)
- `context/self/eros.md` — primary fuel doctrine (Phase 1.5 fuel check)
- `context/self/social.md` — sarges, Small Talk targets, isolation signals (Phase 1.5)
- **Dev state (canonical):** **Tasks** DB (`341f40c2-487b-80ac`) — Phase 2 reads the tracker first; meeting logs record what was planned and accomplished. `This Week` is set only from Aaron's explicit selection each session.
- `context/people/index.md` — delegation matrix, 1:1 tracking
- `context/work/turbo-gear/overview.md` — TG strategic sequence (for Phase 2.4 project selection)
- `context/systems/time-blocks.md` + `config/time-blocks.json` — weekly schedule template, color legend, step `4.tb` (Personal Time Blocks calendar)

<a id="execution-protocol-mandatory-read-contextworkflow-executionmd-contextsystemsworkflow-output-contractsmd"></a>
## Execution Protocol (mandatory — read `context/workflow-execution.md` + `context/systems/workflow-output-contracts.md`)

> **This skill's tables are the spec.** Fixed headers, named sources, one table per turn where noted. Weekly plan is the reference implementation for all workflows.

1. **Date context (every turn, before any weekday or "this week" language):**
   ```
   node scripts/planning-dates.mjs --ledger <path> [--today=YYYY-MM-DD]
   ```
   Aaron may run weekly plan on **any day** — never assume session day = Monday. Use script output for **review week** (look back), **planning week** (look forward), and weekday→ISO mapping. `Today's date` from `user_info` must match `--today` (CT).
2. **Init ledger** at session start: `node scripts/workflow-progress.mjs init --workflow weekly-plan` — sets default Review + Planning Week Tracker rows and `week_of` = **planning Monday** (not review Monday). Weekly log title = `Week of <planning Monday>`.

> **Canonical week boundary — do not reintroduce Monday windows.** The **data/aggregation week is Sunday → Saturday, America/Chicago** (Week Tracker `Week Start` = the Sunday; `Is Current Week` uses Sun=0; Time Punches "this week"; habit scorecard; dev/ops/nutrition rollups; the dashboard). The **`week_of` Monday is a label only** — the Weekly Meeting Log title `Week of <Monday>` is that Sunday + 1 for human readability; it is **not** a second week window. Any weekly *data* aggregation must use `sundayOfWeekContaining` (`scripts/lib/ct-weekday.mjs`) / the Week Tracker range — **never** `mondayOfWeekContaining` (that helper is only for the label + Mon–Fri calendar-event scheduling).
3. **Step `0a` — confirm weeks (before any Phase 0 pull):** Run `node scripts/weekly-plan-weeks.mjs --ledger <path>`. Present the table verbatim. Aaron confirms (letter **A** = defaults, or override IDs). Then `node scripts/weekly-plan-weeks.mjs --ledger <path> --confirm` (or `--review-week-id` / `--planning-week-id` if overriding). Advance `0a` only after `--confirm`.
4. **Notion log** — at step `1.0`: `node scripts/workflow-notion-log.mjs create --ledger <path>` (writes **Review Week** + **Planning Week** relations from ledger). After every `advance`: `workflow-notion-log sync`. Write phase fields per `context/systems/workflow-logs.md`. On commit: `workflow-notion-log complete`.
5. **Every turn:** `node scripts/workflow-progress.mjs status --workflow weekly-plan` — present only `current_step`
6. **Phase banner** on every user-facing message: `**[Weekly Plan · Phase X.Y — title]**`
7. **One sub-step per turn** — never bundle 1.2 + 1.3 + 1.4 + 1.5 + 1.6 + 1.7 (each life domain is a separate step)
8. **Table contract is the spec** — each sub-step lists the **exact tables** to present (column headers fixed). Fill every cell from the named data source; use `—` when data is missing. Do not add metrics, sections, or discussion topics outside that step's tables. **Do not `advance` until every in-scope table for the step is presented and any required Aaron input is collected.**
9. **One question per turn** — Energy rating and Mind intentions are separate turns; PHQ-2/GAD-2 only when `Screening Escalation` is true (one item per turn)
10. **Advance after complete:** `node scripts/workflow-progress.mjs advance --workflow weekly-plan --step <id>`
11. **Print preview (required before each domain advance):** After Aaron approves intentions and **Notion fields for that domain are synced**, run:
    ```
    node scripts/weekly-plan-section-preview.mjs --ledger <path> --section <slug>
    ```
    Present the script output **verbatim** (same 3-row table as Phase 4b Google Doc). Aaron confirms nothing surprising → then `advance`. If wrong, fix Notion and re-run preview.

    | After step | `--section` slug | Notes |
    |------------|------------------|-------|
    | `1.2` | `spirituality` | After death-chart gate confirm + mind Notion sync |
    | `1.3` | `fitness` | |
    | `1.4` | `sleep-schedule` | Sleep + schedule intentions only (no cross-domain pull) |
    | `1.5` | `social` | After fuel check + social Notion sync |
    | `1.6` | `parenting` | |
    | `1.7` | `enjoyment` | |
    | `1.R` (after the personal repair + debt picks) | — | Board only; no section preview |
    | `2.WA` (after Workshop/Admin) | `development` | Includes CL/TG/Systems + Workshop/Admin dev tree when on log |
    | `3.R` (after the work repair + debt picks) | — | Board only; no section preview |
    | `2.sync` (after the combined slate sweep + **Table 2.S** confirmed) | `development` | Includes debt picks from `1.R.3` **and** `3.R.3` |

    `1.3b` (Health & Care) and `1.8` (Money & Admin) have **no print-section slug** — they rate to the log and render in the week summary, but `weekly-plan-section-preview.mjs` has no section for them. Skip the preview on those two steps.

    Optional before **4b** write: `--all` for full seven-domain preview.
12. **Phase gates:** `node scripts/workflow-progress.mjs gate --workflow weekly-plan --phase <1|2|3>` before Phase 2 (development), Phase 3 (operations), or `4.tb`. Gate **1** requires `1.R` (prior-repair retro + personal repair with intent + debt); gate **3** requires `3.R` (prior-repair retro + work repair with intent + debt).
13. **Tangents:** fix/interrupt, then resume ledger `current_step` — do not skip ahead

<a id="interaction-style"></a>
## Interaction Style

- **One question at a time.** Never present a wall of choices. Walk through decisions sequentially.
- **Confirm before executing.** Each phase presents proposed actions, gets approval, then executes before moving on.
- **Exclude Shopping List** (Todoist project `6W36wRPXj8qC2RCc`) from all analysis.
- **Data integrity:** All Knack/Notion/Todoist write operations require explicit user approval before execution (Todoist case-by-case - never batch or assume). Pipedrive writes → **Weekly Ops** only.

<a id="required-notion-fields-index"></a>
## Required Notion fields — index

Each phase ends with an inline **FIELD CHECK** listing its required Weekly Meeting Log properties. Phase 4 (Commit) verifies all sections.

**Dev tracker hygiene (every Phase 2 session):**
1. **Tracker first** — monthly incomplete lists come from Tasks (Notion), not log prose.
2. **Selection accumulates, sync once** — each domain's `.3` (and `2.WA`) records selected Task IDs into `notes.dev_slate_ids`; run `scripts/sync-dev-projects-this-week.mjs` **once at `2.sync`** with the full cumulative set: set `📅 Week Tracker` = planning week on selected only; **clear it on every other open Task** (full-DB sweep). That script also **provisions Toggl 2 tasks for the slate and deletes mirrored tasks** for anything no longer on the slate (keeps Focus uncluttered). Present the finalized bulleted slate (Table 2.S); it must match the Notion `This Week` filter exactly. **Never** run the sync mid-loop with a single domain's IDs.
3. **Missing records** — work Aaron describes that is not in Tasks → create a record (with approval) before toggling `This Week`.
4. **Workshop/Admin mirrors** — Phase 2.WA creates Todoist mirrors for selected Workshop/Admin items (case-by-case approval); verify last week's mirrors via Todoist MCP.

**Gate rules:**
- Before **Phase 2 (Work)**: Phase 1 FIELD CHECK (`1.check`) must pass — which requires the personal repair pick at `1.R`.
- Before **Phase 3 (Operations)**: Phase 2 development block complete (through `2.check`).
- Before **Phase 4 (Commit)**: Phase 3 complete (through `3.R`) — the work repair pick is set or explicitly declined.
- Each phase delivers **only** its table contract (see per-phase **Present** blocks below).

<a id="procedure"></a>
## Procedure

<a id="pre-phase-0-monthly-plan-gate-mandatory-runs-before-everything-else"></a>
## Pre-Phase 0: Monthly Plan Gate (mandatory — runs before everything else)

The weekly plan assumes a committed monthly frame. Do not start Phase 0 until this gate passes.

1. Compute **review month** and **planning month** from today's date (America/Chicago) — same framing as `context/skills/monthly-plan/SKILL.md` (Month framing section). Example: session on 2026-06-05 → review month = May 2026, planning month = June 2026.
2. Query **Monthly Plan Log DB** (`344f40c2-487b-806d`) for an entry whose Name matches `Monthly Plan for [Planning Month YYYY]` OR whose **Month** relation points to **planning month**.
3. **If no matching entry exists** OR **`Session Complete` ≠ Complete** (shell-only / in-progress monthly session does not count):
   - Tell Aaron: "Monthly plan for [planning month] is not committed. Finish monthly plan Phase 12 before weekly plan."
   - **Pause** this workflow. Run `context/skills/monthly-plan/SKILL.md` end-to-end (or resume an in-progress ledger).
   - After monthly plan Phase 12 commits (`Session Complete` = Complete), **resume** weekly plan from Phase 0 below.
   - Do **not** offer to skip or proceed weekly-only — a **completed** monthly plan is a hard prerequisite.
4. **If entry exists and Session Complete = Complete:** Hold for Phase 2 dev review. Continue to **Phase 0a** (confirm weeks), then Phase 0.

<a id="phase-0a-confirm-review-planning-weeks-1-min"></a>
## Phase 0a: Confirm Review + Planning Weeks (~1 min)

**Purpose:** Lock which Week Tracker rows this session **reviews** vs **plans for** — before any data pull. Aaron's default workflow (Fri–Mon):

| Session day | Review week | Planning week | Turnover state |
|-------------|-------------|---------------|----------------|
| **Friday / Saturday** | Current calendar week (still in) | **Next** Week Tracker row | **Pre-turnover** — review week is still `Is Current Week` |
| **Sunday / Monday** | Prior Week Tracker row | Current week (entering) | **Post-turnover** — auto-defer already ran |

**Turnover = the `WeekDefer` automation (Sun ~3 AM CT).** At the week flip it moves every open Task still linked to the ending week onto the new current week (`scripts/defer-week-tasks.mjs`; Done tasks stay put as history). This drives the pre/post-turnover branch in each dev domain's `.3` carryover (Table 2.{D}-Carry):

- **Pre-turnover (Fri/Sat)** — the review week is still current. Auto-defer has **not** run yet, so at carryover ask Aaron which open review-week Tasks he **might still finish** (leave linked to the current/review week — WeekDefer will sweep any that stay open Sunday) vs **defer now** (link to the planning/next week). Tasks legitimately live on two weeks during this window; that's expected.
- **Post-turnover (Sun/Mon)** — WeekDefer already moved every unfinished Task to the current (= planning) week. **Do not** ask "what are you still trying to finish" — treat the carried slate as already deferred; only prune and add.

Detect the state from Table 0a: if the **Review** row's `Is Current Week` is still true → pre-turnover; if it's a past row → post-turnover.

1. Run `node scripts/weekly-plan-weeks.mjs --ledger <path>` — present output **verbatim**.
2. **Present exactly Table 0a:**

```
TABLE 0a — Week pair confirmation
| | Week Tracker | Dates | page_id |
|---|--------------|-------|---------|
| Review (look back) | … | Sun–Sat | … |
| Planning (look forward) | … | Sun–Sat | … |
```

3. Aaron confirms: **A** = accept defaults · **B** = override (agent re-runs with `--review-week-id` / `--planning-week-id`, then confirm).
4. Run `node scripts/weekly-plan-weeks.mjs --ledger <path> --confirm` (sets ledger `weeks_confirmed`, `week_of`, and Meeting Log relations when log exists).
5. `advance --step 0a` — **blocked** until `--confirm` has run.

All Phase 0+ pulls use `--ledger <path>` so habits, dev review, dev slate sync, and 4b write target the confirmed weeks.

<a id="phase-0-data-pull-silent-before-conversation"></a>
## Phase 0: Data Pull (silent, before conversation)

**Prerequisite:** Step `0a` complete (`weeks_confirmed = true`).

**Order:** wellness + log trends first, then work pulls. Mind/body phases run before any work discussion.

<a id="wellness-pulls-run-first"></a>
### Wellness pulls (run first)

```
node "scripts/weekly-wellness-trends.mjs"
```
Output: `output/weekly-wellness-trends-YYYY-MM-DD.md`. **Canonical source for 4-week Weekly Meeting Log trends + prior-week intentions + data-integrity report.** Read this before Phase 0b and Phase 1.

```
node "scripts/weekly-habit-summary.mjs" --ledger <path>
```
Output: `output/weekly-habits-YYYY-MM-DD.md`. Canonical last-week habit numbers + actionable Tasks slate. Includes **work close-out** (days closed + avg sign-off CT from Day Tracker). Do not re-query habit DBs ad-hoc.

```
node "scripts/weekly-journal-feelings.mjs"
```
Output: `output/weekly-journal-feelings-YYYY-MM-DD.md`. **Canonical source for Phase 1.2 mood tables** (valence, mood score frequency, daily mood, escalation, mind insights) **and TABLE 1.2-D-journal** (morning journal gratitude/goals insights). Do not re-query Journal DB ad-hoc.

```
node "scripts/daily-health-sections.mjs"
```
Output: stdout **Mind / Fitness / Sleep** section tables for Phase 1.2–1.4 (days as columns). Run after health pulls.

```
node "scripts/withings-sync.mjs" --days 28 --write
```
> **Watch metrics are now live via HC Webhook → n8n** (steps, sleep+stages, HR, SpO₂ upsert to the Health Data DB continuously). **Do not run `health_persist_recent`** — it's retired along with the Health Sync→Drive→CSV→MCP writer (2026-07-12) and would fight the HC data. Only `withings-sync` (body comp) still writes in Phase 0. If watch data looks stale, that's an HC pipeline issue — check the **Health Sync Watchdog** / HC Webhook app on the phone, don't re-persist.
```
health_get_summary({ days: 28 })
```
Use `days: 28` for 4-week trend context. Returns `stats`, `trend` (last-7 vs prior-7 when days≥14), `daily` (includes `wake_time`, `wake_minutes`, `bedtime_time` per night when Health Sync sleep CSV available). Call `health_source_status` if sources look stale.

Also query **Weekly Meeting Log** (`322f40c2-487b-81bd`) — last **4 entries** sorted by Meeting Date descending (script above summarizes; keep raw entries for Phase 1).

<a id="domain-register-pull-canonical-board-for-1r-3r"></a>
### Domain register pull (canonical board for `1.R` + `3.R`)

```
node scripts/weekly-domain-board.mjs --ledger <path>
```
Output: `output/weekly-domain-board-YYYY-MM-DD.md` (Sunday of the review week). **Canonical source for both repair boards** — every `departments` register row with Area, Domain, Held by, Floor, Target, this week's actual (from the row's `evidence` key), proposed status, current status, and `in_repair`. Read this file at `1.R` and `3.R`; do not re-query the register ad-hoc.

> **Proposed statuses are suppressed while the week is still in progress.** The script counts elapsed days and, for rate-based floors, prints `—` instead of a proposal — three lifts on day three is on-target pace, not a third of the way to failing. Pre-turnover (Fri/Sat) sessions therefore read the **actuals as pace** and rate the prior complete week; only a complete week's proposals are comparable to a weekly floor.

<a id="social-pulls-with-wellness-used-in-phase-15"></a>
### Social pulls (with wellness; used in Phase 1.5)

```
node "scripts/social-phase-pull.mjs"
```
Output: stdout **Small Talk list, days since last entry, daily counts, calendar social events** for the review week. Canonical source for Phase 1.5 Tables 1.5-B/C. Also run after logging a new Small Talk entry mid-session.

- **Small Talk DB** (`121f40c2-487b-802d`): script queries all entries; uses `Created Date` when set.
- **Google Calendar (last 7 days):** script pulls **Personal calendar only** (`hoegenauera@gmail.com` — actual booked events). **Exclude** Personal Time Blocks (`10283d615…@group.calendar.google.com`) — those are time-spending goals, not real events. Flags social-looking events (hangouts, Meetup, dates, fitness classes, etc.). Count → `Social Events Count`.

<a id="development-pulls-after-wellnesssocial-silent-until-phase-2"></a>
### Development pulls (after wellness/social; silent until Phase 2+)

```
node scripts/weekly-dev-review.mjs --ledger <path>
```
Output: `output/weekly-dev-review-YYYY-MM-DD.md` — **canonical Phase 2 source** (prior week plan, dev time by day, monthly incomplete by domain, personal carryover).

Also run `weekly-habit-summary.mjs` (logged/unlogged accomplishments). Todoist MCP in Phase 2.WA for Workshop/Admin mirror completion check. Ops context → ZPT **Departments** `#/departments` for review history (not weekly-ops-pull).
4. **Habit source DBs** (past 7 days — habit summary script is canonical; MCP only if script missing):
   - Workouts: **D1** `workouts` domain (via `weekly-habit-summary.mjs` — not Notion)
   - Small Talk (`121f40c2-487b-802d`): query all, count entries
   - Spirit — **Time Punches** `Category = Spirit` (or `weekly-habit-summary` § Spirit; retired Spirit DB is historical only)
   - **Time Punches** (`394f40c2-487b-8168`): query by **Category** — CL Dev, TG Dev, Ops, Field, Admin, Reading — sum **Total Time** (formula: `properties["Total Time"].formula.number`)
   - Journal (`99c9e393-812f-4d73`): query all, count entries
   - *Legacy per-bucket DBs (Business Development, CL Ops, Field Work, Admin, Reading) are retired — do not query.*
5. **Values context** — `node scripts/weekly-values-context.mjs --ledger <path>` → `output/weekly-values-context-YYYY-MM-DD.md`. Merges prior **closed** week Weekly Meeting Log health selects with Values DB time targets. Excludes current session log via `--ledger`.
6. **Google Calendar**: Next week's events (dev capacity in Phase 2.4)
7. **Hubstaff**: Last week's hours via `hubstaff_get_weekly_report` (dev capacity in Phase 2.4)
8. **Health Data**: `health_get_summary({ days: 28 })` above — used throughout Phase 1.

**Reconcile completions first** — before Phase 2, check what's already been done since last plan and mark complete.

<a id="phase-0b-data-integrity-gate-3-min"></a>
## Phase 0b: Data Integrity Gate (~3 min)

**Purpose:** Surface missing/stale data and fix upstream systems before mind/body review. **Do not skip to Phase 1 with silent gaps.**

1. Read `output/weekly-wellness-trends-YYYY-MM-DD.md` **Data integrity** section.
2. Call `health_source_status` if Withings or Health Sync flags are stale.
3. Present a short **DATA INTEGRITY** table:

```
DATA INTEGRITY CHECK
| Source              | Status   | Action taken / needed        |
|---------------------|----------|------------------------------|
| Weekly Log (4 wk)   |          | missing KPI fields?          |
| Withings / body comp|          | re-run sync?                 |
| Health Sync / sleep |          | wake times available?        |
| Habit summary script|          | ran OK?                      |
```

4. **Attempt fixes before Phase 1** (run yourself, don't hand off):
   - Missing body comp → `withings-sync.mjs --days 28 --write`
   - Missing watch/sleep → HC pipeline issue (not a persist step anymore): check the **Health Sync Watchdog** DM / HC Webhook app on the phone (network, last sync). Data lands live; re-read with `health_get_summary({ days: 28 })`.
   - Missing prior-week log KPIs → backfill from `weekly-habits-*.md` + health MCP into the **prior week's** log entry (with Aaron approval for Notion writes)
5. If a source remains broken after one fix attempt, note it in the table and continue with `--` for affected metrics — but **name the broken pipeline** so it gets fixed outside the meeting.
6. **Hard gate — prior week `Week Intentions`:** Before advancing `0b`, `workflow-progress.mjs` runs `checkPriorWeekIntegrity()`. If the most recent Weekly Meeting Log is missing `Week Intentions` or core KPI numbers, advance is **refused** (exit 4). *(Domain `*Intentions` are qualitative-only and may be blank — no longer gated.)* Remediate via `node scripts/backfill-week-intentions.mjs --meeting-date YYYY-MM-DD --text "..."` (Aaron approval) or re-run prior session Phase 4. Override only with `advance --step 0b --force`.

**Outputs:** Integrity table presented; remediation attempted; known gaps flagged for Phase 1 footers.

<a id="phase-1-life-review-28-min"></a>
## Phase 1: Life Review (~28 min)

**Purpose:** Values context first, then mind → fitness → health & care → sleep → social → parenting → personal enjoyment → money & admin — one domain at a time (review → rate health where applicable → set intentions) — then **one personal repair** at `1.R`. Mind includes wellness screening. Work health rates in Phase 2.H; the **work** repair runs at `3.R`.

> **Intentions vs targets — keep them separate (do not restate numbers).** Every recurring **numeric goal** lives in a **structured target field** that drives the dashboard widgets: `Strength Target`, `Cardio Target` (fitness) · `Sleep Target Hours`, `Target Wake Time` (sleep) · `Calorie Target`, `Weight Goal Direction` (nutrition) · `Social Target` (social small-talk/sarges) · `Workshop Hours Intended` (workshop). The `*Intentions` rich_text fields (`Mind/Fitness/Sleep/Social/Parenting Intentions`, `Week Intentions`) capture **only qualitative changes** for the week — behavioral shifts, experiments, focus themes, one-off adjustments. **Never** write "5 strength workouts" or "3–5 sarges" into an intention field; that number goes in its target field. If a domain has no qualitative change this week, **leave its intention blank** (better empty than restating a target). This keeps the weekly printout + dashboard clean and non-redundant.

**Phase 1 order (session):** `1.0` → `1.1` Values → `1.2` Mind (incl. wellness) → `1.3` Fitness → `1.3b` Health & Care → `1.4` Sleep and Schedule → `1.5` Social → `1.6` Parenting → `1.7` Personal enjoyment → `1.8` Money & Admin → `1.R` Personal — repair & debt → `1.check`

**Print / Week Tracker domain order (Phase 4b):** Sleep and Schedule → Spirituality & Mind → Fitness → Social → Parenting → Personal Enjoyment → Development Work. Each domain renders as **three parts**: *What happened last week* · **Targets for next week** (the structured numbers — Strength/Cardio, Sleep hours + wake time, Calories + direction, Social target, Workshop hours — rendered as a compact line/badges) · *Intentions for next week* (**qualitative only** — behavioral changes / experiments; **omit the line entirely when blank**, don't pad with restated targets). Four-value status badge when rated (**Below floor** / **At floor** / **Healthy** / **Not assessed**); **trend arrow** (↑ / ↓ / −) vs the **prior week's** same-domain rating (numeric score 1–3; `null` when not assessed). This keeps the printout's intentions section clean and meaningful (targets are shown once, as numbers, not repeated as prose).

**Domain status scale** *(all Phase 1 domain ratings + Phase 2.H dev health — one status per turn)*

| Status | Meaning |
|--------|---------|
| **Below floor** | Missed the written floor — clearable on a bad day, but missed this review week |
| **At floor** | Running without excelling. **A pass.** |
| **Healthy** | Hitting target |
| **Not assessed** | New domain or not reviewed this week |

**Rate sub-step shape (every rated domain):** review data → show **Floor · Target · this week's actual** (from the domain register + session pulls) → Aaron picks status → intentions. The status is a lookup against written bars, not a mood letter.

**Gates:** Behavioral adjustments (`1.4-G`, `2.H-adj`, `1.check`) trigger when status = **Below floor**. At floor and above: no behavioral-adjustment table for that domain.

<a id="10-create-weekly-log-entry"></a>
### 1.0 Create Weekly Log Entry

Create the **new week's** Weekly Meeting Log entry (Name = `Week of [next Monday YYYY-MM-DD]`, Meeting Date = today). All Phase 1 fields write to this entry.

**Advance ledger:** `1.0` → then present `1.1` only.

<a id="11-values-context-1-min-first-table-to-aaron"></a>
### 1.1 Values Context (~1 min) — **first table to Aaron**

**Purpose:** Orient to the six Values categories before domain deep-dives. Health ratings and intentions are set per-domain in later steps — this step is context only.

**Data source:** `output/weekly-values-context-YYYY-MM-DD.md` (from `node scripts/weekly-values-context.mjs --ledger <path>` in Phase 0 or at step 1.1). Pass `--ledger` so the in-progress session log is excluded; **Effective Health** = prior **closed** week log rating when set; otherwise Values DB fallback.

**Present exactly Table 1.1:**

| Category | Prior Week Health | Values DB Health | **Effective Health** | Source | Time Target | 1-line note |
|----------|-------------------|------------------|----------------------|--------|-------------|-------------|
| Spirituality | from prior log `Spirituality Health` or `Mind Health` | from Values DB | prior week if set, else Values DB | prior week log / Values DB | from Values DB | |
| Fitness | `Fitness Health` | from Values DB | same rule | | from Values DB | |
| Work | `Work Health` | from Values DB | same rule | | from Values DB | |
| Social | `Social Health` | from Values DB | same rule | | from Values DB | |
| Admin | `Admin Health` (often blank — weekly plan does not rate Admin) | from Values DB | same rule | | from Values DB | |
| Parenting | `Parenting Health` | from Values DB | same rule | | from Values DB | |
| Personal Enjoyment | `Enjoyment Health` | from Values DB | same rule | | from Values DB | |

**Table 1.1-G — Life goal KPIs (REQUIRED every week)**

Phase 2 reviews goal progress for the **dev** domains (Turbo Gear / Chrome Lot / Systems) in Table
`2.{D}-Goals`. This is the same check for the **life** domains — **Personal, Admin, Workshop** — which
otherwise have no weekly review and only surface at the monthly/quarterly layer. Since these goals
carry **weekly** KPIs, a monthly-only check finds out about four missed weeks after the fact.

Source: each goal's `habit_targets` (D1 `goal_inspiration`, `gi_<goalId>`) — `metricKey`, `target`,
`period`. Actuals come from the numbers already pulled this session (e.g. `life.spirit_min` = Spirit
Minutes in Table 1.2-B); **do not re-query.**

| Goal | Value | KPI | Target | Last week actual | On track? |
|------|-------|-----|--------|------------------|-----------|
| one row per active Personal / Admin / Workshop goal with a `habit_targets` entry | linked `value_id` | `name` (`metricKey`) | `target` `unit`/`period` | from this session's pulls | ✓ / ~ / ✗ |

Goals with no `habit_targets` entry are listed with `—` in the KPI columns; flag any that have gone
**two consecutive quarters without a measurable target** for the next quarterly plan (Phase D sets
targets; this step only reads them).

**Agent does not recommend here** — narrative only. Any behavioral response belongs in the owning
domain's intentions step (1.2 Mind for Spirituality, 1.5 Social, 1.3 Fitness).

<a id="12-mind-review-mood-rate-intentions-6-min"></a>
### 1.2 Mind — Review · Mood · Rate · Intentions (~6 min)

**Data sources (read in order, do not re-query):**
1. `output/weekly-journal-feelings-YYYY-MM-DD.md` — **mood tables (canonical)**
2. `output/weekly-habits-YYYY-MM-DD.md` — spirit/journal counts
3. `output/weekly-wellness-trends-YYYY-MM-DD.md` — 4-wk mood trend context
4. `node scripts/daily-health-sections.mjs` — MIND daily section (stdout)
5. Prior week's `Mind Intentions` from wellness trends file

**Sub-step order within 1.2:** `1.2-A` → `1.2-B` → `1.2-B-mj` → `1.2-C` → `1.2-C-mj` → `1.2-C-mood` → `1.2-D` → `1.2-D-journal` → `1.2-E` → `1.2-E-b` → `1.2-F` → (optional `1.2-F-screen` if escalation) → `1.2-G` → `1.2-H`

Copy tables **verbatim** from the feelings file and habit/health scripts — fill every cell; use `—` only when the source file shows `—`.

**Table 1.2-A — Prior mind intention**

| Last week's `Mind Intentions` | Evidence | Met? |
|-----------------------------|----------|------|
| from wellness trends | 1-line summary | ✓ / ~ / ✗ |

**Table 1.2-B — Mind aggregate**

| Metric | Last Week | 4-wk trend | → Notion field |
|--------|-----------|------------|----------------|
| Spirit Minutes | from `weekly-habits-*.md` TABLE 1.2-B-spirit | from wellness file | `Spirit Minutes` |
| Meditation Minutes | from `weekly-habits-*.md` TABLE 1.2-B-spirit | — | (in summary) |
| **Reading Minutes** | from `weekly-habits-*.md` TABLE 1.2-B-spirit (`Category = Reading` time punches) | — | (in summary) |
| Journal Count | from `weekly-habits-*.md` | from wellness file | `Journal Count` |
| Morning Journal days | from `weekly-habits-*.md` TABLE 1.2-B-mj (Journal DB entry exists) | — | (in summary) |
| Affirmations | from `weekly-habits-*.md` TABLE 1.2-B-mj (`Affirmations done` to_do in journal page — separate from journal completion) | — | (in summary) |
| Quick Reading | from `weekly-habits-*.md` TABLE 1.2-B-mj (`Quick reading session done` to_do in journal page — separate from journal completion) | — | (in summary) |

**Table 1.2-B-spirit — Spirit practice aggregate** *(copy from `weekly-habits-*.md` section `TABLE 1.2-B-spirit`)*

**Table 1.2-B-mj — Morning journal aggregate** *(copy from `weekly-habits-*.md` section `TABLE 1.2-B-mj`)*

**Table 1.2-C-mj — Morning journal daily** *(copy from `weekly-habits-*.md` section `TABLE 1.2-C-mj`)*

**Table 1.2-C — Mind daily** *(from `daily-health-sections.mjs` MIND section)*

| Metric | Mon M/D | Tue M/D | Wed M/D | Thu M/D | Fri M/D | Sat M/D | Sun M/D |
|--------|---------|---------|---------|---------|---------|---------|---------|
| Spirit min | | | | | | | |
| Journal | | | | | | | |

**Table 1.2-C-mood — Mood by day** *(copy from `weekly-journal-feelings-*.md` section `TABLE 1.2-C-mood`)*

| Day | Date | Entries | Dominant | Day valence | Scores |
|-----|------|---------|----------|-------------|--------|
| | | | | | |

**Table 1.2-D — Mind insights** *(copy mood row from `weekly-journal-feelings-*.md` section `TABLE 1.2-D`)*

Present the table exactly as generated (2-column or flagged-entry table).

**Table 1.2-D-journal — Morning journal insights** *(copy from `weekly-journal-feelings-*.md` section `TABLE 1.2-D-journal`)*

**Table 1.2-E — Mood from journal** *(copy from `weekly-journal-feelings-*.md` section `TABLE 1.2-E`)*

| Metric | Last Week | 4-wk trend | Prior wk | → Notion field |
|--------|-----------|------------|----------|----------------|
| Mood Valence | | | | `Mood Valence` |
| Mood Negative % | | | | `Mood Negative %` |
| Entries (total) | | | | `Journal Count` |
| Entries scored | | | | (in `Journal Feelings Summary`) |
| Legacy tagged | | | | (in summary) |
| Dominant mood | | | | (in summary) |
| Score volatility | | | | (in summary) |
| Distress entries | | | | `Mood Distress Flag` |

→ Write `Mood Valence`, `Mood Negative %`, `Journal Feelings Summary`, `Mood Distress Flag` from the feelings file **Notion write payload** section. **`Mood Negative %`:** Notion percent field — store as decimal (`0.45` = 45%).

**Table 1.2-E-b — Mood score frequency** *(copy from `weekly-journal-feelings-*.md` section `TABLE 1.2-E-b`)*

| Tag | Count | Valence wt |
|-----|-------|------------|
| | | |

**Table 1.2-F — Energy + escalation** *(copy from feelings file; Energy = one AskQuestion turn)*

| Item | Source | → Notion field |
|------|--------|----------------|
| Energy | Ask Aaron 1–10 | `Energy Rating` |
| PHQ-2 / GAD-2 | Only if `Screening Escalation` = true in feelings file | `PHQ-2 Score`, `GAD-2 Score`, severities |
| Screening Escalation | from feelings file | `Screening Escalation` |

**Table 1.2-F-screen — PHQ-2 / GAD-2** *(only when `Screening Escalation` = true; one row per turn)*

| Item | Prompt | Score | → Notion field |
|------|--------|-------|----------------|
| PHQ-2 Q1 | Little interest or pleasure in doing things? | 0–3 | — |
| PHQ-2 Q2 | Feeling down, depressed, or hopeless? | 0–3 | — |
| PHQ-2 Total | Q1 + Q2 | 0–6 | `PHQ-2 Score` |
| PHQ-2 Severity | derived | | `PHQ-2 Severity` |
| GAD-2 Q1 | Feeling nervous, anxious, or on edge? | 0–3 | — |
| GAD-2 Q2 | Not being able to stop or control worrying? | 0–3 | — |
| GAD-2 Total | Q1 + Q2 | 0–6 | `GAD-2 Score` |
| GAD-2 Severity | derived | | `GAD-2 Severity` |

When escalation is **false**, leave PHQ-2/GAD-2 blank on the log (monthly plan calibrates screens).

**Capacity note** (if `Mood Valence` ≤ −0.3 OR `Mood Negative %` ≥ 60% OR `Energy` ≤ 4): state reduced work capacity — recorded in Phase 2.4 `Dev Capacity Note`.

**Table 1.2-G — Mind health** *(after Floor · Target · actual; Aaron confirms; write with approval)*

| Floor | Target | This week actual | Status | Evidence | → Notion field |
|-------|--------|------------------|--------|----------|----------------|
| from domain register | from domain register | session pulls | Below floor / At floor / Healthy / Not assessed | cite valence, negative %, spirit min | `Mind Health` |

**Table 1.2-H — Mind intentions (upcoming week)** *(Aaron approves before Notion write)*

| Intentions (1–3 bullets) | → Notion field |
|--------------------------|----------------|
| | `Mind Intentions` |

**Death chart (mandatory gate — after 1.2-H, before print preview / advance):** Remind Aaron to **mark this week on the death chart on the wall** (weeks-of-life grid; memento mori). Physical ritual only — **not** a Notion field, **not** a `Mind Intentions` bullet, **not** on the Phase 4b print preview intentions column. Confirm done (or will before the week starts) before `advance --step 1.2` → `1.3`. Record confirmation in ledger `notes` only (e.g. `death_chart: done`).

Append mind row to `Intentions Review` on the Weekly Meeting Log. Sync Notion, then **print preview:** `--section spirituality` — present verbatim; Aaron confirms → advance.

<a id="13-fitness-review-rate-intentions-5-min"></a>
### 1.3 Fitness — Review · Rate · Intentions (~5 min)

**Data sources:** `weekly-habits-*.md`, `weekly-wellness-trends-*.md`, `health_get_summary({ days: 28 })`, `daily-health-sections.mjs` (FITNESS section — includes Nutrition Log calories/protein), prior week's `Fitness Intentions`.

**Present exactly these tables, then health rating, then intentions:**

**Table 1.3-A — Prior fitness intention**

| Last week's `Fitness Intentions` | Evidence | Met? |
|----------------------------------|----------|------|
| | 1-line summary | ✓ / ~ / ✗ |

**Table 1.3-B — Fitness aggregate**

| Metric | Last Week | 4-wk trend | → Notion field |
|--------|-----------|------------|----------------|
| Strength Sessions | | | `Strength Sessions` |
| Cardio Sessions | | | `Cardio Sessions` |
| Weight Avg (lbs) | | | `Weight Avg` |
| Body Fat Avg (%) | | | `Body Fat Avg` |
| Lean Mass Avg (lbs) | | | `Lean Mass Avg` |
| Steps Avg | | | `Steps Avg` |
| Workout Active Min | | | `Workout Active Minutes` *(Workouts DB `Minutes` sum, review week — from `daily-health-sections.mjs` aggregate)* |
| Calories Avg | | | `Calories Avg` *(Nutrition Log per-day mean, logged days only — `daily-health-sections.mjs` Nutrition aggregate)* |
| Protein Avg (g) | | | `Protein Avg` *(mean g/day, logged days)* |
| Protein Days (≥150g) | | | `Protein Days` *(count of logged days hitting 150g)* |
| Supplements (days) | from `weekly-habits-*.md` TABLE 1.3-B-supp | ↑/↓ vs prior week | — |
| **Work close-out (days / avg sign-off CT)** | from `weekly-habits-*.md` TABLE work-closeout | vs prior week row | — |
| Heart Rate Avg (bpm) | | | `Heart Rate Avg` |
| Resting HR Avg (bpm) | | | `Resting HR Avg` *(not tracked — omit from integrity checks)* |
| HRV Avg (ms) | | | `HRV Avg` *(not tracked — omit from integrity checks)* |

**Table 1.3-C — Fitness daily** *(from `daily-health-sections.mjs` FITNESS section — **D1 `workouts`**; **Supplements** from same script / `weekly-habits-*.md` TABLE 1.3-C-supp)*

Copy **TABLE 1.3-B-supp** and **TABLE 1.3-C-supp** verbatim from `weekly-habits-*.md` § Supplements (Day Tracker) before or with Table 1.3-C.

**Strength** and **Cardio** rows show **activity labels**, not counts: strength = `Type` (Pull / Push / Legs / …); cardio = `Name` when set (e.g. Yoga, Vest Walk) else `Type`. Multiple same-day entries comma-separated. `—` when none logged.

| Metric | Mon M/D | Tue M/D | Wed M/D | Thu M/D | Fri M/D | Sat M/D | Sun M/D |
|--------|---------|---------|---------|---------|---------|---------|---------|
| Supplements | | | | | | | |
| Strength | | | | | | | |
| Cardio | | | | | | | |
| Weight (lbs) | | | | | | | |
| Steps | | | | | | | |
| Calories | | | | | | | |
| Protein (g) | | | | | | | |

*Nutrition rows come from the same `daily-health-sections.mjs` FITNESS section (Nutrition Log `393f40c2-487b-815b`). Nutrition logging is voluntary — show `—` on days with no food logged; **not** a FIELD CHECK hard-stop (unlike `Workout Active Minutes`). Persist `Calories Avg` / `Protein Avg` / `Protein Days` only when ≥1 day was logged.*

**Table 1.3-D — Fitness insights**

| Insight 1 | Insight 2 |
|-----------|-----------|
| | |

**Table 1.3-E — Fitness health** *(after Floor · Target · actual)*

| Floor | Target | This week actual | Status | → Notion field |
|-------|--------|------------------|--------|----------------|
| from domain register | from domain register | session pulls | four-value scale | `Fitness Health` |

**Table 1.3-F — Fitness intentions (upcoming week)**

| Intentions (1–3 bullets) | Strength target | Cardio target | → Notion field(s) |
|--------------------------|-----------------|---------------|-------------------|
| | | | `Fitness Intentions`, `Strength Target`, `Cardio Target` |

**Sub-step 1.3-N — Nutrition / calorie target (REQUIRED every week)**

Set an evidence-based daily **calorie goal** for the week (protein stays **150 g/day**). Feeds the dashboard's Fitness "Calories vs target" tile via `Calorie Target`.

> **This is a deterministic algorithm, not a vibe.** Run the steps exactly. The **Tunable weights** table holds every adjustable constant — change *those* (and note why in `Calorie Rationale`) as Aaron's real-world response teaches us his true numbers. Do not invent ad-hoc math elsewhere.

**Inputs** (lookback window `L` days, default **21**; sources: Health Data `Weight Lbs`, Nutrition Log `Calories`, Health Data `Steps` / `Workout Active Minutes`; `weekly-physique-coach.mjs` already compiles these):
- `W_start`, `W_end` = mean body weight over the first/last **3 logged weigh-ins** of the window.
- `R_week` = weekly weight change = `(W_end − W_start) / L × 7` lb/wk (positive = gaining).
- `C_logged` = mean **logged** calories per logged day; `N_logged` = number of logged days in the window.
- `Steps_avg` = mean daily steps. `kg` = `W_end / 2.2046`; `cm`, `age`, `sex` from `secrets/withings-credentials.json` (+ profile).

**Algorithm:**
1. **Observed TDEE** (energy balance): `TDEE_obs = C_logged − R_week × KCAL_PER_LB / 7`. *(Weight up ⇒ ate above maintenance ⇒ subtract; down ⇒ add.)*
2. **Formula TDEE** (Mifflin-St Jeor): `BMR = 10·kg + 6.25·cm − 5·age + S_sex`; `TDEE_formula = BMR × AF(Steps_avg)` using the activity-factor ladder below.
3. **Blend by logging confidence:** `w = clamp(N_logged / N_full, 0, 1)`; `TDEE = w·TDEE_obs + (1−w)·TDEE_formula`. *(Sparse logging ⇒ trust the formula; consistent logging ⇒ trust observed.)*
4. **Under-logging guard:** if `N_logged < N_full` **and** `C_logged` is implausibly low next to high `Steps_avg`/stable weight, **discard `TDEE_obs`**, use `TDEE_formula`, and flag "log intake more consistently" in the rationale.
5. **Target by direction:** `Maintain = TDEE`; `Lose = TDEE − DEFICIT`; `Gain = TDEE + SURPLUS`. **Round to nearest 50.**
6. **Expected weekly change** = `(Target − TDEE) / KCAL_PER_LB × 7` lb/wk. State it.

**Tunable weights** *(adjust these as Aaron's data teaches us; record changes in `Calorie Rationale`)*:

| Constant | Default | Meaning |
|---|---|---|
| `L` | 21 | lookback window (days) |
| `N_full` | 14 | logged-days for full confidence in observed method |
| `KCAL_PER_LB` | 3500 | kcal per lb of body mass |
| `AF` ladder (steps→factor) | <5k→1.4 · 5–8k→1.55 · 8–12k→1.7 · 12–18k→1.8 · >18k→1.9 | activity factor from `Steps_avg` |
| `S_sex` | +5 (male) / −161 (female) | Mifflin sex constant |
| `DEFICIT` | 400 (range 300–500) | daily cut for **Lose** (~0.6–1 lb/wk) |
| `SURPLUS` | 200 (range 150–300) | daily add for **Gain** (~0.25–0.5 lb/wk) |
| `PROTEIN_G` | 150 | daily protein target (constant) |

**Then:** ask direction (lettered — **A Lose / B Maintain / C Gain**), show the computed target + expected change, Aaron accepts or overrides.

**Table 1.3-N — Calorie target**

| TDEE (obs/formula/blend + N_logged) | Direction (A/B/C) | Calorie target | Expected weekly change | → Notion field(s) |
|---|---|---|---|---|
| | | | | `Calorie Target`, `Weight Goal Direction`, `Calorie Rationale` |

Write the estimator inputs + which method won into `Calorie Rationale` so the next run (and the dashboard) can see the reasoning. Append fitness row to `Intentions Review`.

Sync Notion, then **print preview:** `--section fitness` — present verbatim; Aaron confirms → advance.

<a id="13b-health-and-care-review-rate-intentions-2-min"></a>
### 1.3b Health & Care — Review · Rate · Intentions (~2 min)

**Purpose:** The body-maintenance domain that is not training — skin treatments, whitening, dental/medical follow-through, prescriptions. Rated against the **`Health & Care`** register domain (Floor · Target · mechanism live there, same as every other Phase 1 domain).

**Data sources:** `output/weekly-domain-board-*.md` (Floor · Target · actual for `Health & Care`), `context/self/skin.md` + `skin_get_context`, prior week's care intentions.

**Table 1.3b-A — Care review (last week)**

| Item | Last week | Target |
|------|-----------|--------|
| Skin treatments | count from care log | `skinTreatmentTarget` |
| Whitening sessions | count from care log | `whiteningTarget` |
| Appointments / follow-ups | list or `—` | — |

**Table 1.3b-B — Health & Care health** *(after Floor · Target · actual)*

| Floor | Target | This week actual | Status | → Log field |
|-------|--------|------------------|--------|-------------|
| from domain register (`Health & Care`) | from domain register | domain board actual | four-value scale | `care_health` |

**Table 1.3b-C — Care intentions (upcoming week)**

| Intentions (0–2 bullets, qualitative) | Skin treatment target | Whitening target | → Log field(s) |
|---------------------------------------|-----------------------|------------------|----------------|
| | number | number | `careIntentions`, `skinTreatmentTarget`, `whiteningTarget` |

No print-section slug — skip the preview here and `advance --step 1.3b` → `1.4`.

<a id="14-sleep-and-schedule-review-rate-intentions-5-min"></a>
### 1.4 Sleep and Schedule — Review · Rate · Intentions (~5 min)

**Data sources:** `health_get_summary({ days: 28 })`, `daily-health-sections.mjs` (SLEEP section), prior week's `Sleep Intentions` and `Schedule Intentions`.

**Present exactly these tables, then health rating, then intentions:**

**Table 1.4-A — Prior sleep intention**

| Last week's `Sleep Intentions` | Evidence | Met? |
|--------------------------------|----------|------|
| | 1-line summary | ✓ / ~ / ✗ |

**Table 1.4-A-b — Schedule review (last week)**

| Topic | Evidence | Met? |
|-------|----------|------|
| Time blocks honored | | ✓ / ~ / ✗ |
| Leave house early | | ✓ / ~ / ✗ |
| Stay on job | | ✓ / ~ / ✗ |
| Shutdown from work | | ✓ / ~ / ✗ |

→ Write narrative to `Schedule Review` (rich_text).

**Table 1.4-B — Sleep aggregate**

| Metric | Last Week | → Notion field |
|--------|-----------|----------------|
| Sleep Avg (recorded nights only) | | `Sleep Avg` |
| Nights Tracked | e.g. `2/7` | `Sleep Nights Tracked` |
| Wake-up Std Dev (min) | | `Wake Time Std Dev Min` |
| Bedtime Std Dev (min) | | `Bedtime Std Dev Min` |
| Schedule Rating | Consistent / Moderate Variance / Erratic / Unknown | `Sleep Schedule Rating` |

*Wake-up* = got out of bed (watch session end, CT). *Bedtime* = session start.

**Table 1.4-C — Sleep daily** *(from `daily-health-sections.mjs` SLEEP section)*

| Metric | Mon M/D | Tue M/D | Wed M/D | Thu M/D | Fri M/D | Sat M/D | Sun M/D |
|--------|---------|---------|---------|---------|---------|---------|---------|
| Sleep | | | | | | | |
| Wake-up | | | | | | | |
| Bedtime | | | | | | | |
| Deep min | | | | | | | |
| REM min | | | | | | | |

**Table 1.4-D — Sleep insights**

| Insight 1 | Insight 2 |
|-----------|-----------|
| | |

**Table 1.4-E — Sleep health** *(after Floor · Target · actual)*

| Floor | Target | This week actual | Status | → Notion field |
|-------|--------|------------------|--------|----------------|
| from domain register | from domain register | session pulls | four-value scale | `Sleep Health` |

**Table 1.4-F — Sleep and schedule intentions (upcoming week)**

| Sleep intentions (1–3 bullets) | Schedule intentions (1–3 bullets) | Sleep target (h) | Wake target (CT) | → Notion field(s) |
|--------------------------------|-----------------------------------|------------------|------------------|-------------------|
| | | | | `Sleep Intentions`, `Schedule Intentions`, `Sleep Target Hours`, `Target Wake Time` |

**Table 1.4-G — Behavioral adjustments** *(required when **this step's** status = **Below floor** — concrete commitments; **skip table** (write "—") when At floor or better)*

| Adjustment | Reason |
|------------|--------|
| | |

→ Write `Behavioral Adjustments` (append domain-labeled bullets; cumulative across Below-floor domains this session). **At floor and above: no adjustments needed.**

Sync Notion, then **print preview:** `--section sleep-schedule` — present verbatim; Aaron confirms → advance.

<a id="15-social-review-rate-intentions-5-min"></a>
### 1.5 Social — Review · Rate · Intentions (~5 min)

**Purpose:** Same domain pattern as mind/fitness/sleep. Per [social.md](../../self/social.md) — target 3–5 Small Talk entries/week; isolation signal is days since last entry.

**Data sources:** Small Talk DB (Phase 0), calendar social events (Phase 0), `weekly-wellness-trends-*.md`, prior week's `Social Intentions`.

**Present exactly these tables in order** (health and intentions separate turns):

**Table 1.5-A — Prior social intention**

| Last week's `Social Intentions` | Evidence | Met? |
|---------------------------------|----------|------|
| (each line) | 1-line summary | ✓ / ~ / ✗ |

**Overall intentions met** (one question): Mostly met / Partially met / Missed / Deprioritized / N/A → `Social Intentions Met`.

**Table 1.5-B — Social aggregate**

*Data source: `node scripts/social-phase-pull.mjs` (re-run after mid-session Small Talk logs).*

| Metric | Last Week | 4-wk trend | → Notion field |
|--------|-----------|------------|----------------|
| Small Talk entries | N — list each: `date — description` | from wellness file | `Small Talk Count` |
| Calendar social events | N — list each: `date — title` | from script | `Social Events Count` |
| Days since last Small Talk | N (0 = logged today) | | (in `Social Review`) |
| Unlogged contact | Aaron's answer | | (in `Social Review`) |

→ Write `Social Review` (rich_text bullets: what happened, missed, quality).

**Table 1.5-C — Social daily** *(Small Talk entries per day — days as columns)*

| Metric | Mon M/D | Tue M/D | Wed M/D | Thu M/D | Fri M/D | Sat M/D | Sun M/D |
|--------|---------|---------|---------|---------|---------|---------|---------|
| Small Talk | count or — | | | | | | |

**Table 1.5-D — Social insights**

| Insight 1 | Insight 2 |
|-----------|-----------|
| | |

**Table 1.5-E — Fuel check (two-stage)** *(per [eros.md](../../self/eros.md) — **mandatory before `advance` from 1.5**)*

**Stage 1 — Access**

| Question | Aaron answer | → Log |
|----------|--------------|-------|
| Is the fire accessible? | **Not accessible** / Limited / Accessible | `Social Review` (fuel section) |

**Stage 2 — Use** *(N/A when Not accessible)*

| Question | Aaron answer | → Log |
|----------|--------------|-------|
| If accessible: clean or contaminated? | Healthy / Contaminated / Divided / **N/A** | `Social Review` (fuel section) |

**Table 1.5-E-b — Fuel recovery intentions** *(required when Access = Not accessible or Limited, OR Use = Contaminated or Divided)*

| Recovery intentions (1–3 bullets) | → Log |
|-----------------------------------|-------|
| | `Social Review` + `Behavioral Adjustments` if fuel-led |

Two weeks running **Contaminated/Divided** (when accessible) → flag in `Social Review`; route to [eros.md](../../self/eros.md) daily container.

**Table 1.5-F — Social health** *(after Floor · Target · actual)*

| Floor | Target | This week actual | Status | → Notion field |
|-------|--------|------------------|--------|----------------|
| from domain register | from domain register | session pulls | four-value scale | `Social Health` |

**Table 1.5-G — Social intentions (upcoming week)**

| Social Priority | Weekly small-talk / sarges **target** (number → `Social Target`) | Qualitative intention (0–2 bullets, optional) | → Notion field(s) |
|-----------------|---------------------------------------------|-----------------------------------------------|-------------------|
| Active / Maintenance / Deprioritized | e.g. `5` (number only — **not** prose) | qualitative only — e.g. "host a dinner", "reconnect with an old friend"; **blank if no change** | `Social Priority`, `Social Target`, `Social Intentions` |

Deprioritized reason → note in `Social Review`. Append social row to `Intentions Review`.

**Table 1.5-H — Pre-commit** *(only when `Social Priority` = Active, or Maintenance with 0 calendar events)*

| Tactic | Booked? | Calendar / Todoist |
|--------|---------|-------------------|
| Meetup event | yes / no / N/A | |
| Fitness class | yes / no / N/A | |
| Social day block | yes / no / N/A | |
| Organic only | yes / no | |

Execute calendar/Todoist with approval. **Actual bookings** → Personal calendar (`hoegenauera@gmail.com`). **Time-block goals** → Personal Time Blocks calendar (`10283d615…@group.calendar.google.com`).

**Calendar weekday guard (required before any create/update from a weekday name):**
```
node scripts/lib/ct-weekday.mjs --today <YYYY-MM-DD> --weekday Wednesday
```
Uses canonical planning Monday for `--today`. Confirm stdout shows correct ISO + weekday before `calendar_create_event`. **Never** map weekday → date by mental math or `week_of + N`.

Sync Notion, then **print preview:** `--section social` — present verbatim; Aaron confirms → advance.

<a id="16-parenting-review-rate-intentions-4-min"></a>
### 1.6 Parenting — Review · Rate · Intentions (~4 min)

**Data sources:** Prior week's `Parenting Intentions`, prior Weekly Meeting Log parenting notes. **Do not** report legal custody schedule (2-2-5-5 is case context only — see `context/family/custody/`, not weekly planning). **Matthew days** for the upcoming week: read **Personal** calendar (`hoegenauera@gmail.com`) — camp reservations, school events, outings, medical. Time Blocks calendar shows structure goals only; real bookings live on Personal. → `context/family/matthew.md`

**Present exactly these tables in order** (health and intentions separate turns):

**Table 1.6-A — Prior parenting intention**

| Last week's `Parenting Intentions` | Evidence | Met? |
|------------------------------------|----------|------|
| (each line) | 1-line summary | ✓ / ~ / ✗ |

**Table 1.6-B — Parenting review (last week)**

| Metric | Last Week |
|--------|-----------|
| Time with Matthew | Aaron's narrative — camp, exchanges, letters, trips, etc. |
| Activities planned vs done | booked Personal-calendar events + anything notable |
| Quality / presence | 1-line |

**Table 1.6-C — Parenting insights**

| Insight 1 | Insight 2 |
|-----------|-----------|
| | |

**Table 1.6-D — Parenting health** *(after Floor · Target · actual)*

| Floor | Target | This week actual | Status | → Notion field |
|-------|--------|------------------|--------|----------------|
| from domain register | from domain register | session pulls | four-value scale | `Parenting Health` |

**Table 1.6-E — Parenting intentions (upcoming week)**

| Intentions (1–3 bullets) | → Notion field |
|--------------------------|----------------|
| concrete plans (outings, rhythm) + summer sanity note | `Parenting Intentions` |

Append parenting row to `Intentions Review`.

Sync Notion, then **print preview:** `--section parenting` — present verbatim; Aaron confirms → advance.

<a id="17-personal-enjoyment-2-min"></a>
### 1.7 Personal Enjoyment (~2 min)

**Present exactly these tables:**

**Table 1.7-A — Last week**

| Item | Detail |
|------|--------|
| Purely fun on calendar | list events (beyond Phase 1.5 social pre-commit) |
| Unscheduled enjoyment | anything fun not on calendar |

**Table 1.7-B — Enjoyment intention (upcoming week)**

| Intention (1–2 bullets) | On calendar? |
|-------------------------|--------------|
| what fun to plan or protect | yes / no / propose block |

**Table 1.7-C — Enjoyment health** *(after Floor · Target · actual)*

| Floor | Target | This week actual | Status | → Notion field |
|-------|--------|------------------|--------|----------------|
| from domain register | from domain register | session pulls | four-value scale | `Enjoyment Health` |

→ Write `Personal Enjoyment` (rich_text — last week + forward intention). Propose Personal Time Blocks calendar events with approval.

Sync Notion, then **print preview:** `--section enjoyment` — present verbatim; Aaron confirms → advance.

<a id="18-money-and-admin-review-rate-intentions-2-min"></a>
### 1.8 Money & Admin — Review · Rate · Intentions (~2 min)

**Purpose:** The last Phase 1 life domain — money hygiene and personal-life admin (bills paid, budget reviewed, statements reconciled, custody/legal/house paperwork moving). Rated against the **`Money & Admin`** register domain; rolls up to the **Admin** Values category.

**Data sources:** `output/weekly-domain-board-*.md` (Floor · Target · actual for `Money & Admin`), SimpleFIN / Finance dashboard tiles, prior week's `moneyIntentions`.

**Table 1.8-A — Money & admin review (last week)**

| Item | Last week | Note |
|------|-----------|------|
| Bills / statements handled | Aaron's narrative | — |
| Budget or accounts reviewed | yes / no | — |
| Admin paperwork moved | list or `—` | custody, will, house |

**Table 1.8-B — Money & Admin health** *(after Floor · Target · actual)*

| Floor | Target | This week actual | Status | → Log field |
|-------|--------|------------------|--------|-------------|
| from domain register (`Money & Admin`) | from domain register | domain board actual | four-value scale | `money_health` |

**Table 1.8-C — Money & admin intentions (upcoming week)**

| Intentions (0–2 bullets, qualitative) | → Log field |
|---------------------------------------|-------------|
| behavioral change only — **blank if no change** | `moneyIntentions` |

No print-section slug — skip the preview here and `advance --step 1.8` → `1.R`.

<a id="1r-personal-repair-and-debt-required-closes-phase-1"></a>
### `1.R` — Personal Repair & Debt *(REQUIRED — closes Phase 1)*

**Purpose:** Every personal domain has now been rated one at a time. This is the first moment they appear **side by side**, so the comparison — not the individual rating — drives the choice of what to fix. Pick **exactly one** personal domain to repair this week and **at most one** personal debt project to pay down.

> **The cap is per section, not per week.** One personal repair (`1.R`) + one work repair (`3.R`) = **at most two domains in repair per week**. Personal and work draw on different capacity, so one of each is affordable where two of either is not. The cap still exists: without it, the thirteen-domain list comes straight back in a new container.

**Data source:** `output/weekly-domain-board-*.md` — rows whose **Area = `Personal`**. Overlay this session's fresh ratings (`1.2`–`1.8`) on top of the register's stored status; a domain rated this session shows the fresh pick.

**Sub-step order within `1.R`:** `1.R.0` retrospective on **last week's** repair → `1.R.1` comparative board → `1.R.2` pick repair + intent → `1.R.3` pick debt. The retro runs **first** — before the board, before any new pick.

#### The repair loop across weeks *(applies to `1.R` and `3.R` alike)*

A repair is not a one-week gesture; it is a loop with four beats:

| Beat | Where | What happens |
|------|-------|--------------|
| **Pick** | `1.R.2` / `3.R.2` | One domain, one move, and **the intent** — the specific change Aaron is making |
| **Visible** | week board (`weekrepair` widget) | The domain, the move, and the intent text render all week on the **Personal — This Week** dashboard |
| **Review** | next week's `1.R.0` / `3.R.0` | Outcome picked against the domain's **fresh** rating from that session |
| **Resolve** | same turn | Either the rating changed (repair done → domain back to maintenance) or the **mechanism** gets adjusted (`*_repair_retro`) |

> **The loop is the whole point.** Without `.0`, the repair slot just rotates — a new domain every week, nothing ever seen through, and the register slowly fills with domains that were "worked on" and never fixed. The retro is what converts a repair from an intention into a closed loop.

**In-week surface — `weekrepair` widget.** The active repair(s) render on the **`personal-week`** dashboard (*Personal — This Week*) via the `weekrepair` widget, reading `week.plan.repairs` (both sections, filtered to rows with a domain). Each card shows **scope** (Personal / Work), **domain**, **the move** (Below floor → At floor, or At floor → Healthy, with the target status badge), and **the intent text**. It has **no checkbox by design** — a repair is not a task, so there is nothing to tick. When no repair is set the widget says so explicitly rather than rendering empty.

#### `1.R.0` — Last week's repair: retrospective (one turn, first) *(REQUIRED when one exists)*

Opens the step. Read the **review week's** log fields `personal_repair_domain` / `personal_repair_move` / `personal_repair_intent`. If the review week had **no** personal repair, skip straight to `1.R.1`.

**Table 1.R.0 — Prior personal repair**

| Domain | Move attempted | What he said he'd do | Reads now | Outcome |
|--------|----------------|----------------------|-----------|---------|
| `personal_repair_domain` from review-week log | Below floor → At floor *or* At floor → Healthy | `personal_repair_intent`, quoted verbatim | that domain's **fresh** rating from this session (`1.2`–`1.8`) | **Worked** / **Partial** / **Didn't work** → `personal_repair_outcome` |

Show the *now* rating **before** asking for the outcome — the fresh rating is the evidence, not Aaron's memory of the week.

- **Worked**, and the domain now rates above Below floor → note that **if that's the new normal rather than one good week**, the repair is done and the domain returns to maintenance (clear `in_repair`). Do not re-pick it at `1.R.2` on reflex.
- **Partial** or **Didn't work** → **required** free text: *"What blocked it — and does the mechanism need to change?"* → `personal_repair_retro`. The framing to hold him to: **effort is rarely the answer twice — what would hold this without you?** A retro that just says "be more disciplined" is not an answer.
- **Didn't work** → warn: **two failed attempts on the same domain means the floor is set wrong or nothing is holding it.** Revisit the floor itself (`floor_md` on the register row), not the effort.

#### `1.R.1` — Personal domain board (one turn)

**Table 1.R.1 — Personal domains** *(read-only; sorted worst-first — **Below floor** rows at top, then At floor / Not assessed, then Healthy)*

| Domain | Held by | Floor | Target | Status | Reviewed |
|--------|---------|-------|--------|--------|----------|
| one row per `Personal`-area register row | `delegation` or `—` | `floor_md` | `target_md` | effective status + trend ↑ / ↓ / − vs review-week log | `this week` when rated in this session, else **N days since review** |

Lead the turn with the count of domains below floor. If nothing is below floor, say so — the repair is then a push from **At floor → Healthy**, not a rescue.

#### `1.R.2` — One personal repair (one turn)

**Table 1.R.2 — Personal repair pick**

| Domain in repair | Move | Intent — *"What are you actually going to do?"* |
|------------------|------|------------------------------------------------|
| exactly one row | **Below floor → At floor** *or* **At floor → Healthy** | the specific change — a time block, a different default, a person or service to ask. **Explicitly not "try harder."** |

Set `in_repair` on exactly one `Personal` register row (`personalRepairDomainId` → `personal_repair_domain`, `personalRepairMove` → `personal_repair_move`), and capture the intent text (`personalRepairIntent` → `personal_repair_intent`).

**The intent is required, not optional colour.** It is what renders on the week board all week, and it is what gets read back verbatim at next week's `1.R.0`. "A repair without a stated move is just a wish" — if Aaron can't name the change, the pick isn't ready.

> **A repair is never a checkbox task.** It produces an **intention**, a **time block** scheduled at Phase `4.tb`, or a **change to the mechanism** that holds the domain. The only exception is when the repair *is* buying or installing a mechanism — that one becomes a real Task.

#### `1.R.3` — Personal debt (one turn)

**Table 1.R.3 — Personal debt pick**

| Debt project | Why now | Task IDs to add to slate |
|--------------|---------|--------------------------|
| at most one (`personalDebtProjectIds`) | | append to `notes.dev_slate_ids` |

**Debt is task-shaped** — unlike a repair. The selected project promotes onto the This Week slate and counts against the 40h cap. **Append** its Task IDs to the cumulative slate; **do NOT** run the sync here — the single sweep happens at `2.sync`.

`advance --step 1.R` → `1.check`.

**FIELD CHECK — Phase 1** *(Table 1.check)*

| Group | Required Notion fields |
|-------|------------------------|
| Last-week KPIs | `Strength Sessions`, `Cardio Sessions`, `Spirit Minutes`, `Journal Count`, `Weight Avg`, `Body Fat Avg`, `Lean Mass Avg`, `Sleep Avg`, `Sleep Nights Tracked`, `Wake Time Std Dev Min`, `Bedtime Std Dev Min`, `Sleep Schedule Rating`, `Steps Avg`, `Workout Active Minutes` |
| Week theme | `Week Intentions` — 1–3 sentence summary of the week's overarching theme (written/confirmed in Phase 4) |
| 1.2 | `Intentions Review` (mind row), `Mind Health`, `Mind Intentions`, `Mood Valence`, `Mood Negative %`, `Journal Feelings Summary`, `Mood Distress Flag`, `Energy Rating`, `Screening Escalation`; PHQ/GAD only if `Screening Escalation` = true |
| 1.3–1.4 | `Fitness/Sleep Health`, `Fitness/Sleep Intentions`, `Schedule Intentions`, `Schedule Review`, `Strength Target`, `Cardio Target`, `Calorie Target`, `Weight Goal Direction`, `Calorie Rationale`, `Sleep Target Hours`, `Target Wake Time`, `Behavioral Adjustments` |
| 1.5 | `Small Talk Count`, `Social Events Count`, `Social Review` (incl. **fuel rating**), `Social Intentions Met`, `Social Health`, `Social Priority`, `Social Target` (number), `Social Intentions` (qualitative, optional) |
| `1.4` (Below floor) | `Behavioral Adjustments` — **required for that domain**; skip when At floor or better |
| `1.5` | Fuel **Stage 1 + 2** + **recovery intentions** (1.5-E-b when triggered) in `Social Review` |
| 1.6 | `Parenting Health`, `Parenting Intentions` |
| 1.7 | `Personal Enjoyment`, `Enjoyment Health` |
| 1.3b | `care_health`, `careIntentions` (optional), `skinTreatmentTarget`, `whiteningTarget` |
| 1.8 | `money_health`, `moneyIntentions` (qualitative, optional) |
| `1.R.0` | `personal_repair_outcome` — **required whenever the review week had a personal repair**; `personal_repair_retro` — **required when outcome = Partial or Didn't work** |
| 1.R | Personal repair domain + move + **intent** (`personalRepairDomainId` → `personal_repair_domain`, `personalRepairMove` → `personal_repair_move`, `personalRepairIntent` → `personal_repair_intent`) + `in_repair` set on exactly one `Personal` register row; personal debt project (`personalDebtProjectIds`) if any, with its Task IDs appended to `notes.dev_slate_ids` |

**Do not proceed to Phase 2 (Work) until Table 1.check passes.**

<a id="phase-2-development-domain-first-18-min"></a>
## Phase 2: Development (domain-first, ~18 min)

**Purpose:** Review and plan dev work **one domain at a time** — **Turbo Gear → Chrome Lot → Systems** — surfacing the **strategy layer** (active **Goals** + their **milestones**, and **standalone Projects**), not just the Task tracker. For each dev domain: *review last week → set a weekly time goal → select the Goals / milestones / projects to get done this week.* Then a lighter **Workshop + Admin** tail, one **overall dev-health** rating, and a **single combined slate sync**.

**Step codes (ledger order):** `2.TG` Turbo Gear → `2.CL` Chrome Lot → `2.SY` Systems → `2.H` dev health → `2.WA` Workshop + Admin → `2.sync` commit slate → `2.check`. Each dev-domain ledger step spans three turns: **`.1` Review · `.2` Time goal · `.3` Select work** (advance the ledger once, after `.3`). **Repair & debt is no longer a Phase 2 step** — it is section-scoped: personal at `1.R`, work at `3.R`.

**Source files:** `output/weekly-dev-review-*.md` — now includes, per dev domain, a **`## Domain goals & projects — {domain}`** block (active Goals + their milestones + standalone Projects) alongside the existing review-week queue / time / carryover sections — plus `output/weekly-habits-*.md` and `node scripts/scan-tg-backlog.mjs` (TG orphan backlog).

**Layer model — read `context/systems/horizon-roadmap.md`.** **Goal** = finish-line outcome (the *why*). **Milestone** = a Projects row linked to a Goal (`🥅 Goals` set) — a stop on that goal's trail. **Standalone project** = a Projects row with no Goal. **Task** = the execution unit — the *only* layer that carries `📅 Week Tracker` (the weekly slate), Toggl, and time. **Selecting a milestone / standalone project for the week means promoting it to its Task tree and putting those Tasks on This Week** (`node scripts/promote-roadmap-to-dev-project.mjs --page=<projectId>` or the ▶ Start webhook); if it already has an open Task tree, just queue those Tasks.

**Selection accumulates across domains — sync ONCE.** Each `.3` records the resulting **Task IDs** into a running slate (ledger `notes.dev_slate_ids`). The actual `📅 Week Tracker` write happens **only** at `2.sync`, with the cumulative set — because `sync-dev-projects-this-week.mjs` **clears** the week relation on every open Task *not* passed in `--selected` (full-DB sweep). **Never** run the sync mid-loop with a single domain's IDs (it would clobber the other domains' picks).

**Presentation:** group by domain; nest sub-items under parents; letter each selectable root.

---

<a id="per-domain-loop-run-2tg-2cl-2sy"></a>
### Per-domain loop — run `2.TG` → `2.CL` → `2.SY`

The three dev domains share one shape. `{D}` = the domain: **Turbo Gear** (`2.TG`) · **Chrome Lot** (`2.CL`) · **Systems** (`2.SY`). Deep work = all three. Run three turns per domain, then `advance --step 2.{code}`.

#### `.1` — Review last week (one turn)

**Table 2.{D}-A — {D} accomplishments & carryover**

Bulleted tree (parents → sub-items) of that domain's review-week queued Tasks — `weekly-dev-review` § *Review week — queued dev work* → **{D}** group. ~~Strikethrough~~ finished (Status = Done); open items plain. Weave in the domain's rows from `weekly-dev-review` § *Table 2.1-B² — Activity summaries* (what actually shipped). Note any **detected unlogged** wins for this domain from `weekly-habits` sweep (Aaron flags: count as shipped y/n).

→ Contributes to `Accomplishments`, accomplishment counts, `Focused Output Hours Estimate`, `Dev Review` (written cumulatively; finalize at `2.sync`).

**Table 2.{D}-B — {D} time logged (review week)**

*Source: `weekly-dev-review` § Table 2.1-B — this domain's category column (Turbo Gear → **TG Dev**; Systems → **Systems**; Chrome Lot → **CL Dev**). `Total Time` formula.*

| Day | {D} min |
|-----|---------|
| … each day | |
| **Total** | |

Last week's **actual** dev minutes for the domain — anchors the time goal in `.2`, and the three domains' totals sum into `Deep Work Minutes` at `2.sync`.

#### `.2` — Weekly time goal (one turn) — REQUIRED

**Table 2.{D}-G — {D} weekly time goal**

| Last week actual (h) | This week goal (h) | → Notion field |
|----------------------|--------------------|----------------|
| 2.{D}-B total ÷ 60 | Aaron sets | **`Turbo Gear Hours Intended`** / **`Systems Hours Intended`** / **`Chrome Lot Hours Intended`** |

Agent shows last week's actual + a one-line capacity note (energy/capacity flags from Phase 1.2, upcoming calendar); Aaron gives the **hours** he wants to spend in this domain this week. Write the number to the Weekly Meeting Log field at commit.

> **Dashboard wiring (automatic):** the three `* Hours Intended` numbers dual-write to D1 and feed the daily dashboard's **Development → "Dev time — week vs goal"** tiles — `goal.tg_dev_target_min` / `goal.systems_target_min` / `goal.cl_dev_target_min` (minutes = hours × 60) plus the **Total Dev** goal (`goal.dev_total_target_min`). Each week's goals appear on the day board once the log syncs to D1 (next snapshot rebuild). Nothing extra to do here — just set the number.

#### `.3` — Select work (one or more turns) — the strategy layer

Present, for this domain, from `weekly-dev-review` § **Domain goals & projects — {D}**:

**Table 2.{D}-Goals — Active Goals + milestones** *(lettered)*

Each active Goal (`Status` = In progress / Not started) with progress % + target date, and its **open milestones** (Projects linked via `🥅 Goals`, Status not Done/Paused) nested beneath with completion %. Letter each **milestone** as a selectable pick.

**Table 2.{D}-Proj — Standalone Projects** *(lettered)*

Open standalone Projects (`🥅 Goals` empty, Status Roadmapped / In progress) with completion %. Letter each.

**Table 2.{D}-Carry — Open carryover Tasks** *(this domain, from 2.{D}-A)* — branch on turnover state (Phase 0a):

- **Pre-turnover (Fri/Sat — review week still current):** **A** = keep finishing this week (leave on current week; WeekDefer sweeps it Sunday) · **B** = defer now onto the planning-week slate (reply letters). *"Keep finishing" items are simply not added to the slate — leave their `📅 Week Tracker` on the current week.*
- **Post-turnover (Sun/Mon — auto-defer already ran):** **A** = continue all remaining open items · **B** = prune (reply letters to drop → clears `📅 Week Tracker`, back to backlog).

Before/while presenting, mark any items Aaron names as **Done** now (`Status → Done`) and re-fetch. Archive **empty Task records** (no `Name`) — do not present.

**(Turbo Gear only) Table 2.TG-Backlog — TG orphan-task backlog** *(capacity-gated)*

`node scripts/scan-tg-backlog.mjs` — un-queued TG **standalone Tasks** (bugs / small features / optimizations with **no Project**), grouped by Priority (High → Medium → Low → Unset), oldest-first, with a health line. This is **bottom-up** (orphan Tasks), distinct from the top-down Goals/Projects above. Surface only enough to fill the remaining TG hours from `2.TG-G` — cap **~3–5**, High/Medium first. Flag any **High aging past ~14d** as "must address." Record the backlog health line in `Dev Priority Context`. *(Browsing view: [Turbo Gear Backlog](https://notion.so/39bf40c2487b81f9a232d2ba0f1ab8e6).)*

**Reply:** letters of the milestones / standalone Projects / backlog / carryover items to **commit this week**.

**On selection — approval before any write:**

1. For each selected **milestone / standalone Project** **not yet promoted** (Status Roadmapped, no linked `Task`): promote → `node scripts/promote-roadmap-to-dev-project.mjs --page=<projectId>` (or ▶ Start). Creates its Task tree, Status → In progress. For already-promoted picks, take their **open Task IDs**.
2. For carryover keep/defer + TG-backlog picks: collect the **Task IDs** directly.
3. **Append** every resulting Task ID to the cumulative slate — record in ledger `notes.dev_slate_ids`. **Do NOT** run `sync-dev-projects-this-week.mjs` yet — the single sweep happens at `2.sync`.

`advance --step 2.{code}` → next domain (or `2.H` after `2.CL`).

<a id="2h-dev-health-review-one-turn-after-all-three-domains"></a>
### `2.H` — Dev health review (one turn, after all three domains)

**Table 2.H — Dev health** *(agent summarizes across TG + Systems + CL; Aaron rates once — no task list, selection is already done)*

Cover briefly: **(1) Output** — accomplishments vs queued work across the three domains (from each `.1-A`); **(2) Time** — total dev minutes logged (sum of the three `.1-B` totals) vs realistic capacity and vs the goals just set in `.2`; **(3) Goal progress** — what moved on the domains' Goals/milestones this review week (`Progress` / `Completion` %), what's stuck. **Agent does not recommend** — narrative only.

Then show **Floor · Target · this week's actual** for Work, and Aaron picks one status (four-value scale):

| Floor | Target | This week actual | Status | → Notion field |
|-------|--------|------------------|--------|----------------|
| from domain register | from domain register | session pulls | Below floor / At floor / Healthy / Not assessed | `Work Health`, `Dev Week Rating`, `Dev Intentions Met` |

Fold the summary into `Dev Review`. Optional forward theme (qualitative, 0–2 bullets) → `Dev Intentions`.

**Table 2.H-adj — Adjustments** *(only if `Work Health` = Below floor; skip when At floor or better)* — ask what adjustments Aaron commits to; capture his words only → `Dev Adjustments`.

`advance --step 2.H` → `2.WA`.

---

<a id="2wa-workshop-admin-lighter-tail"></a>
### `2.WA` — Workshop + Admin (lighter tail)

*Systems is now a first-class **dev** domain (handled in `2.SY`). This step covers only the two **non-dev** blocks — Workshop (QoL/hobby) and Admin (personal-life/legal) — which sit in their own blocks, never the deep-work block, and don't count as dev. One ledger step (`2.WA`); present the sub-tables below across turns, then advance.*

**Domain → block mapping (single-axis labels, see `capacity-rules.md`):**
- **Workshop** = QoL/hobby (home automation, Plex, dashboards) — schedule a **separate Workshop block, capped ~3 hr/wk**. Never in the deep-work block. Does not count as dev.
- **Admin** = personal-life/legal (custody, will, house) — schedule a **separate Admin block, outside the deep-work block**. Not dev.

**Required:** the plan must place a distinct **Admin block** and a distinct **Workshop block** outside the deep-work block. If Workshop candidates exceed the ~3 hr cap, defer the remainder — do not let tinkering crowd out deep work.

#### `2.WA-A` — Planning-month Workshop / Admin + last week

**Table 2.WA-A — Planning month Workshop / Admin Tasks**

Present `weekly-dev-review` § **Planning month — Workshop / Admin** (`🌙 Month` → planning month), **grouped by domain: Workshop / Admin**. Full parent → sub-bullet tree. Flag which block each root feeds (Workshop → capped block; Admin → admin block).

**Table 2.WA-B — Last week Workshop / Admin plan**

**Bulleted tree only** — one parent block per root; indent children. Append mirror + outcome on each line.

```
**Workshop**
- **Parent** — Status · mirror: none / open / completed · done: ✓ / ✗
  - **Child** — Status · mirror: … · done: …
**Admin**
- …
```

Source: prior `Dev Projects Intended` (Workshop/Admin) + prior-week `This Week` carryover for those domains + **Todoist MCP** (tasks due/completed last week). Confirm completions with Aaron.

#### `2.WA-W` — Workshop time intention *(REQUIRED every week)*

Workshop (QoL/hobby) is time-boxed so tinkering never crowds out deep work. Set the budget **and** aim it at specific item(s).

1. **Retrospective:** show **last week's Workshop actual** (Week Tracker `Workshop` rollup / `weekly-dev-review` § non-dev logged Workshop minutes) vs the prior week's **`Workshop Hours Intended`**. One line: `Workshop last week: actual Xh vs intended Yh`.
2. **Table 2.WA-W — Workshop plan** — present open **Domain = Workshop** items (lettered tree from `2.WA-A` Workshop group). Aaron replies with:
   - **hours** for `Workshop Hours Intended` (cap **~3 hr**; **0 = skip Workshop this week**), and
   - **letter(s)** for the item(s) that time goes to.
3. **Cap guard:** if the intention exceeds ~3 hr, confirm it's a deliberate trade against deep work before accepting. If 0, no Workshop items go on the slate this week.
4. **Append** selected Workshop + Admin Task IDs to the cumulative slate (`notes.dev_slate_ids`) — they're synced with everything else at `2.sync`. Write `Workshop Hours Intended` (number) + `Workshop Focus` (item names + one-line intention) to the Weekly Meeting Log at commit.

#### `2.WA-H` — Systems & Workshop health *(quick, every week)*

Rounds out the life-category ratings so **all** categories are trended (their `* Score` fields feed the annual/monthly/quarterly trend review). One rating per turn, four-value scale — **Floor · Target · actual** before each pick:

1. **Systems:** quick check — are your systems (automations, LCC, home/dev infra) running well? Any breakage or friction? Recommend 0–1 concrete change (→ a Systems Task/Project if warranted). **Rate `Systems Health`.** *(This is the standalone life-category `Systems Health` select for trending — distinct from the overall dev `Work Health` rated in `2.H`.)*
2. **Workshop:** given this week's Workshop intention/actual, **rate `Workshop Health`** (is the hobby/QoL tinkering in a good place — neither crowded out nor crowding out deep work?).

Write `Systems Health` + `Workshop Health` (select) to the Weekly Meeting Log at Phase 4 commit; the `Systems Score` / `Workshop Score` formulas populate automatically for trending (scores 1–3; `null` when Not assessed).

#### `2.WA-mirror` — Todoist mirrors

For each selected **Workshop / Admin** item, propose a **Todoist mirror** (due date + project) — **case-by-case approval** before create. Verify last week's mirrors via Todoist MCP.

`advance --step 2.WA` → `2.sync`.

---

<a id="2sync-commit-the-combined-slate-run-once"></a>
### `2.sync` — Commit the combined slate (run once)

**Now** run the single sweep with the full accumulated set from every `.3` + `2.WA` + the debt picks from **both** `1.R.3` (personal) **and** `3.R.3` (work) (approval before Notion writes):

`node scripts/sync-dev-projects-this-week.mjs --selected=<all cumulative Task IDs>`

1. **Link** `📅 Week Tracker` = **planning week** on every selected Task.
2. **Clear** `📅 Week Tracker` on **every other open** Task (full open-DB sweep → back to backlog), and on any **Done** rows still linked (stale bulk-close artifacts).
3. **Toggl tasks** — same script chains `sync-dev-projects-toggl-tasks.mjs`: create/assign Focus tasks for the slate; **delete** off-slate mirrors (and clear `Toggl Task ID`).

*Pre-turnover: "keep finishing" items keep their current-week link untouched — this sweep only manages the **planning-week** slate.*

> **Debt picks from both repair steps land here.** `1.R.3` resolves before this sweep, so its Task IDs are already on `notes.dev_slate_ids`. `3.R.3` resolves **after** it (Operations runs after Development), so when a work debt project is picked at `3.R`, append its Task IDs to the same cumulative slate and **re-run this sweep** at Phase 4 step 3 before confirming the slate. The slate is one list for the week, not one per phase.

→ Write `Deep Work Minutes` (sum of the three domains' review-week actual dev minutes from each `2.{D}-B`), `Dev Projects Intended` (slate snapshot), `Dev Priority Context` (incl. the TG backlog health line).

Sync Notion, then **print preview:** `node scripts/weekly-plan-section-preview.mjs --ledger <path> --section development` — present verbatim; Aaron confirms.

**Present Table 2.S — This Week slate** — sync-script output, all domains (Chrome Lot / Turbo Gear / Systems / Workshop / Admin). Aaron executes from the [This Week — Dev Slate](https://notion.so/39bf40c2487b81d2a7acf44e0706775f) view (Start Timer · Logged Minutes This Week · TWC). Aaron confirms before `2.check`.

`advance --step 2.sync` → `2.check`.

---

**Table 2.check — Final This Week slate** *(required before Phase 4)*

Run `sync-dev-projects-this-week.mjs` output (or `weekly-habit-summary.mjs` § This Week) and present **one bulleted tree** covering all domains:

```
**Chrome Lot**
- parent → children
**Turbo Gear**
- …
**Systems**
- …
**Workshop**
- …
**Admin**
- …
```

This list must **exactly match** the Notion Tasks view filtered to `This Week = true` (open items). Aaron confirms **A** = matches / **B** = drift to fix.

**FIELD CHECK — Phase 2** *(gate)*

| Field | Step |
|-------|------|
| `Deep Work Minutes`, `Accomplishments`, counts | 2.{D}.1 / 2.sync |
| `Turbo Gear Hours Intended` | 2.TG.2 |
| `Systems Hours Intended` | 2.SY.2 |
| `Chrome Lot Hours Intended` | 2.CL.2 |
| `Dev Review`, `Work Health`, `Dev Week Rating`, `Dev Intentions Met` | 2.H |
| `Dev Adjustments` | 2.H-adj *(Below floor only; Aaron-supplied)* |
| `Dev Projects Intended`, `Dev Priority Context` | 2.sync |
| `This Week` slate synced (all domains) | 2.sync |
| `Workshop Hours Intended`, `Workshop Focus` | 2.WA-W |
| `Systems Health`, `Workshop Health` | 2.WA-H |
| **Final slate = Notion view** | **2.check** |

*Repair & debt is no longer checked here — personal repair is gated at `1.check`, work repair at `3.R`.*

**Do not proceed to Phase 3 (Operations) until `2.check` passes.**

<a id="phase-3-operations-10-min"></a>
## Phase 3: Operations (~10 min)

**Purpose:** Office ops and field/CRM review, then the **work** repair. Ops review and assignment run as `3.ops.1` / `3.ops.2` (Office ops — review · this week) and `3.field.1` / `3.field.2` (Field & CRM — review · this week). Phase 3 closes with `3.R` — pick exactly one **professional** domain to elevate (`work_repair_domain`).

**Step codes (ledger order):** `3.ops.1` → `3.ops.2` → `3.field.1` → `3.field.2` → `3.R` Work — repair & debt → `4.tb`.

<a id="3r-work-repair-and-debt-required-closes-phase-3"></a>
### `3.R` — Work Repair & Debt *(REQUIRED — closes Phase 3)*

**Purpose:** The work-side mirror of `1.R`. Every work-area domain side by side, worst-first; pick **exactly one** to repair and **at most one** work debt project to pay down.

> **Per-section cap.** One personal repair (`1.R`) + one work repair (`3.R`) = **at most two domains in repair per week** — never two work repairs. Personal and work draw on different capacity, which is why one of each is affordable. The cap itself stays: drop it and the thirteen-domain list returns in a new container.

**Scope:** every register row whose Area is **not** `Personal` — **Chrome Lot, Turbo Gear, Systems, Workshop, Admin**.

**Data source:** `output/weekly-domain-board-*.md` (see Phase 0 § *Domain register pull*), overlaid with this session's fresh ratings (`2.H`, `2.WA-H`, the ops steps).

> **A stale review is itself a signal.** Departments that were **not** rated in this session display their **last register status** plus **how many days since review** — an unreviewed department reads as unreviewed, not silently as a pass. A CL department last rated 40 days ago is information, not a blank.

**Sub-step order within `3.R`:** `3.R.0` retrospective on **last week's** work repair → `3.R.1` comparative board → `3.R.2` pick repair + intent → `3.R.3` pick debt. Same four-beat loop as `1.R` — see *The repair loop across weeks* in the `1.R` section.

#### `3.R.0` — Last week's work repair: retrospective (one turn, first) *(REQUIRED when one exists)*

Read the **review week's** log fields `work_repair_domain` / `work_repair_move` / `work_repair_intent`. If the review week had no work repair, skip to `3.R.1`.

**Table 3.R.0 — Prior work repair**

| Domain | Move attempted | What he said he'd do | Reads now | Outcome |
|--------|----------------|----------------------|-----------|---------|
| `work_repair_domain` from review-week log | Below floor → At floor *or* At floor → Healthy | `work_repair_intent`, quoted verbatim | that domain's status as of this session (`2.H`, `2.WA-H`, the ops steps — or its register status + days since review if unrated) | **Worked** / **Partial** / **Didn't work** → `work_repair_outcome` |

- **Worked**, and the domain now rates above Below floor → if that's the **new normal rather than one good week**, the repair is done and the domain returns to maintenance (clear `in_repair`).
- **Partial** / **Didn't work** → **required** free text: *"What blocked it — and does the mechanism need to change?"* → `work_repair_retro`. **Effort is rarely the answer twice — what would hold this without you?** On the work side that usually means delegation, a Process Street workflow, or an automation, not more of Aaron's hours.
- **Didn't work** → warn: two failed attempts on the same domain means **the floor is set wrong or nothing is holding it** — revisit `floor_md`, not the effort.

#### `3.R.1` — Work domain board (one turn)

**Table 3.R.1 — Work domains** *(read-only; sorted worst-first — **Below floor** rows at top, then At floor / Not assessed, then Healthy)*

| Domain | Held by | Floor | Target | Status | Reviewed |
|--------|---------|-------|--------|--------|----------|
| one row per non-`Personal` register row (domain name · area) | `delegation` or `—` | `floor_md` | `target_md` | effective status + trend ↑ / ↓ / − vs review-week log | `this week` when rated in this session, else **N days since review** |

Lead the turn with the count of domains below floor. If nothing is below floor, the repair is a push from **At floor → Healthy**.

#### `3.R.2` — One work repair (one turn)

**Table 3.R.2 — Work repair pick**

| Domain in repair | Move | Intent — *"What are you actually going to do?"* |
|------------------|------|------------------------------------------------|
| exactly one row | **Below floor → At floor** *or* **At floor → Healthy** | the specific change — a time block, a different default, a person or service to ask. **Explicitly not "try harder."** |

Set `in_repair` on exactly one non-`Personal` register row (`workRepairDomainId` → `work_repair_domain`, `workRepairMove` → `work_repair_move`), and capture the intent text (`workRepairIntent` → `work_repair_intent`).

**The intent is required.** It renders on the week board all week (`weekrepair` widget, scope **Work**) and is read back verbatim at next week's `3.R.0`.

> **A repair is never a checkbox task** — intention, time block, or mechanism change. Exception: when the repair *is* buying or installing a mechanism, that becomes a real Task.

#### `3.R.3` — Work debt (one turn)

**Table 3.R.3 — Work debt pick**

| Debt project | Why now | Task IDs to add to slate |
|--------------|---------|--------------------------|
| at most one (`workDebtProjectIds`) | | append to `notes.dev_slate_ids` |

**Debt is task-shaped** — it goes on the This Week slate and counts against the 40h cap. Append its Task IDs to the cumulative slate and **re-run the `2.sync` sweep** at Phase 4 step 3 so the slate reflects both repair steps.

**FIELD CHECK — Phase 3** *(gate)*

| Field | Step |
|-------|------|
| `opsHoursIntended` | 3.ops.2 |
| `fieldHoursIntended`, `fieldActivityTarget` | 3.field.2 |
| `work_repair_outcome` *(required when the review week had a work repair)*; `work_repair_retro` *(required when outcome = Partial / Didn't work)* | 3.R.0 |
| Work repair domain + move + **intent** (`work_repair_domain`, `work_repair_move`, `work_repair_intent`) + `in_repair` on exactly one non-`Personal` register row | 3.R.2 |
| Work debt project (`workDebtProjectIds`) if any + Task IDs on `notes.dev_slate_ids` | 3.R.3 |

`advance --step 3.R` → `4.tb`.

**Do not proceed to Phase 4 until last week's work repair is closed out (`3.R.0`, when one existed) and this week's work repair is set with its intent (or explicitly declined for the week).**

<a id="phase-4-commit-5-min"></a>
## Phase 4: Commit (~5 min)

**Purpose:** Final review, capacity check, execute remaining actions, log everything.

1. **Summary table:** Everything planned across all phases (project selections, personal items, dev intentions)
2. **Final capacity check:** Total planned hours vs. available hours. If total exceeds available, something must move. This is non-negotiable.
3. **Confirm "This Week" checkboxes:** Verify all selected Tasks have `This Week = true` and no deselected ones still have it checked. **If `3.R.3` picked a work debt project after the `2.sync` sweep, re-run `sync-dev-projects-this-week.mjs` with the full cumulative slate first** — otherwise the sweep would clear that debt project's week link.
4. **Store project KPIs on Weekly Meeting Log:** Write `Projects Completed` (count of projects marked Done this week) and `Projects In Progress` (count of projects with This Week checked for the new week).
5. **Verify all FIELD CHECKs (REQUIRED):** Re-run Phase 1 (`1.check`), Phase 2 (`2.check`), and the Phase 3 check (incl. both repair retros and both repair picks). Confirm nothing is blank without N/A + reason. (Activity KPIs + Team Activity Details → **Weekly Ops** commit.)
6. **Write / confirm `Week Intentions` (REQUIRED):** 1–3 sentence week theme capturing the overarching focus for the planning week. Agent proposes from session context; Aaron confirms or edits → write to Weekly Meeting Log. `node scripts/weekly-plan-log-check.mjs commit --ledger <path>` must pass before `workflow-notion-log complete`.
7. **Record life health ratings (REQUIRED):** Verify weekly-rated selects are set — `Mind Health` (→ `Spirituality Health` in Phase 4), `Fitness Health` (1.3), `care_health` (1.3b), `Sleep Health` (1.4), `Social Health` (1.5), `Parenting Health` (1.6), `Enjoyment Health` (1.7), `money_health` (1.8), `Work Health` (2.H), `Systems Health` / `Workshop Health` (2.WA-H). Values: **Below floor / At floor / Healthy / Not assessed** (four-value scale). The **Admin** Values category is now covered by `1.8` Money & Admin.
7b. **Record the repair-loop fields (REQUIRED):** Verify the D1 `meeting_logs` record carries this week's repair picks **and** last week's close-out. These are the fields that make the loop work across weeks — a blank `*_repair_intent` leaves the week board with nothing to show, and a blank `*_repair_outcome` breaks next week's `.0` retro.

    | D1 log field | Set at | Required when |
    |--------------|--------|---------------|
    | `money_health` | 1.8 | always (four-value scale) |
    | `personal_repair_domain` | 1.R.2 | a personal repair is picked |
    | `personal_repair_move` | 1.R.2 | a personal repair is picked |
    | `personal_repair_intent` | 1.R.2 | a personal repair is picked |
    | `personal_repair_outcome` | 1.R.0 | the **review week** had a personal repair |
    | `personal_repair_retro` | 1.R.0 | outcome = **Partial** or **Didn't work** |
    | `work_repair_domain` | 3.R.2 | a work repair is picked |
    | `work_repair_move` | 3.R.2 | a work repair is picked |
    | `work_repair_intent` | 3.R.2 | a work repair is picked |
    | `work_repair_outcome` | 3.R.0 | the **review week** had a work repair |
    | `work_repair_retro` | 3.R.0 | outcome = **Partial** or **Didn't work** |

    Repair **move** values are stored as `below-to-floor` / `floor-to-healthy`; outcome as `Worked` / `Partial` / `Didn't work`. Debt picks persist alongside as `personal_debt_project_ids` / `work_debt_project_ids`.

8. **Update Values DB Health (with approval):** For each category where this week's status differs from current Values DB Health, update via `personal_notion_update_page` on the category page in Values DB (`342f40c2-487b-80c5`). Include **Personal Enjoyment** when added to Values DB. Use the four-value vocabulary.
9. **Confirm accomplishment fields (REQUIRED):** Verify Phase 2 (`2.sync`) wrote `Logged/Unlogged/Total Accomplishments Count`, `Focused Output Hours Estimate`, and `Accomplishments`. Backfill from habit summary if missing.
9b. **Write Workshop intention (REQUIRED):** From Phase 2.WA-W, write `Workshop Hours Intended` (number, 0 allowed) + `Workshop Focus` (rich_text — selected item(s) + one-line intention) to the Weekly Meeting Log. `weekly-plan-log-check` commit gate requires `Workshop Hours Intended`.
10. **Body comp already persisted.** Withings written in Phase 0 (`--days 28`). Don't re-run here.
11. **Execute remaining:** Create any Todoist/Calendar/Notion items not yet committed during earlier phases.
12. **Log to Notion:** Finalize the Weekly Meeting Log entry (`322f40c2-487b-81bd`) with key decisions, action items, and plan summary. Set `Status = Done`, `Session Complete = Complete`.
13. **Personal Time Blocks (`4.tb` — REQUIRED after scheduling, before 4b):**

    **Delegate to subskill** — execute [`context/skills/plan-weekly-schedule/SKILL.md`](../plan-weekly-schedule/SKILL.md) **end-to-end** (Gates 0–4). Do not inline a shortened version. Weekly plan provides ledger context only:

    ```
    node scripts/weekly-time-blocks.mjs --ledger <path>
    ```

    Subskill gates (mandatory):
    1. **PWS-0** — declare Bus/custody state; ask if unknown
    2. **PWS-1** — confirm remote/in-person + locations for meetings; resolve piano/pickup conflicts → `config/schedule-overrides-{week_sunday}.json`
    3. Dry-run → present PWS-B / PWS-C / PWS-H
    4. Aaron approves → `--bus-confirmed --events-confirmed --apply` (or skip **C**)

    `advance --step 4.tb` → proceed to `4b`.

**Table 4.tb-A — Proposed time blocks (planning week Mon–Fri)**

| Day | Time | Block | Type (color) |
|-----|------|-------|--------------|
| Mon M/D | HH:MM–HH:MM | summary | Gym / Home / Office / Field / Bus / Wind-down |
| … | | | |

*Source: `output/weekly-time-blocks-{week}.md` from dry-run script — present verbatim, grouped by day.*

**Table 4.tb-B — Calendar integration (conflicts & adjustments)**

| Source calendar | Event | Overlap / conflict | Adjustment |
|-----------------|-------|-------------------|------------|
| Personal | e.g. Minecraft camp Jul 20–24 | all-day camp | switched to `camp_all_day` template |
| Work | e.g. client call Tue 2 PM | overlaps deep work | carved ops block earlier |
| — | none | — | default template |

*Source: script conflict section + agent notes from Phase 1.5/1.6 social/parenting pre-commits and Work calendar pull.*

14. **Week Tracker summary (4b — REQUIRED):**
    - Planning week was confirmed at **0a** (`planning_week_page_id` on ledger). Optional gate — `node scripts/weekly-plan-section-preview.mjs --ledger <path> --all` (full print preview).
    - After Aaron confirms, run `node scripts/weekly-plan-week-summary.mjs --ledger <path>` (uses ledger planning week — no end-of-session week picker). This (1) renders a **print PDF** via Playwright (also saves `output/weekly-plan-print-{week}.html` + `.pdf`), (2) uploads PDF to `Plan Records/weekly/`, (3) sets **`Plan Doc URL`** on the planning week record, (4) appends/replaces the **Weekly Plan** section on that Notion page. Aaron approves production writes.
15. **Update context files** if anything changed.

<a id="cross-cutting-rules"></a>
## Cross-Cutting Rules

- **Table contract per phase.** Phase 1 = life domains (`1.2` → `1.3` → `1.3b` → `1.4` → `1.5` → `1.6` → `1.7` → `1.8`) → `1.R` personal repair & debt → `1.check`. Phase 2 = domain-first loop `2.TG` → `2.CL` → `2.SY` (each `.1` review · `.2` time goal · `.3` select) → `2.H` dev health → `2.WA` Workshop/Admin → `2.sync` → `2.check`. Phase 3 = `3.ops.1/.2` → `3.field.1/.2` → `3.R` work repair & debt. CL ops detail → **Weekly Ops** skill.
- **Repair is section-scoped, one per section.** One personal repair (`1.R`) + one work repair (`3.R`) — at most two domains `in_repair` per week, never two of the same section. Debt: at most one project per section, both landing on the single This Week slate.
- **Repair closes its loop.** Each repair step runs `.0` retro → `.1` board → `.2` pick + intent → `.3` debt. The retro on **last week's** repair happens *before* a new one is chosen, judged against this session's fresh rating — never on memory. Between sessions the repair stays visible on the week board (`weekrepair` widget). Skipping `.0` is what turns the repair slot into a rotation where nothing is ever seen through.
- **FIELD CHECK gates.** Run `1.check` before Phase 2 development; `2.check` after development; the Phase 3 check after `3.R`; verify all in Phase 4 commit.
- **Route every item into a bucket.** Each surfaced item is Automated (n8n), Delegated (team 1:1s), or a Scheduled slice (calendar + Todoist mirror).
- **Capacity is non-negotiable.** If total planned work exceeds available hours minus 10-15% buffer, the system pushes back. Something must move.
- **Delegation by default.** For any task deferred 3+ times, suggest delegation before rescheduling. Use the delegation framework in `context/systems/capacity-rules.md`.
- **Max 5 must-do Todoist tasks per day.** If morning briefing shows >5, defer.
- **All data pulls happen in Phase 0 silently.** ~40 minutes is for discussion and decisions.
- **Quarterly docket is the source.** Weekly project selection pulls only from the current quarter's assigned projects. Don't ad-hoc backlog items.

<a id="outputs"></a>
## Outputs

- **Pre-Phase 0:** Monthly plan gate pass (or full monthly plan run + resume).
- **Phase 0a:** Review + Planning Week Tracker rows confirmed on ledger + Weekly Meeting Log relations.
- **Phase 0:** Wellness + journal feelings + social + dev data pulls; **domain board** (`weekly-domain-board.mjs`); trend files; 4-week log history.
- **Phase 0b:** Data integrity table; remediation before Phase 1.
- **Phase 1:** Life review (values, mind, fitness, health & care, sleep, social, parenting, personal enjoyment, money & admin) + targets on Weekly Meeting Log + **last week's personal repair closed out** (`1.R.0`) and **one personal domain in repair with a stated intent**, at most one personal debt project (`1.R`).
- **Phase 2:** Development review + next-week dev plan + the single combined slate sweep.
- **Phase 3:** Office ops + field/CRM plan + **last week's work repair closed out** (`3.R.0`) and **one work domain in repair with a stated intent**, at most one work debt project (`3.R`).
- **Phase 4:** Full Weekly Meeting Log finalized + all FIELD CHECKs; Values DB sync (with approval).
- **Phase 4.tb** — Personal Time Blocks calendar — one-time Mon–Fri structure events; adjust-if-exists or full regenerate with approval. Skill → `context/skills/plan-weekly-schedule/SKILL.md`.
- **Phase 4b:** Planning week record (from 0a) gets domain-by-domain plan summary + linked Google Doc in `Plan Records/weekly/`.
- **Between sessions:** the week's repair(s) — scope, domain, move, and intent text — render on the **`personal-week`** dashboard via the `weekrepair` widget (no checkbox), and come back as the input to next week's `1.R.0` / `3.R.0`.

<a id="failure-modes-and-graceful-degradation"></a>
## Failure modes & graceful degradation

- **Monthly Plan Log missing for planning month:** Hard stop — run monthly plan first (Pre-Phase 0). Do not proceed weekly-only.
- **Weekly data pull script missing:** Rely on parallel MCP pulls per Phase 0 list.
- **Withings sync / `sources.notion.error`:** Skip or note body-comp unavailable; use `--` for body-comp rows; note "Withings sync inactive" or "Withings sync needs attention" in scorecard/footer as specified in Phases 1–2.
- **`sources.health_sync.ok = false`:** Drop Recovery + Activity lines in Values Pulse fitness callout (Phase 1); append diagnostic note; render watch metrics as `--` in habit scorecard with "Health Sync data missing" in footer; do not abort the workflow.
- **`health_persist_recent` / Health Sync issues:** Note in footer; continue with MCP + archived Notion data where available.
- **Partial or null watch fields (RHR, HRV):** Render as `--` until Health Sync folders are enabled.
- **Prior week missing `Week Intentions`:** Hard stop at Phase 0b — `workflow-progress advance --step 0b` refuses until backfilled (`scripts/backfill-week-intentions.mjs`) or prior session Phase 4 re-run. Next week's plan depends on reading last week's theme.
- **Commit missing required fields:** `workflow-notion-log complete` runs `checkCommitFields()` — refuses if `Week Intentions` or other 1.check/2.check fields blank.

<a id="see-also"></a>
## See also

- `../../router.md`
- `../monthly-plan/SKILL.md` — monthly review cadence
- `../quarterly-plan/SKILL.md` — quarterly docket source for project selection
- `../../systems/cadences.md`
- `../../systems/capacity-rules.md`
- `../../systems/notion-databases.md`
- `../../systems/knack-fields.md`
- `../../systems/hubstaff.md`
- `../../systems/health-data.md`
- `../../self/values.md`, `../../self/current-priorities.md`
- `../../people/index.md`
- `../../systems/weekly-plan-app.md` — in-app weekly wizard (domain ratings + repair picks)
- `../../work/turbo-gear/overview.md`
