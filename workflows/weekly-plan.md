> **Source:** [`context/skills/weekly-planning/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-planning/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

﻿---
updated: 2026-06-18
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
  - [Social pulls (with wellness; used in Phase 1.5)](#social-pulls-with-wellness-used-in-phase-15)
  - [Development pulls (after wellness/social; silent until Phase 2+)](#development-pulls-after-wellnesssocial-silent-until-phase-2)
- [Phase 0b: Data Integrity Gate (~3 min)](#phase-0b-data-integrity-gate-3-min)
- [Phase 1: Life Review (~28 min)](#phase-1-life-review-28-min)
  - [1.0 Create Weekly Log Entry](#10-create-weekly-log-entry)
  - [1.1 Values Context (~1 min) — **first table to Aaron**](#11-values-context-1-min-first-table-to-aaron)
  - [1.2 Mind — Review · Mood · Rate · Intentions (~6 min)](#12-mind-review-mood-rate-intentions-6-min)
  - [1.3 Fitness — Review · Rate · Intentions (~5 min)](#13-fitness-review-rate-intentions-5-min)
  - [1.4 Sleep and Schedule — Review · Rate · Intentions (~5 min)](#14-sleep-and-schedule-review-rate-intentions-5-min)
  - [1.5 Social — Review · Rate · Intentions (~5 min)](#15-social-review-rate-intentions-5-min)
  - [1.6 Parenting — Review · Rate · Intentions (~4 min)](#16-parenting-review-rate-intentions-4-min)
  - [1.7 Personal Enjoyment (~2 min)](#17-personal-enjoyment-2-min)
- [Phase 2: Development (domain-first, ~18 min)](#phase-2-development-domain-first-18-min)
  - [2.1 Dev Review (~8 min)](#21-dev-review-8-min)
  - [2.2 Systems / Workshop / Admin Review (~7 min)](#22-systems-workshop-admin-review-7-min)
- [Phase 4: Commit (~5 min)](#phase-4-commit-5-min)
- [Cross-Cutting Rules](#cross-cutting-rules)
- [Outputs](#outputs)
- [Failure modes & graceful degradation](#failure-modes-and-graceful-degradation)
- [See also](#see-also)

---


<a id="trigger"></a>
## Trigger

This skill activates when Aaron says "weekly plan", "weekly meeting", "plan this week", "sprint planning", or "Monday review". Target duration: ~45 minutes (Phase 1 life ~28 min · Phase 2.1 Dev Review ~8 min · Phase 2.2 Personal Projects ~7 min · Phase 4 commit ~5 min). **CL operations** run in **Weekly Ops** — `context/skills/weekly-ops/SKILL.md`.

<a id="inputs"></a>
## Inputs

Load via the router. Read these before starting:

- `context/systems/cadences.md` — cadence health check, Phase 0 schedule
- `context/systems/capacity-rules.md` — limits, overcommitment triggers, intervention protocol
- `context/systems/notion-databases.md` — every DB ID referenced below (Tasks, Weekly Meeting Log, source DBs, Values, Workouts, etc.)
- `context/systems/knack-fields.md` — Customer + Photographer field references
- `context/systems/hubstaff.md` — member IDs, weekly-report tool
- `context/systems/health-data.md` — MCP architecture, expected fields
- `context/self/values.md` — six categories and current Health statuses (Phase 1.1 context; per-domain ratings in Phase 1 + 2.2)
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
    | `2.1` (after sync + **2.1-S** confirmed) | `development` | Includes CL/TG + personal dev tree when on log |

    Optional before **4b** write: `--all` for full seven-domain preview.
12. **Phase gates:** `node scripts/workflow-progress.mjs gate --workflow weekly-plan --phase <1|2>` before Phase 2 (work) or Phase 4 (commit)
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
2. **Selection sync** — after each Phase 2 sub-step (2.1 then 2.2), run `scripts/sync-dev-projects-this-week.mjs` with the cumulative selected page IDs: set `This Week = true` on selected only; **`This Week = false` on every other open Task** (domain-scoped within that phase, then combined pass at 2.check). That script also **provisions Toggl 2 tasks for the slate and deletes mirrored tasks** for anything no longer on `This Week` (keeps Focus uncluttered). Present the finalized bulleted slate; it must match the Notion `This Week` filter exactly.
3. **Missing records** — work Aaron describes that is not in Tasks → create a record (with approval) before toggling `This Week`.
4. **Personal mirrors** — Phase 2.2 creates Todoist tasks for selected Personal items (case-by-case approval); verify last week's mirrors via Todoist MCP.

**Gate rules:**
- Before **Phase 2 (Work)**: Phase 1 FIELD CHECK (`1.check`) must pass.
- Before **Phase 4 (Commit)**: Phase 2 development block complete (through `2.check`).
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
4. **If entry exists and Session Complete = Complete:** Hold for Phase 2.1. Continue to **Phase 0a** (confirm weeks), then Phase 0.

<a id="phase-0a-confirm-review-planning-weeks-1-min"></a>
## Phase 0a: Confirm Review + Planning Weeks (~1 min)

**Purpose:** Lock which Week Tracker rows this session **reviews** vs **plans for** — before any data pull. Aaron's default workflow (Fri–Mon):

| Session day | Review week | Planning week | Turnover state |
|-------------|-------------|---------------|----------------|
| **Friday / Saturday** | Current calendar week (still in) | **Next** Week Tracker row | **Pre-turnover** — review week is still `Is Current Week` |
| **Sunday / Monday** | Prior Week Tracker row | Current week (entering) | **Post-turnover** — auto-defer already ran |

**Turnover = the `WeekDefer` automation (Sun ~3 AM CT).** At the week flip it moves every open Task still linked to the ending week onto the new current week (`scripts/defer-week-tasks.mjs`; Done tasks stay put as history). This drives the pre/post-turnover branch in Phase 2.1-F carryover:

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

Also run `weekly-habit-summary.mjs` (logged/unlogged accomplishments). Todoist MCP in Phase 2.2 for Personal mirror completion check. Ops pulls → **Weekly Ops** `scripts/weekly-ops-pull.mjs`.
4. **Habit source DBs** (past 7 days — habit summary script is canonical; MCP only if script missing):
   - Workouts (`127f40c2-487b-80ba`): query all, count by Type
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

**Purpose:** Values context first, then mind → fitness → sleep → social → parenting → personal enjoyment — one domain at a time (review → rate health where applicable → set intentions). Mind includes wellness screening. Work health rates in Phase 2.2.

> **Intentions vs targets — keep them separate (do not restate numbers).** Every recurring **numeric goal** lives in a **structured target field** that drives the dashboard widgets: `Strength Target`, `Cardio Target` (fitness) · `Sleep Target Hours`, `Target Wake Time` (sleep) · `Calorie Target`, `Weight Goal Direction` (nutrition) · `Social Target` (social small-talk/sarges) · `Workshop Hours Intended` (workshop). The `*Intentions` rich_text fields (`Mind/Fitness/Sleep/Social/Parenting Intentions`, `Week Intentions`) capture **only qualitative changes** for the week — behavioral shifts, experiments, focus themes, one-off adjustments. **Never** write "5 strength workouts" or "3–5 sarges" into an intention field; that number goes in its target field. If a domain has no qualitative change this week, **leave its intention blank** (better empty than restating a target). This keeps the weekly printout + dashboard clean and non-redundant.

**Phase 1 order (session):** `1.0` → `1.1` Values → `1.2` Mind (incl. wellness) → `1.3` Fitness → `1.4` Sleep and Schedule → `1.5` Social → `1.6` Parenting → `1.7` Personal enjoyment → `1.check`

**Print / Week Tracker domain order (Phase 4b):** Sleep and Schedule → Spirituality & Mind → Fitness → Social → Parenting → Personal Enjoyment → Development Work. Each domain renders as **three parts**: *What happened last week* · **Targets for next week** (the structured numbers — Strength/Cardio, Sleep hours + wake time, Calories + direction, Social target, Workshop hours — rendered as a compact line/badges) · *Intentions for next week* (**qualitative only** — behavioral changes / experiments; **omit the line entirely when blank**, don't pad with restated targets). Five-level health badge when rated; **trend arrow** (↑ / ↓ / −) vs the **prior week's** same-domain rating. This keeps the printout's intentions section clean and meaningful (targets are shown once, as numbers, not repeated as prose).

**Health rating scale** *(all Phase 1 domain ratings + Phase 2.1 dev health — one letter per turn)*

| | Rating |
|---|--------|
| **A** | Very Unhealthy |
| **B** | Unhealthy |
| **C** | Okay |
| **D** | Healthy |
| **E** | Very Healthy |

**Gates:** Starved Values + behavioral adjustments (1.4-G) trigger when rating is **A or B** (Unhealthy or worse). Okay and above: no starve, no behavioral-adjustment table for that domain.

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

**Table 1.2-G — Mind health** *(Aaron confirms; write with approval)*

| Rating | Evidence (mood + spirit + journal) | → Notion field |
|--------|-----------------------------------|----------------|
| A–E (health scale) | cite valence, negative %, spirit min | `Mind Health` |

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

**Table 1.3-C — Fitness daily** *(from `daily-health-sections.mjs` FITNESS section — **Workouts DB** `127f40c2-487b-80ba`; **Supplements** from same script / `weekly-habits-*.md` TABLE 1.3-C-supp)*

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

**Table 1.3-E — Fitness health**

| Rating | → Notion field |
|--------|----------------|
| A–E (health scale) | `Fitness Health` |

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

**Table 1.4-E — Sleep health**

| Rating | → Notion field |
|--------|----------------|
| A–E (health scale) | `Sleep Health` |

**Table 1.4-F — Sleep and schedule intentions (upcoming week)**

| Sleep intentions (1–3 bullets) | Schedule intentions (1–3 bullets) | Sleep target (h) | Wake target (CT) | → Notion field(s) |
|--------------------------------|-----------------------------------|------------------|------------------|-------------------|
| | | | | `Sleep Intentions`, `Schedule Intentions`, `Sleep Target Hours`, `Target Wake Time` |

**Table 1.4-G — Behavioral adjustments** *(required when **this step's** domain = A or B — concrete commitments; **skip table** (write "—") when C or better)*

| Adjustment | Reason |
|------------|--------|
| | |

→ Write `Behavioral Adjustments` (append domain-labeled bullets; cumulative across A/B domains this session). **C+ domains: no adjustments needed.**

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

**Table 1.5-F — Social health**

| Rating | → Notion field |
|--------|----------------|
| A–E (health scale) | `Social Health` |

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

**Table 1.6-D — Parenting health**

| Rating | → Notion field |
|--------|----------------|
| A–E (health scale) | `Parenting Health` |

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

**Table 1.7-C — Enjoyment health**

| Rating | → Notion field |
|--------|----------------|
| A–E (health scale) | `Enjoyment Health` |

→ Write `Personal Enjoyment` (rich_text — last week + forward intention). Propose Personal Time Blocks calendar events with approval.

Sync Notion, then **print preview:** `--section enjoyment` — present verbatim; Aaron confirms → advance.

**FIELD CHECK — Phase 1** *(Table 1.check)*

| Group | Required Notion fields |
|-------|------------------------|
| Last-week KPIs | `Strength Sessions`, `Cardio Sessions`, `Spirit Minutes`, `Journal Count`, `Weight Avg`, `Body Fat Avg`, `Lean Mass Avg`, `Sleep Avg`, `Sleep Nights Tracked`, `Wake Time Std Dev Min`, `Bedtime Std Dev Min`, `Sleep Schedule Rating`, `Steps Avg`, `Workout Active Minutes` |
| Week theme | `Week Intentions` — 1–3 sentence summary of the week's overarching theme (written/confirmed in Phase 4) |
| 1.2 | `Intentions Review` (mind row), `Mind Health`, `Mind Intentions`, `Mood Valence`, `Mood Negative %`, `Journal Feelings Summary`, `Mood Distress Flag`, `Energy Rating`, `Screening Escalation`; PHQ/GAD only if `Screening Escalation` = true |
| 1.3–1.4 | `Fitness/Sleep Health`, `Fitness/Sleep Intentions`, `Schedule Intentions`, `Schedule Review`, `Strength Target`, `Cardio Target`, `Calorie Target`, `Weight Goal Direction`, `Calorie Rationale`, `Sleep Target Hours`, `Target Wake Time`, `Behavioral Adjustments` |
| 1.5 | `Small Talk Count`, `Social Events Count`, `Social Review` (incl. **fuel rating**), `Social Intentions Met`, `Social Health`, `Social Priority`, `Social Target` (number), `Social Intentions` (qualitative, optional) |
| `1.4` (A or B) | `Behavioral Adjustments` — **required for that domain**; skip when C or better |
| `1.5` | Fuel **Stage 1 + 2** + **recovery intentions** (1.5-E-b when triggered) in `Social Review` |
| 1.6 | `Parenting Health`, `Parenting Intentions` |
| 1.7 | `Personal Enjoyment`, `Enjoyment Health` |

**Do not proceed to Phase 2 (Work) until Table 1.check passes.**

<a id="phase-2-development-domain-first-18-min"></a>
## Phase 2: Development (domain-first, ~18 min)

**Purpose:** Review and plan dev work **one domain at a time** — **Turbo Gear → Systems → Chrome Lot** — surfacing the **strategy layer** (active **Goals** + their **milestones**, and **standalone Projects**), not just the Task tracker. For each dev domain: *review last week → set a weekly time goal → select the Goals / milestones / projects to get done this week.* Then a lighter **Workshop + Admin** tail, one **overall dev-health** rating, and a **single combined slate sync**.

**Step codes (ledger order):** `2.TG` Turbo Gear → `2.SY` Systems → `2.CL` Chrome Lot → `2.H` dev health → `2.WA` Workshop + Admin → `2.sync` commit slate → `2.check`. Each dev-domain ledger step spans three turns: **`.1` Review · `.2` Time goal · `.3` Select work** (advance the ledger once, after `.3`).

**Source files:** `output/weekly-dev-review-*.md` — now includes, per dev domain, a **`## Domain goals & projects — {domain}`** block (active Goals + their milestones + standalone Projects) alongside the existing review-week queue / time / carryover sections — plus `output/weekly-habits-*.md` and `node scripts/scan-tg-backlog.mjs` (TG orphan backlog).

**Layer model — read `context/systems/horizon-roadmap.md`.** **Goal** = finish-line outcome (the *why*). **Milestone** = a Projects row linked to a Goal (`🥅 Goals` set) — a stop on that goal's trail. **Standalone project** = a Projects row with no Goal. **Task** = the execution unit — the *only* layer that carries `📅 Week Tracker` (the weekly slate), Toggl, and time. **Selecting a milestone / standalone project for the week means promoting it to its Task tree and putting those Tasks on This Week** (`node scripts/promote-roadmap-to-dev-project.mjs --page=<projectId>` or the ▶ Start webhook); if it already has an open Task tree, just queue those Tasks.

**Selection accumulates across domains — sync ONCE.** Each `.3` records the resulting **Task IDs** into a running slate (ledger `notes.dev_slate_ids`). The actual `📅 Week Tracker` write happens **only** at `2.sync`, with the cumulative set — because `sync-dev-projects-this-week.mjs` **clears** the week relation on every open Task *not* passed in `--selected` (full-DB sweep). **Never** run the sync mid-loop with a single domain's IDs (it would clobber the other domains' picks).

