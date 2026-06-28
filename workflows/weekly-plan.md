> **Source:** [`context/skills/weekly-planning/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-planning/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Weekly Planning — SKILL

## Table of contents

- [Trigger](#trigger)
- [Inputs](#inputs)
- [Execution Protocol (mandatory — read `context/workflow-execution.md` + `context/systems/workflow-output-contracts.md`)](#execution-protocol-mandatory-read-contextworkflow-executionmd-contextsystemsworkflow-output-contractsmd)
- [Interaction Style](#interaction-style)
- [Required Notion fields — index](#required-notion-fields-index)
- [Procedure](#procedure)
- [Pre-Phase 0: Monthly Plan Gate (mandatory — runs before everything else)](#pre-phase-0-monthly-plan-gate-mandatory-runs-before-everything-else)
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
  - [1.4 Sleep — Review · Rate · Intentions (~5 min)](#14-sleep-review-rate-intentions-5-min)
  - [1.5 Social — Review · Rate · Intentions (~5 min)](#15-social-review-rate-intentions-5-min)
  - [1.6 Parenting — Review · Rate · Intentions (~4 min)](#16-parenting-review-rate-intentions-4-min)
  - [1.7 Personal Enjoyment (~2 min)](#17-personal-enjoyment-2-min)
- [Phase 2: Development (~15 min)](#phase-2-development-15-min)
  - [2.1 Dev Review (~8 min)](#21-dev-review-8-min)
  - [2.2 Personal Project Review (~7 min)](#22-personal-project-review-7-min)
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
- `context/systems/notion-databases.md` — every DB ID referenced below (Dev Projects, Weekly Meeting Log, source DBs, Values, Workouts, etc.)
- `context/systems/knack-fields.md` — Customer + Photographer field references
- `context/systems/hubstaff.md` — member IDs, weekly-report tool
- `context/systems/health-data.md` — MCP architecture, expected fields
- `context/self/values.md` — six categories and current Health statuses (Phase 1.1 context; per-domain ratings in Phase 1 + 2.2)
- `context/self/eros.md` — primary fuel doctrine (Phase 1.5 fuel check)
- `context/self/social.md` — sarges, Small Talk targets, isolation signals (Phase 1.5)
- **Dev state (canonical):** **Dev Projects** DB (`341f40c2-487b-80ac`) — Phase 2 reads the tracker first; meeting logs record what was planned and accomplished. `This Week` is set only from Aaron's explicit selection each session.
- `context/people/index.md` — delegation matrix, 1:1 tracking
- `context/work/turbo-gear/overview.md` — TG strategic sequence (for Phase 2.4 project selection)

<a id="execution-protocol-mandatory-read-contextworkflow-executionmd-contextsystemsworkflow-output-contractsmd"></a>
## Execution Protocol (mandatory — read `context/workflow-execution.md` + `context/systems/workflow-output-contracts.md`)

> **This skill's tables are the spec.** Fixed headers, named sources, one table per turn where noted. Weekly plan is the reference implementation for all workflows.

1. **Date context (every turn, before any weekday or "this week" language):**
   ```
   node scripts/planning-dates.mjs [--today=YYYY-MM-DD] [--week-of=<ledger week_of>]
   ```
   Aaron may run weekly plan on **any day** — never assume session day = Monday. Use script output for review week, planning week, and weekday→ISO mapping. `Today's date` from `user_info` must match `--today` (CT).
2. **Init ledger** at session start: `node scripts/workflow-progress.mjs init --workflow weekly-plan` — omit `--week-of` to auto-set canonical planning Monday from today; if set manually, must be a Monday and should match `planning-dates.mjs`. Weekly log title = `Week of <planning Monday>`.
3. **Notion log** — at step `1.0`: `node scripts/workflow-notion-log.mjs create --ledger <path>`. After every `advance`: `workflow-notion-log sync`. Write phase fields per `context/systems/workflow-logs.md`. On commit: `workflow-notion-log complete`.
4. **Every turn:** `node scripts/workflow-progress.mjs status --workflow weekly-plan` — present only `current_step` (status includes date warnings if ledger `week_of` is wrong)
5. **Phase banner** on every user-facing message: `**[Weekly Plan · Phase X.Y — title]**`
6. **One sub-step per turn** — never bundle 1.2 + 1.3 + 1.4 + 1.5 + 1.6 + 1.7 (each life domain is a separate step)
7. **Table contract is the spec** — each sub-step lists the **exact tables** to present (column headers fixed). Fill every cell from the named data source; use `—` when data is missing. Do not add metrics, sections, or discussion topics outside that step's tables. **Do not `advance` until every in-scope table for the step is presented and any required Aaron input is collected.**
8. **One question per turn** — Energy rating and Mind intentions are separate turns; PHQ-2/GAD-2 only when `Screening Escalation` is true (one item per turn)
9. **Advance after complete:** `node scripts/workflow-progress.mjs advance --workflow weekly-plan --step <id>`
10. **Phase gates:** `node scripts/workflow-progress.mjs gate --workflow weekly-plan --phase <1|2>` before Phase 2 (work) or Phase 4 (commit)
11. **Tangents:** fix/interrupt, then resume ledger `current_step` — do not skip ahead

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
1. **Tracker first** — monthly incomplete lists come from Dev Projects (Notion), not log prose.
2. **Selection sync** — after each Phase 2 sub-step (2.1 then 2.2), run `scripts/sync-dev-projects-this-week.mjs` with the cumulative selected page IDs: set `This Week = true` on selected only; **`This Week = false` on every other open Dev Project** (domain-scoped within that phase, then combined pass at 2.check). Present the finalized bulleted slate; it must match the Notion `This Week` filter exactly.
3. **Missing records** — work Aaron describes that is not in Dev Projects → create a record (with approval) before toggling `This Week`.
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
3. **If no matching entry exists:**
   - Tell Aaron: "No Monthly Plan Log for [planning month]. Monthly plan must run before weekly plan."
   - **Pause** this workflow. Run `context/skills/monthly-plan/SKILL.md` end-to-end.
   - After monthly plan Phase 12 commits the planning-month entry, **resume** weekly plan from Phase 0 below.
   - Do **not** offer to skip or proceed weekly-only — monthly plan is a hard prerequisite.
4. **If entry exists:** Hold for Phase 2.1. Continue to Phase 0.

<a id="phase-0-data-pull-silent-before-conversation"></a>
## Phase 0: Data Pull (silent, before conversation)

**Order:** wellness + log trends first, then work pulls. Mind/body phases run before any work discussion.

<a id="wellness-pulls-run-first"></a>
### Wellness pulls (run first)

```
node "scripts/weekly-wellness-trends.mjs"
```
Output: `output/weekly-wellness-trends-YYYY-MM-DD.md`. **Canonical source for 4-week Weekly Meeting Log trends + prior-week intentions + data-integrity report.** Read this before Phase 0b and Phase 1.

```
node "scripts/weekly-habit-summary.mjs"
```
Output: `output/weekly-habits-YYYY-MM-DD.md`. Canonical last-week habit numbers + actionable Dev Projects slate. Do not re-query habit DBs ad-hoc.

```
node "scripts/weekly-journal-feelings.mjs"
```
Output: `output/weekly-journal-feelings-YYYY-MM-DD.md`. **Canonical source for Phase 1.2 mood tables** (valence, tag frequency, daily feelings, escalation, mind insights) **and TABLE 1.2-D-journal** (morning journal gratitude/goals insights). Do not re-query Journal DB ad-hoc.

```
node "scripts/daily-health-sections.mjs"
```
Output: stdout **Mind / Fitness / Sleep** section tables for Phase 1.2–1.4 (days as columns). Run after health pulls.

```
node "scripts/withings-sync.mjs" --days 28 --write
```
```
health_persist_recent({ days: 28 })
```
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
node scripts/weekly-dev-review.mjs
```
Output: `output/weekly-dev-review-YYYY-MM-DD.md` — **canonical Phase 2 source** (prior week plan, dev time by day, monthly incomplete by domain, personal carryover).

Also run `weekly-habit-summary.mjs` (logged/unlogged accomplishments). Todoist MCP in Phase 2.2 for Personal mirror completion check. Ops pulls → **Weekly Ops** `scripts/weekly-ops-pull.mjs`.
4. **Habit source DBs** (past 7 days — habit summary script is canonical; MCP only if script missing):
   - Workouts (`127f40c2-487b-80ba`): query all, count by Type
   - Small Talk (`121f40c2-487b-802d`): query all, count entries
   - Spirit (`2aaf40c2-487b-8070`): query all, sum Total Time
   - Business Development (`127f40c2-487b-803e`): query all, sum Total Time (formula property -- reader must read `properties["Total Time"].formula.number`, not `.number`)
   - Chrome Lot Operations (`136f40c2-487b-80ba`): query all, sum Total Time
   - Field Work (`237f40c2-487b-80ab`): query all, sum Total Time
   - Journal (`99c9e393-812f-4d73`): query all, count entries
5. **Values DB** (`342f40c2-487b-80c5`): Pull all 6 categories with Health status
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
   - Missing watch/sleep → `health_persist_recent({ days: 28 })` then `health_get_summary({ days: 28 })`
   - Missing prior-week log KPIs → backfill from `weekly-habits-*.md` + health MCP into the **prior week's** log entry (with Aaron approval for Notion writes)
5. If a source remains broken after one fix attempt, note it in the table and continue with `--` for affected metrics — but **name the broken pipeline** so it gets fixed outside the meeting.

**Outputs:** Integrity table presented; remediation attempted; known gaps flagged for Phase 1 footers.

<a id="phase-1-life-review-28-min"></a>
## Phase 1: Life Review (~28 min)

**Purpose:** Values context first, then mind → fitness → sleep → social → parenting → personal enjoyment — one domain at a time (review → rate health where applicable → set intentions). Mind includes wellness screening. Work health rates in Phase 2.2.

**Phase 1 order (session):** `1.0` → `1.1` Values → `1.2` Mind (incl. wellness) → `1.3` Fitness → `1.4` Sleep → `1.5` Social → `1.6` Parenting → `1.7` Personal enjoyment → `1.check`

**Print / Week Tracker domain order (Phase 4b):** Sleep and Schedule → Spirituality & Mind → Fitness → Social → Parenting → Personal Enjoyment → Development Work. Each domain renders as a two-column table: *What happened last week · Intentions for next week* (adjustments merged into intentions, bold first), with a Healthy/Unhealthy badge when rated.

<a id="10-create-weekly-log-entry"></a>
### 1.0 Create Weekly Log Entry

Create the **new week's** Weekly Meeting Log entry (Name = `Week of [next Monday YYYY-MM-DD]`, Meeting Date = today). All Phase 1 fields write to this entry.

**Advance ledger:** `1.0` → then present `1.1` only.

<a id="11-values-context-1-min-first-table-to-aaron"></a>
### 1.1 Values Context (~1 min) — **first table to Aaron**

**Purpose:** Orient to the six Values categories before domain deep-dives. Health ratings and intentions are set per-domain in later steps — this step is context only.

**Data source:** Values DB (`342f40c2-487b-80c5`) — pulled in Phase 0.

**Present exactly Table 1.1:**

| Category | Values DB Health | Time Target | 1-line note |
|----------|------------------|-------------|-------------|
| Spirituality | from Values DB | from Values DB | |
| Fitness | from Values DB | from Values DB | |
| Work | from Values DB | from Values DB | |
| Social | from Values DB | from Values DB | |
| Admin | from Values DB | from Values DB | |
| Parenting | from Values DB | from Values DB | |

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
| Spirit Minutes | from `weekly-habits-*.md` | from wellness file | `Spirit Minutes` |
| Journal Count | from `weekly-habits-*.md` | from wellness file | `Journal Count` |
| Morning Journal days | from `weekly-habits-*.md` TABLE 1.2-B-mj | — | (in summary) |
| Affirmations | same as Morning Journal (TABLE 1.2-B-mj) | — | (included in journal ritual) |

**Table 1.2-B-mj — Morning journal aggregate** *(copy from `weekly-habits-*.md` section `TABLE 1.2-B-mj`)*

**Table 1.2-C-mj — Morning journal daily** *(copy from `weekly-habits-*.md` section `TABLE 1.2-C-mj`)*

**Table 1.2-C — Mind daily** *(from `daily-health-sections.mjs` MIND section)*

| Metric | Mon M/D | Tue M/D | Wed M/D | Thu M/D | Fri M/D | Sat M/D | Sun M/D |
|--------|---------|---------|---------|---------|---------|---------|---------|
| Spirit min | | | | | | | |
| Journal | | | | | | | |

**Table 1.2-C-mood — Feelings by day** *(copy from `weekly-journal-feelings-*.md` section `TABLE 1.2-C-mood`)*

| Day | Date | Entries | Dominant | Day valence | Tags |
|-----|------|---------|----------|-------------|------|
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
| Entries tagged | | | | (in `Journal Feelings Summary`) |
| Dominant tag | | | | (in summary) |
| Tag volatility | | | | (in summary) |
| Distress entries | | | | `Mood Distress Flag` |

→ Write `Mood Valence`, `Mood Negative %`, `Journal Feelings Summary`, `Mood Distress Flag` from the feelings file **Notion write payload** section. **`Mood Negative %`:** Notion percent field — store as decimal (`0.45` = 45%).

**Table 1.2-E-b — Tag frequency** *(copy from `weekly-journal-feelings-*.md` section `TABLE 1.2-E-b`)*

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
| Healthy / Unhealthy | cite valence, negative %, spirit min | `Mind Health` |

**Table 1.2-H — Mind intentions (upcoming week)** *(Aaron approves before Notion write)*

| Intentions (1–3 bullets) | → Notion field |
|--------------------------|----------------|
| | `Mind Intentions` |

**Death chart (mandatory — same turn as 1.2-H, before advancing):** Remind Aaron to **mark this week on the death chart on the wall** (weeks-of-life grid; memento mori). Physical ritual only — not a Notion field. Confirm he did it (or will before the week starts) before `advance --step 1.2` → `1.3`. Always include on Phase 4b printout under Spirituality & Mind → Goals.

Append mind row to `Intentions Review` on the Weekly Meeting Log.

<a id="13-fitness-review-rate-intentions-5-min"></a>
### 1.3 Fitness — Review · Rate · Intentions (~5 min)

**Data sources:** `weekly-habits-*.md`, `weekly-wellness-trends-*.md`, `health_get_summary({ days: 28 })`, `daily-health-sections.mjs` (FITNESS section), prior week's `Fitness Intentions`.

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
| Supplements (days) | from `weekly-habits-*.md` TABLE 1.3-B-supp | ↑/↓ vs prior week | — |
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

**Table 1.3-D — Fitness insights**

| Insight 1 | Insight 2 |
|-----------|-----------|
| | |

**Table 1.3-E — Fitness health**

| Rating | → Notion field |
|--------|----------------|
| Healthy / Unhealthy | `Fitness Health` |

**Table 1.3-F — Fitness intentions (upcoming week)**

| Intentions (1–3 bullets) | Strength target | Cardio target | → Notion field(s) |
|--------------------------|-----------------|---------------|-------------------|
| | | | `Fitness Intentions`, `Strength Target`, `Cardio Target` |

Append fitness row to `Intentions Review`.

<a id="14-sleep-review-rate-intentions-5-min"></a>
### 1.4 Sleep — Review · Rate · Intentions (~5 min)

**Data sources:** `health_get_summary({ days: 28 })`, `daily-health-sections.mjs` (SLEEP section), prior week's `Sleep Intentions`.

**Present exactly these tables, then health rating, then intentions:**

**Table 1.4-A — Prior sleep intention**

| Last week's `Sleep Intentions` | Evidence | Met? |
|--------------------------------|----------|------|
| | 1-line summary | ✓ / ~ / ✗ |

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
| Healthy / Unhealthy | `Sleep Health` |

**Table 1.4-F — Sleep intentions (upcoming week)**

| Intentions (1–3 bullets) | Sleep target (h) | Wake target (CT) | → Notion field(s) |
|--------------------------|------------------|------------------|-------------------|
| | | | `Sleep Intentions`, `Sleep Target Hours`, `Target Wake Time` |

**Table 1.4-G — Behavioral adjustments** *(required when **this step's** domain = Unhealthy — concrete commitments; **skip table** (write "—") when Healthy)*

| Adjustment | Reason |
|------------|--------|
| | |

→ Write `Behavioral Adjustments` (append domain-labeled bullets; cumulative across Unhealthy domains this session). **Healthy domains: no adjustments needed.**

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
| Healthy / Unhealthy | `Social Health` |

**Table 1.5-G — Social intentions (upcoming week)**

| Social Priority | Intentions (1–3 bullets) | → Notion field(s) |
|-----------------|--------------------------|-------------------|
| Active / Maintenance / Deprioritized | e.g. "2 Small Talk + 1 fitness class" | `Social Priority`, `Social Intentions` |

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

<a id="16-parenting-review-rate-intentions-4-min"></a>
### 1.6 Parenting — Review · Rate · Intentions (~4 min)

**Data sources:** Prior week's `Parenting Intentions`, prior Weekly Meeting Log parenting notes. **Do not** report legal custody schedule (2-2-5-5 is case context only — see `context/family/custody/`, not weekly planning). **Matthew days** for the upcoming week: read **Personal Time Blocks** calendar (`10283d615…@group.calendar.google.com`) — look for `Bus` / `Parent Time` blocks (stable week-to-week; exceptions like camp are Aaron's narrative).

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
| Healthy / Unhealthy | `Parenting Health` |

**Table 1.6-E — Parenting intentions (upcoming week)**

| Intentions (1–3 bullets) | → Notion field |
|--------------------------|----------------|
| concrete plans (outings, rhythm) + summer sanity note | `Parenting Intentions` |

Append parenting row to `Intentions Review`.

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

→ Write `Personal Enjoyment` (rich_text — last week + forward intention). Propose Personal Time Blocks calendar events with approval.

**FIELD CHECK — Phase 1** *(Table 1.check)*

| Group | Required Notion fields |
|-------|------------------------|
| Last-week KPIs | `Strength Sessions`, `Cardio Sessions`, `Spirit Minutes`, `Journal Count`, `Weight Avg`, `Body Fat Avg`, `Lean Mass Avg`, `Sleep Avg`, `Sleep Nights Tracked`, `Wake Time Std Dev Min`, `Bedtime Std Dev Min`, `Sleep Schedule Rating`, `Steps Avg`, `Workout Active Minutes` |
| 1.2 | `Intentions Review` (mind row), `Mind Health`, `Mind Intentions`, `Mood Valence`, `Mood Negative %`, `Journal Feelings Summary`, `Mood Distress Flag`, `Energy Rating`, `Screening Escalation`; PHQ/GAD only if `Screening Escalation` = true |
| 1.3–1.4 | `Fitness/Sleep Health`, `Fitness/Sleep Intentions`, `Strength Target`, `Cardio Target`, `Sleep Target Hours`, `Target Wake Time`, `Behavioral Adjustments` |
| 1.5 | `Small Talk Count`, `Social Events Count`, `Social Review` (incl. **fuel rating**), `Social Intentions Met`, `Social Health`, `Social Priority`, `Social Intentions` |
| `1.4` (Unhealthy) | `Behavioral Adjustments` — **required for that domain**; skip when Healthy |
| `1.5` | Fuel **Stage 1 + 2** + **recovery intentions** (1.5-E-b when triggered) in `Social Review` |
| 1.6 | `Parenting Health`, `Parenting Intentions` |
| 1.7 | `Personal Enjoyment` |

**Do not proceed to Phase 2 (Work) until Table 1.check passes.**

<a id="phase-2-development-15-min"></a>
## Phase 2: Development (~15 min)

**Purpose:** Review last week's dev work (CL + Turbo Gear) and personal projects separately; plan next week from the **monthly Dev Projects tracker**.

**Domains:** Phase **2.1** = Chrome Lot + Turbo Gear dev work. Phase **2.2** = Personal projects (Todoist mirrors). Habits and errands stay in Phase 1 — not here.

**Source files:** `output/weekly-dev-review-*.md`, `output/weekly-habits-*.md`.

**Presentation format:** Group by domain; nest sub-items under parents with indented sub-bullets.

---

<a id="21-dev-review-8-min"></a>
### 2.1 Dev Review (~8 min)

#### A — Last week review (one table group per turn)

**Table 2.1-A — Queued dev work (week under review)**

*The queue row is the main payload — CL/TG Dev Projects with `This Week` checked during the week being reviewed (Mon–Sun before this session). That tracker is canonical; the log snapshot is a cross-check only.*

| Field | Value | Source |
|-------|-------|--------|
| Work health (dev efforts) | | Prior Weekly Log `Work Health` |
| Adjustments trying | | Prior `Dev Adjustments` |
| **Queued dev work** | Parents + sub-items, grouped by domain (CL, then TG) | **Primary:** `weekly-dev-review` § Review week — queued CL/TG dev work, or `weekly-habits` § This Week (CL/TG only). **Cross-check:** prior log `Dev Projects Intended` when populated |

**Table 2.1-B — Dev time logged**

| Day (Mon–Sun) | Minutes | |
|---------------|---------|---|
| | | |
| **Total** | | `weekly-dev-review` Business Development DB |

→ Write `Deep Work Minutes` (review week total). Ops/Field from `weekly-habits` if needed.

**Table 2.1-C — What was accomplished**

**From plan** — *same tree as **2.1-A** (domain → parent → sub-bullets). Do **not** flatten to a table. Mark finished items with ~~strikethrough~~ (Status = Done). Open items stay plain.*

```
**Chrome Lot**
- **Parent** — Status
  - Sub-item — Status
**Turbo Gear**
- ...
```

**Off-queue logged** *(optional — only if `weekly-habits` § Dev Projects completed shows real wins with `This Week` checked; never infer from `last_edited` alone)*

**Detected unlogged** *(from `weekly-habits` sweep — includes **Media stack / Audiobookshelf cluster** for ABS/NAS/Plex incident work often shipped via Cursor without a Dev Project; Aaron flags count + hours)*

| Highlight | Count as shipped? |
|-----------|-------------------|
| | Aaron: yes / no |

→ Write `Accomplishments`, `Logged/Unlogged/Total Accomplishments Count`, `Focused Output Hours Estimate`, `Dev Review`.

#### B — Plan for next week (one table per turn)

**Table 2.1-D — Dev health review** *(agent summarizes; Aaron rates)*

Agent presents a short narrative — **no recommendations, no task list** (selection is **2.1-G**). Cover:

1. **Output** — accomplishments vs queued dev work (2.1-C counts + queue tree).
2. **Time** — dev minutes logged (2.1-B) vs realistic capacity.
3. **Goal progress** — review week vs **planning month** Dev Projects (`weekly-dev-review` § Planning month — CL/TG/Personal trees via `🌙 Month`): what moved, what’s stuck, parked domains noted.

Then Aaron rates:

| | Rating |
|---|--------|
| **A** | Healthy |
| **B** | Unhealthy |

→ Write `Work Health`, `Dev Week Rating`, `Dev Intentions Met`, and fold the summary into `Dev Review`.

**Table 2.1-E — Adjustments** *(only if 2.1-D = Unhealthy)*

**Skip entirely when Healthy.** When Unhealthy: ask Aaron what adjustments he wants to commit to for the next period. **Agent does not recommend adjustments** — capture his words only.

→ Write `Dev Adjustments` (Aaron’s commitments, or leave empty when Healthy).

**Planning next week — three turns (do not combine)**

**Table 2.1-F — Carryover (last week leftovers)**

Open items from the **2.1-C queue tree** still not Done, with **full parent → sub-bullet nesting** (fetch entire ancestor chain; never flatten). **Done parent + open child is not stale** — often a bulk-close artifact (e.g. Operational Reset); Aaron defers explicitly. Empty-titled records: archive, don't present. One turn only.

| | |
|---|---|
| **A** | Continue all on this week's plate |
| **B** | Defer some — reply with item letters to **drop** from carryover |

Deferred items: `This Week = false` at sync (approval). Carried items stay selected for sync.

**Table 2.1-G — Add from roadmap?**

| | |
|---|---|
| **A** | Yes — show current-quarter open items (**2.1-H**) |
| **B** | No — carryover only |

**Table 2.1-H — Roadmap adds** *(only if 2.1-G = Yes)*

Source: `weekly-dev-review` § **Planning month — {domain}** (`🌙 Month` → planning month). Skip **parked** domains. Exclude items already on carryover. Letter each row.

**Empty Dev Project records** (no `Name` title): archive in Notion when detected — do not present.

**Sync** *(after F + G/H confirmed — **run immediately**, approval before Notion writes)*

Execute `node scripts/sync-dev-projects-this-week.mjs --selected=<comma-separated page IDs>`:

1. `This Week = true` on carryover + any roadmap picks only.
2. `This Week = false` on **every other open** CL/TG Dev Project (not just unselected domain items — full open-database sweep for CL/TG types).
3. Clear `This Week` on any **Done** records still checked (stale bulk-close artifacts).

→ Write `Dev Projects Intended` (snapshot), `Dev Priority Context`.

**Present Table 2.1-S — CL/TG This Week slate** *(bulleted tree; must match Notion `This Week` filter exactly)*

Output of sync script § Chrome Lot + Turbo Gear only. Aaron confirms before **2.2**.

---

<a id="22-personal-project-review-7-min"></a>
### 2.2 Personal Project Review (~7 min)

#### A — Context + last week

**Table 2.2-A — Planning month Personal Dev Projects**

Present `weekly-dev-review` § **Planning month — Personal** (`🌙 Month` → planning month). Full parent → sub-bullet tree. **No free-text priority themes** — monthly plan links records in Phase 11b / Phase 5.

**Table 2.2-B — Last week personal plan**

**Bulleted tree only** — same nesting as **2.2-A** / **2.1-C**. Do **not** flatten to a table. One parent block per root; indent children. Append mirror + outcome on each line.

```
**Personal**
- **Parent** — Status · mirror: none / open / completed · done: ✓ / ✗
  - **Child** — Status · mirror: … · done: …
```

Source: prior `Dev Projects Intended` (Personal only) + prior-week Personal `This Week` carryover + **Todoist MCP** (tasks due/completed last week). Confirm completions with Aaron.

#### B — Plan for next week

**Table 2.2-C — Planning month adds** *(only if 2.2-B needs items not on carryover)*

Same source as **2.2-A** — letter rows to add to `This Week` that aren’t already on the plate from **2.1-F** carryover (Personal domain).

**Table 2.2-D — Selection** *(multi-select — reply with letters, or **A** = all / **B** = none)*

Present as a **lettered bulleted tree** (same nesting as **2.2-A**). Letter each **root**; children inherit parent selection unless Aaron splits them explicitly.

After selection — **run immediately** (approval before Notion writes):

Execute `node scripts/sync-dev-projects-this-week.mjs --selected=<all CL/TG IDs from 2.1-S plus selected Personal IDs>`:

1. `This Week = true` on selected Personal Dev Projects only.
2. `This Week = false` on **every other open** Personal Dev Project.
3. Re-run with **combined** CL/TG + Personal selected IDs so the full slate is authoritative (one sync pass).
4. For each selected item: propose **Todoist mirror** (due date + project) — **case-by-case approval** before create.

---

**Table 2.check — Final This Week slate** *(required before Phase 4)*

Run `sync-dev-projects-this-week.mjs` output (or `weekly-habit-summary.mjs` § This Week) and present **one bulleted tree** covering all domains:

```
**Chrome Lot**
- parent → children
**Turbo Gear**
- …
**Personal**
- …
```

This list must **exactly match** the Notion Dev Projects view filtered to `This Week = true` (open items). Aaron confirms **A** = matches / **B** = drift to fix.

**FIELD CHECK — Phase 2** *(gate)*

| Field | Step |
|-------|------|
| `Deep Work Minutes`, `Accomplishments`, counts | 2.1-C |
| `Dev Review`, `Work Health`, `Dev Week Rating`, `Dev Intentions Met` | 2.1-D |
| `Dev Adjustments` | 2.1-E *(Unhealthy only; Aaron-supplied)* |
| `Dev Projects Intended` | 2.1 sync (after F/G/H) |
| `Dev Priority Context` | 2.1-G |
| CL/TG `This Week` sync | 2.1-S |
| Personal `This Week` sync | 2.2-D |
| **Final slate = Notion view** | **2.check** |

**Do not proceed to Phase 4 until `2.check` passes.**

<a id="phase-4-commit-5-min"></a>
## Phase 4: Commit (~5 min)

**Purpose:** Final review, capacity check, execute remaining actions, log everything.

1. **Summary table:** Everything planned across all phases (project selections, personal items, dev intentions)
2. **Final capacity check:** Total planned hours vs. available hours. If total exceeds available, something must move. This is non-negotiable.
3. **Confirm "This Week" checkboxes:** Verify all selected Dev Projects have `This Week = true` and no deselected ones still have it checked.
4. **Store project KPIs on Weekly Meeting Log:** Write `Projects Completed` (count of projects marked Done this week) and `Projects In Progress` (count of projects with This Week checked for the new week).
5. **Verify all FIELD CHECKs (REQUIRED):** Re-run Phase 1 (`1.check`) and Phase 2 (`2.check`). Confirm nothing is blank without N/A + reason. (Activity KPIs + Team Activity Details → **Weekly Ops** commit.)
6. **Record life health ratings (REQUIRED):** Verify weekly-rated selects are set — `Mind Health` (→ `Spirituality Health` in Phase 4), `Fitness Health` (1.3), `Social Health` (1.5), `Parenting Health` (1.6), `Work Health` (2.2). Values: `Healthy` or `Unhealthy`. Admin is not rated in weekly plan.
7. **Update Values DB Health (with approval):** For each category where this week's rating differs from current Values DB Health, update via `personal_notion_update_page` on the category page in Values DB (`342f40c2-487b-80c5`).
8. **Record Starved Values:** Derive from weekly health ratings — set `Starved Values` multi_select to every **rated** category marked **Unhealthy** (Spirituality, Fitness, Work, Social, Parenting). Admin excluded from weekly rating.
9. **Confirm accomplishment fields (REQUIRED):** Verify Phase 2.2 wrote `Logged/Unlogged/Total Accomplishments Count`, `Focused Output Hours Estimate`, and `Accomplishments`. Backfill from habit summary if missing.
10. **Body comp already persisted.** Withings written in Phase 0 (`--days 28`). Don't re-run here.
11. **Execute remaining:** Create any Todoist/Calendar/Notion items not yet committed during earlier phases.
12. **Log to Notion:** Finalize the Weekly Meeting Log entry (`322f40c2-487b-81bd`) with key decisions, action items, and plan summary. Set `Status = Done`, `Session Complete = Complete`.
13. **Week Tracker summary (4b — REQUIRED, two sub-steps):**
    - **4b.1 Confirm week record:** Run `node scripts/weekly-plan-week-summary.mjs --ledger <path> --list-candidates`. Present the candidate table to Aaron via **AskQuestion** (one question). Plans finish on different weekdays — Aaron picks which **Week Tracker** page to write to (letter A–E). Do **not** write until confirmed.
    - **4b.2 Write plan:** After Aaron confirms, run `node scripts/weekly-plan-week-summary.mjs --ledger <path> --week-page-id <id>`. This (1) creates a **Google Doc** in `Plan Records/weekly/` (`scripts/provision-plan-records-drive.mjs` once), (2) sets **`Plan Doc URL`** on the confirmed week record, (3) appends/replaces the **Weekly Plan** section on that Notion page. Suggested title: `Week Starting M/D` = Sunday before ledger `week_of` (Mon 2026-06-08 → `Week Starting 6/7`). Aaron approves production writes.
14. **Update context files** if anything changed.

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
- **Phase 0:** Wellness + journal feelings + social + dev data pulls; trend files; 4-week log history.
- **Phase 0b:** Data integrity table; remediation before Phase 1.
- **Phase 1:** Life review (values, mind, fitness, sleep, social, parenting, personal enjoyment) + targets on Weekly Meeting Log.
- **Phase 2:** Development review + next-week dev plan.
- **Phase 4:** Full Weekly Meeting Log finalized + all FIELD CHECKs; Values DB sync (with approval).
- **Phase 4b:** Confirmed Week Tracker page gets domain-by-domain plan summary + linked Google Doc in `Plan Records/weekly/`.

<a id="failure-modes-and-graceful-degradation"></a>
## Failure modes & graceful degradation

- **Monthly Plan Log missing for planning month:** Hard stop — run monthly plan first (Pre-Phase 0). Do not proceed weekly-only.
- **Weekly data pull script missing:** Rely on parallel MCP pulls per Phase 0 list.
- **Withings sync / `sources.notion.error`:** Skip or note body-comp unavailable; use `--` for body-comp rows; note "Withings sync inactive" or "Withings sync needs attention" in scorecard/footer as specified in Phases 1–2.
- **`sources.health_sync.ok = false`:** Drop Recovery + Activity lines in Values Pulse fitness callout (Phase 1); append diagnostic note; render watch metrics as `--` in habit scorecard with "Health Sync data missing" in footer; do not abort the workflow.
- **`health_persist_recent` / Health Sync issues:** Note in footer; continue with MCP + archived Notion data where available.
- **Partial or null watch fields (RHR, HRV):** Render as `--` until Health Sync folders are enabled.

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