**Presentation:** group by domain; nest sub-items under parents; letter each selectable root.

---

<a id="21-dev-review-8-min"></a>
### 2.1 Dev Review (~8 min)

#### A — Last week review (one table group per turn)

**Table 2.1-A — Queued dev work (week under review)**

*The queue row is the main payload — CL/TG Tasks with `This Week` checked during the week being reviewed (Mon–Sun before this session). That tracker is canonical; the log snapshot is a cross-check only.*

| Field | Value | Source |
|-------|-------|--------|
| Work health (dev efforts) | | Prior Weekly Log `Work Health` |
| Adjustments trying | | Prior `Dev Adjustments` |
| **Queued dev work** | Parents + sub-items, grouped by domain (CL, TG, Personal) | **Primary:** `weekly-dev-review` § Review week — queued dev work (all types). **Cross-check:** prior log `Dev Projects Intended` when populated |

**Table 2.1-B — Time logged** *(two tiers — dev first, then all logged)*

*Source: Time Punches (`Total Time` formula). **Reading** → Phase **1.2** Mind (TABLE 1.2-B-spirit), not here.*

**Tier 1 — Dev time** *(CL Dev + TG Dev + Admin; excludes Ops/Field)*

| Day | CL Dev | TG Dev | Admin | Dev total |
|-----|--------|--------|-------|-----------|
| | | | | |
| **Total** | | | | |

→ Write `Deep Work Minutes` = **dev total** (CL Dev + TG Dev + Admin only).

**Tier 2 — All logged time** *(dev + Ops + Field)*

| Day | Dev | Ops | Field | All logged |
|-----|-----|-----|-------|------------|
| | | | | |
| **Total** | | | | |

*Present Tier 1 first in 2.1-D narrative; Tier 2 for full work-week picture.*

**Table 2.1-B² — Activity summaries** *(what shipped during logged time)*

*Source: `weekly-dev-review` § Table 2.1-B² — auto-generated on each Time Punch from GitHub commits + Cursor cloud agents in that window. Only punches with detected activity have text; use when narrating what shipped in 2.1-D / 2.1-F.*

| Date | Category | Min | Title | Summary |
|------|----------|-----|-------|---------|
| | | | | |

**Table 2.1-C — What was accomplished** *(legacy ID — content now presented as **2.1-F Turn 1**; keep field writes here)*

**From plan** — *same tree as **2.1-A** across **Chrome Lot · Turbo Gear · Systems · Workshop · Admin**. Do **not** flatten to a table. Mark finished items with ~~strikethrough~~ (Status = Done). Open items stay plain.*

```
**Chrome Lot**
- **Parent** — Status
  - Sub-item — Status
**Turbo Gear**
- ...
```

**Off-queue logged** *(optional — only if `weekly-habits` § Tasks completed shows real wins with `This Week` checked; never infer from `last_edited` alone)*

**Detected unlogged** *(from `weekly-habits` sweep — includes **Media stack / Audiobookshelf cluster** for ABS/NAS/Plex incident work often shipped via Cursor without a Task; Aaron flags count + hours)*

| Highlight | Count as shipped? |
|-----------|-------------------|
| | Aaron: yes / no |

→ Write `Accomplishments`, `Logged/Unlogged/Total Accomplishments Count`, `Focused Output Hours Estimate`, `Dev Review`.

#### B — Plan for next week (one table per turn)

**Table 2.1-D — Dev health review** *(agent summarizes; Aaron rates)*

Agent presents a short narrative — **no recommendations, no task list** (selection is **2.1-G**). Cover:

1. **Output** — accomplishments vs queued dev work (2.1-C counts + queue tree).
2. **Time** — dev minutes logged (2.1-B) vs realistic capacity.
3. **Goal progress** — review week vs **planning month** Tasks (`weekly-dev-review` § Planning month — CL/TG/Personal trees via `🌙 Month`): what moved, what’s stuck, parked domains noted.

Then Aaron rates:

| | Rating |
|---|--------|
| **A** | Very Unhealthy |
| **B** | Unhealthy |
| **C** | Okay |
| **D** | Healthy |
| **E** | Very Healthy |

→ Write `Work Health`, `Dev Week Rating`, `Dev Intentions Met`, and fold the summary into `Dev Review`.

**Table 2.1-E — Adjustments** *(only if 2.1-D = A or B)*

**Skip entirely when C or better.** When A or B: ask Aaron what adjustments he wants to commit to for the next period. **Agent does not recommend adjustments** — capture his words only.

→ Write `Dev Adjustments` (Aaron’s commitments, or leave empty when C+).

**Planning next week — four turns (do not combine)**

**Table 2.1-F — Review week close-out** *(all Task types: Chrome Lot · Turbo Gear · Personal)*

*Personal Tasks log as **Admin** time in Time Punches. Present all three types together — not CL/TG only.*

**Turn 1 — What was accomplished** *(was 2.1-C content; now first in F)*

Same tree as review-week queue (`weekly-dev-review` § Review week — queued dev work). Group **Chrome Lot → Turbo Gear → Systems → Workshop → Admin** (deep work = CL/TG/Systems; Workshop capped; Admin standing). **Always show sub-items** (full parent → child nesting). ~~Strikethrough~~ on Done. *Skip off-queue agent/infra tuning during the session unless Aaron flags it.*

→ Fold into `Accomplishments`, accomplishment counts, `Focused Output Hours Estimate`.

**Turn 2 — What's still open**

Re-present **only open items** from the same tree (full parent → sub nesting; letter each root). Cross-check prior log `Dev Projects Intended` when Week Tracker link is empty.

| | |
|---|---|
| **Reply** | Letters of items to **mark Done** now (or **none**) |

Per-item Notion `Status → Done` only after Aaron names the letter(s). Re-fetch tree before Turn 3.

**Turn 3 — Carryover decision** *(branch on turnover state from Phase 0a)*

**Pre-turnover (Fri/Sat — review week still current):** split open review-week items into finish-this-week vs defer.

| | |
|---|---|
| **A** | Keep finishing this week — item stays linked to the **current/review** week (WeekDefer sweeps it to the new week Sunday if still open) |
| **B** | Defer now to the **planning/next** week — reply with letters to move onto the planning-week slate |

*"Keep finishing" items are simply not added to the planning-week slate; leave their `📅 Week Tracker` on the current week. The Sun 3 AM WeekDefer moves whatever is still open forward automatically.*

**Post-turnover (Sun/Mon — auto-defer already ran):** every unfinished Task is already on the current (= planning) week. Do **not** ask about finishing.

| | |
|---|---|
| **A** | Continue all **remaining** open items on this week's plate |
| **B** | Prune some — reply with letters to **drop** from the plate (clears `📅 Week Tracker`, i.e. back to backlog) |

Dropped/pruned items: week relation cleared at sync (approval). Carried items stay selected for sync.

**Turn 4 — Table 2.1-G — Add from Projects?**

| | |
|---|---|
| **A** | Yes — show current-quarter open items (**2.1-H**) |
| **B** | No — carryover only |

**Table 2.1-H — Project adds** *(only if 2.1-G = Yes)*

Source: `weekly-dev-review` § **Planning month — {domain}** (`🌙 Month` → planning month). Skip **parked** domains. Exclude items already on carryover. Letter each row.

**Empty Task records** (no `Name` title): archive in Notion when detected — do not present.

**Table 2.1-J — Turbo Gear backlog (Pass 2 — bottom-up)** *(after Projects/Tasks are committed above; TG dev block only)*

Passes 2.1-F→H are **top-down** (Projects + their Tasks). This is **bottom-up**: the standalone TG backlog (bugs / small features / optimizations with **no Project**). Run `node scripts/scan-tg-backlog.mjs` — it lists un-queued TG orphan roots **grouped by Priority (High → Medium → Low → Unset), oldest-first**, with a health line (counts + oldest-High age). *(Aaron's browsing view: [Turbo Gear Backlog](https://notion.so/39bf40c2487b81f9a232d2ba0f1ab8e6).)*

Present **capacity-gated**: after Pass-1 TG commitments, estimate the **TG-dev hours left** in the sprint and surface only enough backlog to fill it — default cap **~3–5 items**, High/Medium first (Low only if capacity clearly allows). Letter each surfaced row. Flag any **High aging past ~14d** as "must address."

| | |
|---|---|
| **A** | None this week — backlog stays as-is |
| **(letters)** | Add selected orphan tasks to the sprint |

Selected orphan IDs join the **Sync** `--selected` set below (they get `📅 Week Tracker` = planning week like any pick). If a High item is aging and repeatedly skipped, treat it like a 3×-deferred task (delegation/convert-to-Project or cut). Record the backlog health line in `Dev Priority Context`.

**Sync** *(after F + G/H + J confirmed — **run immediately**, approval before Notion writes)*

Execute `node scripts/sync-dev-projects-this-week.mjs --selected=<comma-separated page IDs>`:

1. **Link** `📅 Week Tracker` = **planning week** on carryover + roadmap/project picks (2.1-H) + TG backlog picks (2.1-J).
2. **Clear** `📅 Week Tracker` on **every other open** CL/TG Task (full open-database sweep for CL/TG types → back to backlog).
3. Clear `📅 Week Tracker` on any **Done** records still linked to the planning week (stale bulk-close artifacts).
4. **Toggl tasks** — same script chains `sync-dev-projects-toggl-tasks.mjs`: create/assign tasks for selected items; **delete** mirrored Toggl tasks (and clear `Toggl Task ID` on Tasks) for anything not on the slate.

*Pre-turnover: "keep finishing" items keep their current-week link untouched — the sync only manages the **planning-week** slate. The slate view = Tasks whose `📅 Week Tracker` = current week (`Is Current Week` rollup).*

→ Write `Dev Projects Intended` (snapshot), `Dev Priority Context`.

**Present Table 2.1-S — This Week slate** *(CL + TG + Personal; must match the `Is Current Week` / planning-week filter exactly)* — Aaron executes this slate from the [This Week — Dev Slate](https://notion.so/39bf40c2487b81d2a7acf44e0706775f) view (Start Timer · Logged Minutes This Week · TWC).

Output of sync script — all domains. Aaron confirms before **2.2** (Todoist mirrors only if Personal items selected).

Sync Notion (`Dev Projects Intended`, `Dev Intentions`, `Work Health`, etc.), then **print preview:** `--section development` — present verbatim; Aaron confirms before **2.2** (re-run after **2.2** if personal dev slate changes the print tree materially).

---

<a id="22-systems-workshop-admin-review-7-min"></a>
### 2.2 Systems / Workshop / Admin Review (~7 min)

*Carryover and week selection run in **2.1-F** (all domains). Phase 2.2 = Todoist mirror check + block scheduling for the non-CL/TG domains + any gaps not covered in 2.1-F/G.*

**Domain → block mapping (single-axis labels, see `capacity-rules.md`):**
- **Systems** = deep work — joins the **One Thing / deep-work block** alongside Chrome Lot + Turbo Gear. Counts toward the dev goal.
- **Workshop** = QoL/hobby (home automation, Plex, dashboards) — schedule a **separate Workshop block, capped ~3 hr/wk**. Never in the deep-work block. Does not count as dev.
- **Admin** = personal-life/legal (custody, will, house) — schedule a **separate Admin block, outside the deep-work block**. Not dev.

**Required:** the plan must place a distinct **Admin block** and a distinct **Workshop block** outside the deep-work block. If Workshop candidates exceed the ~3 hr cap, defer the remainder — do not let tinkering crowd out deep work.

#### A — Context + last week

**Table 2.2-A — Planning month Systems / Workshop / Admin Tasks**

Present `weekly-dev-review` § **Planning month** for the non-CL/TG domains (`🌙 Month` → planning month), **grouped by domain: Systems / Workshop / Admin**. Full parent → sub-bullet tree. **No free-text priority themes** — monthly plan links records in Phase 11b / Phase 5. Flag which block each root feeds (Systems → deep work; Workshop → capped block; Admin → admin block).

**Table 2.2-B — Last week personal plan**

**Bulleted tree only** — same nesting as **2.2-A** / **2.1-C**. Do **not** flatten to a table. One parent block per root; indent children. Append mirror + outcome on each line.

```
**Systems**
- **Parent** — Status · mirror: none / open / completed · done: ✓ / ✗
  - **Child** — Status · mirror: … · done: …
**Workshop**
- …
**Admin**
- …
```

Source: prior `Dev Projects Intended` field (Systems/Workshop/Admin) + prior-week `This Week` carryover for those domains + **Todoist MCP** (tasks due/completed last week). Confirm completions with Aaron.

#### B — Plan for next week

**Table 2.2-C — Planning month adds** *(only if 2.2-B needs items not on carryover)*

Same source as **2.2-A** — letter rows to add to `This Week` that aren’t already on the plate from **2.1-F** carryover (Personal domain).

**Table 2.2-D — Selection** *(multi-select — reply with letters, or **A** = all / **B** = none)*

Present as a **lettered bulleted tree** (same nesting as **2.2-A**). Letter each **root**; children inherit parent selection unless Aaron splits them explicitly.

After selection — **run immediately** (approval before Notion writes):

Execute `node scripts/sync-dev-projects-this-week.mjs --selected=<all CL/TG IDs from 2.1-S plus selected Personal IDs>`:

1. `This Week = true` on selected Personal Tasks only.
2. `This Week = false` on **every other open** Personal Task.
3. Re-run with **combined** CL/TG + Personal selected IDs so the full slate is authoritative (one sync pass).
4. **Toggl tasks** — same script chains `sync-dev-projects-toggl-tasks.mjs`: create/assign Focus tasks for selected items; **delete** off-slate mirrors (and clear `Toggl Task ID`).
5. For each selected item: propose **Todoist mirror** (due date + project) — **case-by-case approval** before create.

#### W — Workshop time intention *(step `2.2-W` — REQUIRED every week)*

Workshop (QoL/hobby) is time-boxed so tinkering never crowds out deep work. Set the budget **and** aim it at specific item(s).

1. **Retrospective:** show **last week's Workshop actual** (Week Tracker `Workshop` rollup / `weekly-dev-review` § non-dev logged Workshop minutes) vs the prior week's **`Workshop Hours Intended`**. One line: `Workshop last week: actual Xh vs intended Yh`.
2. **Table 2.2-W — Workshop plan** — present open **Domain = Workshop** items (lettered tree from 2.2-A Workshop group). Aaron replies with:
   - **hours** for `Workshop Hours Intended` (cap **~3 hr**; **0 = skip Workshop this week**), and
   - **letter(s)** for the item(s) that time goes to.
3. **Cap guard:** if the intention exceeds ~3 hr, confirm it's a deliberate trade against deep work before accepting. If 0, no Workshop items go on the slate this week.
4. Selected Workshop items are included in the **2.2-D** `This Week` sync (they carry Domain = Workshop). Write `Workshop Hours Intended` (number) + `Workshop Focus` (item names + one-line intention) to the Weekly Meeting Log at **Phase 4 commit**.

#### H — Systems & Workshop health *(step `2.2-H` — quick, every week)*

Rounds out the life-category ratings so **all** categories are trended (their `* Score` fields feed the annual/monthly/quarterly trend review). One rating per turn, five-level scale:

1. **Systems:** quick check — are your systems (automations, LCC, home/dev infra) running well? Any breakage or friction? Recommend 0–1 concrete change (→ a Systems Task/Project if warranted). **Rate `Systems Health`.**
2. **Workshop:** given this week's Workshop intention/actual, **rate `Workshop Health`** (is the hobby/QoL tinkering in a good place — neither crowded out nor crowding out deep work?).

Write `Systems Health` + `Workshop Health` (select) to the Weekly Meeting Log at Phase 4 commit; the `Systems Score` / `Workshop Score` formulas populate automatically for trending.

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
| `Deep Work Minutes`, `Accomplishments`, counts | 2.1-C |
| `Dev Review`, `Work Health`, `Dev Week Rating`, `Dev Intentions Met` | 2.1-D |
| `Dev Adjustments` | 2.1-E *(A/B only; Aaron-supplied)* |
| `Dev Projects Intended` | 2.1 sync (after F/G/H) |
| `Dev Priority Context` | 2.1-G |
| CL/TG `This Week` sync | 2.1-S |
| Personal `This Week` sync | 2.2-D |
| `Workshop Hours Intended`, `Workshop Focus` | 2.2-W |
| `Systems Health`, `Workshop Health` | 2.2-H |
| **Final slate = Notion view** | **2.check** |

**Do not proceed to Phase 4 until `2.check` passes.**

<a id="phase-4-commit-5-min"></a>
## Phase 4: Commit (~5 min)

**Purpose:** Final review, capacity check, execute remaining actions, log everything.

1. **Summary table:** Everything planned across all phases (project selections, personal items, dev intentions)
2. **Final capacity check:** Total planned hours vs. available hours. If total exceeds available, something must move. This is non-negotiable.
3. **Confirm "This Week" checkboxes:** Verify all selected Tasks have `This Week = true` and no deselected ones still have it checked.
4. **Store project KPIs on Weekly Meeting Log:** Write `Projects Completed` (count of projects marked Done this week) and `Projects In Progress` (count of projects with This Week checked for the new week).
5. **Verify all FIELD CHECKs (REQUIRED):** Re-run Phase 1 (`1.check`) and Phase 2 (`2.check`). Confirm nothing is blank without N/A + reason. (Activity KPIs + Team Activity Details → **Weekly Ops** commit.)
6. **Write / confirm `Week Intentions` (REQUIRED):** 1–3 sentence week theme capturing the overarching focus for the planning week. Agent proposes from session context; Aaron confirms or edits → write to Weekly Meeting Log. `node scripts/weekly-plan-log-check.mjs commit --ledger <path>` must pass before `workflow-notion-log complete`.
7. **Record life health ratings (REQUIRED):** Verify weekly-rated selects are set — `Mind Health` (→ `Spirituality Health` in Phase 4), `Fitness Health` (1.3), `Sleep Health` (1.4), `Social Health` (1.5), `Parenting Health` (1.6), `Enjoyment Health` (1.7), `Work Health` (2.2). Values: **Very Unhealthy → Very Healthy** (five-level scale). Admin is not rated in weekly plan.
8. **Update Values DB Health (with approval):** For each category where this week's rating differs from current Values DB Health, update via `personal_notion_update_page` on the category page in Values DB (`342f40c2-487b-80c5`). Include **Personal Enjoyment** when added to Values DB.
9. **Record Starved Values:** Derive from weekly health ratings — set `Starved Values` multi_select to every **rated** category marked **Unhealthy or Very Unhealthy** (A or B: Spirituality, Fitness, Work, Social, Parenting, Personal Enjoyment). Admin excluded from weekly rating.
10. **Confirm accomplishment fields (REQUIRED):** Verify Phase 2.2 wrote `Logged/Unlogged/Total Accomplishments Count`, `Focused Output Hours Estimate`, and `Accomplishments`. Backfill from habit summary if missing.
10b. **Write Workshop intention (REQUIRED):** From Phase 2.2-W, write `Workshop Hours Intended` (number, 0 allowed) + `Workshop Focus` (rich_text — selected item(s) + one-line intention) to the Weekly Meeting Log. `weekly-plan-log-check` commit gate requires `Workshop Hours Intended`.
11. **Body comp already persisted.** Withings written in Phase 0 (`--days 28`). Don't re-run here.
12. **Execute remaining:** Create any Todoist/Calendar/Notion items not yet committed during earlier phases.
13. **Log to Notion:** Finalize the Weekly Meeting Log entry (`322f40c2-487b-81bd`) with key decisions, action items, and plan summary. Set `Status = Done`, `Session Complete = Complete`.
14. **Personal Time Blocks (`4.tb` — REQUIRED after scheduling, before 4b):**

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

15. **Week Tracker summary (4b — REQUIRED):**
    - Planning week was confirmed at **0a** (`planning_week_page_id` on ledger). Optional gate — `node scripts/weekly-plan-section-preview.mjs --ledger <path> --all` (full print preview).
    - After Aaron confirms, run `node scripts/weekly-plan-week-summary.mjs --ledger <path>` (uses ledger planning week — no end-of-session week picker). This (1) renders a **print PDF** via Playwright (also saves `output/weekly-plan-print-{week}.html` + `.pdf`), (2) uploads PDF to `Plan Records/weekly/`, (3) sets **`Plan Doc URL`** on the planning week record, (4) appends/replaces the **Weekly Plan** section on that Notion page. Aaron approves production writes.
16. **Update context files** if anything changed.

<a id="cross-cutting-rules"></a>
## Cross-Cutting Rules

- **Table contract per phase.** Phase 1 = life domains → `1.check`. Phase 2 = `2.1` Dev Review (CL/TG) → `2.2` Personal Projects → `2.check`. CL ops → **Weekly Ops** skill.
- **FIELD CHECK gates.** Run `1.check` before Phase 2 development; `2.check` after development; verify all in Phase 4 commit.
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
- **Phase 0:** Wellness + journal feelings + social + dev data pulls; trend files; 4-week log history.
- **Phase 0b:** Data integrity table; remediation before Phase 1.
- **Phase 1:** Life review (values, mind, fitness, sleep, social, parenting, personal enjoyment) + targets on Weekly Meeting Log.
- **Phase 2:** Development review + next-week dev plan.
- **Phase 4:** Full Weekly Meeting Log finalized + all FIELD CHECKs; Values DB sync (with approval).
- **Phase 4.tb** — Personal Time Blocks calendar — one-time Mon–Fri structure events; adjust-if-exists or full regenerate with approval. Skill → `context/skills/plan-weekly-schedule/SKILL.md`.
- **Phase 4b:** Planning week record (from 0a) gets domain-by-domain plan summary + linked Google Doc in `Plan Records/weekly/`.

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
- `../weekly-ops/SKILL.md` — CL operations (separate session)
- `../../work/turbo-gear/overview.md`
