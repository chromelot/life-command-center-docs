> **Source:** [`context/skills/weekly-planning/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-planning/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Weekly Planning — SKILL

## Trigger

This skill activates when Aaron says "weekly plan", "weekly meeting", "plan this week", "sprint planning", or "Monday review". Target duration: ~78 minutes (Phase 1 life review ~28 min · Phase 2 work ~45 min · Phase 3 personal + commit ~10 min).

## Inputs

Load via the router. Read these before starting:

- `context/systems/cadences.md` — cadence health check, Phase 0 schedule
- `context/systems/capacity-rules.md` — limits, overcommitment triggers, intervention protocol
- `context/systems/notion-databases.md` — every DB ID referenced below (Dev Projects, Weekly Meeting Log, source DBs, Values, Workouts, etc.)
- `context/systems/pipedrive.md` — pipeline IDs, stages, user IDs, completion-time API quirks
- `context/systems/knack-fields.md` — Customer + Photographer field references
- `context/systems/hubstaff.md` — member IDs, weekly-report tool
- `context/systems/health-data.md` — MCP architecture, expected fields
- `context/self/values.md` — six categories and current Health statuses (Phase 1.1 context; per-domain ratings in Phase 1 + 2.2 + 3)
- `context/self/eros.md` — primary fuel doctrine (Phase 1.5 fuel check, Phase 3 integration)
- `context/self/dating.md` — relationship status + integration guardrails + risk surface (Phase 3)
- `context/self/social.md` — sarges, Small Talk targets, isolation signals (Phase 1.5)
- **Planning context (canonical):** Monthly + Quarterly Meeting Log fields — pulled in Phase 0; see Phase 2.1. `context/self/current-priorities.md` is fallback only.
- `context/people/index.md` — delegation matrix, 1:1 tracking
- `context/work/chrome-lot/customer-service.md` — Phase 2.6 logic
- `context/work/chrome-lot/sales.md` — Phase 2.7 logic
- `context/work/chrome-lot/operations.md` — Phase 2.8 photographer review logic
- `context/work/turbo-gear/overview.md` — TG strategic sequence (for Phase 2.4 project selection)

## Execution Protocol (mandatory — read `context/workflow-execution.md`)

1. **Init ledger** at session start: `node scripts/workflow-progress.mjs init --workflow weekly-plan --week-of <next-monday>`
2. **Every turn:** `node scripts/workflow-progress.mjs status --workflow weekly-plan` — present only `current_step`
3. **Phase banner** on every user-facing message: `**[Weekly Plan · Phase X.Y — title]**`
4. **One sub-step per turn** — never bundle 1.2 + 1.3 + 1.4 + 1.5 (mind / fitness / sleep / social are separate steps)
5. **Table contract is the spec** — each sub-step lists the **exact tables** to present (column headers fixed). Fill every cell from the named data source; use `—` when data is missing. Do not add metrics, sections, or discussion topics outside that step's tables.
6. **One question per turn** — PHQ-2/GAD-2 items are separate turns
7. **Advance after complete:** `node scripts/workflow-progress.mjs advance --workflow weekly-plan --step <id>`
8. **Phase gates:** `node scripts/workflow-progress.mjs gate --workflow weekly-plan --phase <1|2>` before Phase 2 (work) or Phase 3 (personal)
9. **Tangents:** fix/interrupt, then resume ledger `current_step` — do not skip ahead

## Interaction Style

- **One question at a time.** Never present a wall of choices. Walk through decisions sequentially.
- **Confirm before executing.** Each phase presents proposed actions, gets approval, then executes before moving on.
- **Exclude Shopping List** (Todoist project `6W36wRPXj8qC2RCc`) from all analysis.
- **Data integrity:** All Knack/Pipedrive/Notion/Todoist write operations require explicit user approval before execution (Todoist case-by-case - never batch or assume).

## Required Notion fields — index

Each phase ends with an inline **FIELD CHECK** listing its required Weekly Meeting Log properties. Phase 4 (Commit) verifies all sections. Monthly planning context lives on **Monthly Meeting Log** (`Priority Stack`, `Domains Parked`, `Active CL Sprint`) — backfill in Phase 2.1 if missing.

**Gate rules:**
- Before **Phase 2 (Work)**: Phase 1 FIELD CHECK (`1.check`) must pass.
- Before **Phase 3 (Personal)**: Phase 2 work block complete (through `2.8` + `2.check`).
- Each phase delivers **only** its table contract (see per-phase **Present** blocks below).

## Procedure

## Pre-Phase 0: Monthly Plan Gate (mandatory — runs before everything else)

The weekly plan assumes a committed monthly frame. Do not start Phase 0 until this gate passes.

1. Compute **review month** and **planning month** from today's date (America/Chicago) — same framing as `context/skills/monthly-plan/SKILL.md` (Month framing section). Example: session on 2026-06-05 → review month = May 2026, planning month = June 2026.
2. Query **Monthly Meeting Log DB** (`344f40c2-487b-806d`) for an entry whose **Month** relation points to **review month** OR whose Name matches `Month of [Review Month YYYY]`.
3. **If no matching entry exists:**
   - Tell Aaron: "No Monthly Meeting Log entry for [review month]. Monthly plan must run before weekly plan."
   - **Pause** this workflow. Run `context/skills/monthly-plan/SKILL.md` end-to-end.
   - After monthly plan Phase 12 commits the review-month entry, **resume** weekly plan from Phase 0 below.
   - Do **not** offer to skip or proceed weekly-only — monthly plan is a hard prerequisite.
4. **If entry exists:** Hold for Phase 2.1. Continue to Phase 0.

## Phase 0: Data Pull (silent, before conversation)

**Order:** wellness + log trends first, then work pulls. Mind/body phases run before any work discussion.

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

### Social pulls (with wellness; used in Phase 1.5)

- **Small Talk DB** (`121f40c2-487b-802d`): all entries from **last 7 days** — list Description + Created Date (habit summary has count; Phase 2 needs the list).
- **Google Calendar (last 7 days):** Pull Aaron's primary + Personal Time Blocks calendars. Flag events that look social: friend hangouts, fitness classes, group events, Meetup, dates (non-work). Count → `Social Events Count`.

### Development + work pulls (after wellness/social; silent until Phase 2+)

```
node "scripts/weekly-data-pull.mjs"
```

Pull via MCP in parallel:
1. **Todoist**: Overdue, this week, completion stats (exclude Shopping List)
2. **Dev Projects DB** (`341f40c2-487b-80ac`): `This Week = true` carryover; current-quarter projects Status ≠ Done
3b. **Monthly Meeting Log** (`344f40c2-487b-806d`): review month entry — `Priority Stack`, `Domains Parked`, `Active CL Sprint`, `Action Items`, `Key Wins`, `Key Misses`
3c. **Quarterly Meeting Log** (`344f40c2-487b-80ed`): current quarter — `Priority Stack`, `Domains Parked`, `Next Quarter Focus`
4. **Habit source DBs** (past 7 days — habit summary script is canonical; MCP only if script missing):
   - Workouts (`127f40c2-487b-80ba`): query all, count by Type
   - Small Talk (`121f40c2-487b-802d`): query all, count entries
   - Spirit (`2aaf40c2-487b-8070`): query all, sum Total Time
   - Business Development (`127f40c2-487b-803e`): query all, sum Total Time (formula property -- reader must read `properties["Total Time"].formula.number`, not `.number`)
   - Chrome Lot Operations (`136f40c2-487b-80ba`): query all, sum Total Time
   - Field Work (`237f40c2-487b-80ab`): query all, sum Total Time
   - Journal (`99c9e393-812f-4d73`): query all, count entries
5. **Values DB** (`342f40c2-487b-80c5`): Pull all 6 categories with Health status
6. **Notion Work**: CL tasks/projects status
7. **Pipedrive**: Sales pipeline (pipeline 1), CS pipeline (pipeline 6, full pull), activities (overdue + this week). **For completed activity KPIs:** use `pipedrive_get_activities` with `done: "1"` and `updated_since` set to the Monday of the previous week (RFC3339), then filter results client-side by `marked_as_done_time` within the target week. Pull per-user for open activities (user_id: 18865844, 22704318, 20938631, 19274648) since omitting user_id only returns the authenticated user.
8. **Knack Customers** (`object_2`): field_464 (Current Customer), field_1035 (HJD), field_1601 (Health Status), field_1438 (Account Manager), field_1410 (Invoice Follow Up Status), field_1428 (Adjusted Late Invoices), field_1491 (Aged Invoices)
9. **Knack Photographers** (`object_7`): field_33 (Status), field_1267 (Call-ins Last Month), field_1362 (Avg Issues Per Car), field_1446 (Time Off Requests Last Month), field_1338 (Performance Grade)
10. **Google Calendar**: Next week's events
11. **Hubstaff**: Last week's hours per member via `hubstaff_get_weekly_report`
12. **Health Data**: `health_get_summary({ days: 28 })` above — used throughout Phase 1.

**Reconcile completions first** — before work phases, check what's already been done since last plan and mark complete.

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

## Phase 1: Life Review (~28 min)

**Purpose:** Values context first, then mind → fitness → sleep → social one domain at a time (review → rate health → set intentions). Mind includes wellness screening. Work health rates in Phase 2.2; Admin/Parenting in Phase 3.

**Phase 1 order:** `1.0` → `1.1` Values → `1.2` Mind (incl. wellness) → `1.3` Fitness → `1.4` Sleep → `1.5` Social → `1.check`

### 1.0 Create Weekly Log Entry

Create the **new week's** Weekly Meeting Log entry (Name = `Week of [next Monday YYYY-MM-DD]`, Meeting Date = today). All Phase 1 fields write to this entry.

**Advance ledger:** `1.0` → then present `1.1` only.

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

### 1.2 Mind — Review · Wellness · Rate · Intentions (~6 min)

**Data sources:** `weekly-habits-*.md`, `weekly-wellness-trends-*.md`, `node scripts/daily-health-sections.mjs` (MIND section), prior week's `Mind Intentions`.

**Present exactly these tables in order** (wellness rows one question per turn; health and intentions separate turns):

**Table 1.2-A — Prior mind intention**

| Last week's `Mind Intentions` | Evidence | Met? |
|-----------------------------|----------|------|
| | 1-line summary | ✓ / ~ / ✗ |

**Table 1.2-B — Mind aggregate**

| Metric | Last Week | 4-wk trend | → Notion field |
|--------|-----------|------------|----------------|
| Spirit Minutes | | from wellness file | `Spirit Minutes` |
| Journal Count | | from wellness file | `Journal Count` |

**Table 1.2-C — Mind daily** *(from `daily-health-sections.mjs` MIND section)*

| Metric | Mon M/D | Tue M/D | Wed M/D | Thu M/D | Fri M/D | Sat M/D | Sun M/D |
|--------|---------|---------|---------|---------|---------|---------|---------|
| Spirit min | | | | | | | |
| Journal | | | | | | | |

**Table 1.2-D — Mind insights**

| Insight 1 | Insight 2 |
|-----------|-----------|
| | |

**Table 1.2-E — Wellness screening** *(one row per turn)*

| Item | Prompt | Score | → Notion field |
|------|--------|-------|----------------|
| PHQ-2 Q1 | Little interest or pleasure in doing things? | 0–3 | — |
| PHQ-2 Q2 | Feeling down, depressed, or hopeless? | 0–3 | — |
| PHQ-2 Total | Q1 + Q2 | 0–6 | `PHQ-2 Score` |
| PHQ-2 Severity | None / Mild / Moderate / Moderately Severe / Severe | derived | `PHQ-2 Severity` |
| GAD-2 Q1 | Feeling nervous, anxious, or on edge? | 0–3 | — |
| GAD-2 Q2 | Not being able to stop or control worrying? | 0–3 | — |
| GAD-2 Total | Q1 + Q2 | 0–6 | `GAD-2 Score` |
| GAD-2 Severity | None / Mild / Moderate / Moderately Severe / Severe | derived | `GAD-2 Severity` |
| Energy | How is your energy and capacity this week? | 1–10 | `Energy Rating` |

**Capacity note** (if PHQ-2 ≥ 3 or GAD-2 ≥ 3 or Energy ≤ 4): state reduced work capacity — recorded in Phase 2.4 `Dev Capacity Note`.

**Table 1.2-F — Mind health** *(Aaron confirms; write with approval)*

| Rating | → Notion field |
|--------|----------------|
| Healthy / Unhealthy | `Mind Health` |

**Table 1.2-G — Mind intentions (upcoming week)** *(Aaron approves before Notion write)*

| Intentions (1–3 bullets) | → Notion field |
|--------------------------|----------------|
| | `Mind Intentions` |

Append mind row to `Intentions Review` on the Weekly Meeting Log.

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
| Workout Active Min | | | `Workout Active Minutes` |
| Heart Rate Avg (bpm) | | | `Heart Rate Avg` |
| Resting HR Avg (bpm) | | | `Resting HR Avg` |
| HRV Avg (ms) | | | `HRV Avg` |

**Table 1.3-C — Fitness daily** *(from `daily-health-sections.mjs` FITNESS section)*

| Metric | Mon M/D | Tue M/D | Wed M/D | Thu M/D | Fri M/D | Sat M/D | Sun M/D |
|--------|---------|---------|---------|---------|---------|---------|---------|
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

**Table 1.4-G — Behavioral adjustments** *(if any calendar moves)*

| Adjustment | Reason |
|------------|--------|
| | |

→ Write `Behavioral Adjustments`. Append sleep row to `Intentions Review`.

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

| Metric | Last Week | 4-wk trend | → Notion field |
|--------|-----------|------------|----------------|
| Small Talk entries | N — list each: `date — description` | from wellness file | `Small Talk Count` |
| Calendar social events | N — list each: `date — title` | | `Social Events Count` |
| Days since last Small Talk | N | | (in `Social Review`) |
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

**Table 1.5-E — Fuel check** *(per [eros.md](../../self/eros.md))*

| Signal | This week | Two weeks running? | → Action |
|--------|-----------|-------------------|----------|
| Fuel clean / contaminated / divided | Aaron rates | yes / no | Flag for Phase 3 if contaminated or divided two weeks running |

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

Execute calendar/Todoist with approval (Personal Time Blocks calendar `10283d615faeb91862fc0ccd8f3ac216c7299a58f2196185e912be8f3e3cbe83@group.calendar.google.com`). Append booked events to `Social Intentions`.

**FIELD CHECK — Phase 1** *(Table 1.check)*

| Group | Required Notion fields |
|-------|------------------------|
| Last-week KPIs | `Strength Sessions`, `Cardio Sessions`, `Spirit Minutes`, `Journal Count`, `Weight Avg`, `Body Fat Avg`, `Lean Mass Avg`, `Sleep Avg`, `Sleep Nights Tracked`, `Wake Time Std Dev Min`, `Bedtime Std Dev Min`, `Sleep Schedule Rating`, `Steps Avg`, `Workout Active Minutes` |
| 1.2 | `Intentions Review` (mind row), `Mind Health`, `Mind Intentions`, `PHQ-2 Score`, `GAD-2 Score`, `Energy Rating`, `PHQ-2 Severity`, `GAD-2 Severity` |
| 1.3–1.4 | `Fitness/Sleep Health`, `Fitness/Sleep Intentions`, `Strength Target`, `Cardio Target`, `Sleep Target Hours`, `Target Wake Time`, `Behavioral Adjustments` |
| 1.5 | `Small Talk Count`, `Social Events Count`, `Social Review`, `Social Intentions Met`, `Social Health`, `Social Priority`, `Social Intentions` |

**Do not proceed to Phase 2 (Work) until Table 1.check passes.**

## Phase 2: Work (~45 min)

**Purpose:** All work content — development review, CL operations, CS, sales, people — after Phase 1 life review is complete.

Load (development steps): `weekly-wellness-trends-*.md`, `weekly-habits-*.md`, Phase 0 monthly/quarterly log pulls, prior week `Dev Intentions` + `Dev Projects Intended`.

### 2.1 Current Development Priority (~4 min)

**Present exactly Table 2.1:**

| Source | Field | Value | → Notion |
|--------|-------|-------|----------|
| Quarterly ([Q# YYYY]) | Priority Stack | numbered list | |
| Quarterly | Domains Parked | multi-select | |
| Monthly ([planning month]) | Priority Stack | numbered list — authoritative dev focus | |
| Monthly | Domains Parked | multi-select — what's on pause | |
| Monthly | Active CL Sprint | A–E or Maintenance | feeds Phase 2.5 |
| Monthly | Action Items | planning-month commitments | |

→ Write `Dev Priority Context` (rich_text — copy table narrative).

**Monthly log backfill gate:** If review-month Monthly Log is missing `Priority Stack`, `Domains Parked`, or `Active CL Sprint`:
1. Pull from Quarterly Log → `context/self/current-priorities.md` as draft.
2. Ask Aaron to confirm values (AskQuestion per field or one confirmation).
3. Write to Monthly Meeting Log with approval (`personal_notion_update_page`).
4. Flag: "Planning context was backfilled — rerun monthly Phase 11b properly next month."

Write snapshot to Weekly Meeting Log `Dev Priority Context` (rich_text — copy the table narrative).

**Enforcement (read aloud):**
- If `Domains Parked` includes **Turbo Gear**, do not select TG Dev Projects in 2.4 unless Aaron explicitly overrides.
- `Active CL Sprint` drives Phase 2.5 CL Currency Check.

Ask via AskQuestion (multi-select): "Anything from the priority context that must get a dev slice **this** week?" Options: one per `Priority Stack` line + one per distinct `Action Items` theme + "None — on track as written."

### 2.2 Last Week — Development Scorecard (~7 min)

**Present exactly these tables:**

**Table 2.2-A — Development scorecard**

| Metric | Last Wk | 4-wk trend | → Notion field |
|--------|---------|------------|----------------|
| Deep Work Minutes | | | `Deep Work Minutes` |
| Ops Minutes | | | `Ops Minutes` |
| Field Work Minutes | | | `Field Work Minutes` |
| Logged accomplishments | | | `Logged Accomplishments Count` |
| Unlogged (Aaron-flagged) | | | `Unlogged Accomplishments Count` |
| Total accomplishments | | | `Total Accomplishments Count` |
| Focused output hours est. | | | `Focused Output Hours Estimate` |

**Table 2.2-B — Intended vs actual**

| Source | Content |
|--------|---------|
| Prior `Dev Intentions` | copy from prior log |
| Prior `Dev Projects Intended` | copy from prior log |
| Logged shipped | Dev Projects Done — grouped Personal / CL / TG from `weekly-habits-*.md` |
| Unlogged sweep | from habit summary footer |

**Table 2.2-C — Unlogged accomplishments**

| Source | Count | Highlights |
|--------|-------|------------|
| (from Phase 0 sweep) | | |

Ask: "Any of these count as meaningful shipped slices?" → fold into `Accomplishments`.

**Table 2.2-D — Project narrative** *(one row per parent with `This Week` items last week)*

| Parent project | Status | Notes |
|----------------|--------|-------|
| | done / stalled / deferred | |

**Table 2.2-E — Week rating**

| Field | Value | → Notion field |
|-------|-------|----------------|
| Dev Week Rating | Exceeded / Met / Partial / Missed / Deprioritized | `Dev Week Rating` |
| Dev Intentions Met | same scale + N/A | `Dev Intentions Met` |

→ Write `Accomplishments`, `Dev Review`.

**Table 2.2-F — Work health** *(after scorecard; Aaron confirms)*

| Rating | Actual last week | → Notion field |
|--------|------------------|----------------|
| Healthy / Unhealthy | Deep Work __ min · Ops __ min · Field __ min (from Table 2.2-A) | `Work Health` |

### 2.3 Trend Assessment (~3 min)

**Present exactly Table 2.3:**

| Trend bullet | Evidence |
|--------------|----------|
| 1 | from `weekly-wellness-trends-*.md` + last 4 `Dev Week Rating` |
| 2 | |
| 3 | |

→ Write `Dev Trend Notes`.

### 2.4 Plan Next Week (~8 min)

**Present exactly Table 2.4-A — Capacity**

| Input | Value |
|-------|-------|
| Hubstaff last week | __ h |
| Wellness gate (PHQ/GAD/Energy) | reduced? yes / no |
| Calendar load | __ h blocked |
| Realistic dev hours | __ h (~25h baseline − trip/PTO/custody) |

→ Write `Dev Capacity Note`.

**Table 2.4-B — Adjustments** *(if underperforming)*

| Move | Reason |
|------|--------|
| | |

→ Write `Dev Adjustments`.

**Queue projects (Notion `This Week`):**
1. Tell Aaron: "Open Dev Projects and toggle `This Week = true` on every project/sub-item you intend this week." [Dev Projects](https://www.notion.so/341f40c2487b80acae1fd344d334096c) — wait for confirmation.
2. Read **"This Week — actionable slate"** from `weekly-habits-*.md`. Group by parent; show Type/Status/Completion/Due Date.
3. **One Thing check** (AskQuestion, single-select parent): capture in `Key Decisions`.
4. **Capacity sanity:** >12 sub-items or >available hours → recommend a cut before proceeding.

Write `Dev Intentions` (1–3 bullets — the week's dev focus) + `Dev Projects Intended` (snapshot list of selected sub-items/parents).

**Personal projects → Todoist (required for Personal-type selections):**
Personal work does not run in the protected dev block. For each **Personal** sub-item selected:
1. Decide when in the week (day/slot) with Aaron.
2. Propose Todoist mirror (due date, project) — **case-by-case approval before create**.
3. Custody items: 1–2 concrete tasks in planned slot, not a floating worry.

**FIELD CHECK — Development** *(Table 2.check)*

| Required field |
|----------------|
| `Dev Priority Context`, `Deep Work Minutes`, `Ops Minutes`, `Field Work Minutes`, `Work Health`, `Dev Review`, `Dev Week Rating`, `Dev Intentions Met`, `Dev Trend Notes`, `Dev Capacity Note`, `Dev Adjustments`, `Dev Intentions`, `Dev Projects Intended`, `Accomplishments`, `Logged/Unlogged/Total Accomplishments Count`, `Focused Output Hours Estimate` |

### 2.5 CL Operations Review (~8 min)

**Purpose:** Pipedrive activity scorecard + CL operating currency.

### Part A -- Pipedrive Activity Scorecard

Pull completed activities for the past week using `pipedrive_get_activities` with `done: "1"` and `updated_since` set to the Monday of the target week. Filter results client-side to activities where `marked_as_done_time` falls within the 7-day window. Present as:

```
COMPLETED ACTIVITIES -- WEEK-OVER-WEEK
| User    | Stop | Call/Text | Task | Meeting | Total | Last Wk |
|---------|------|-----------|------|---------|-------|---------|
| Aaron   |      |           |      |         |       |         |
| Lexie   |      |           |      |         |       |         |
| Tristen |      |           |      |         |       |         |
| Ran     |      |           |      |         |       |         |
| Total   |      |           |      |         |       |         |
```

"Last Wk" total comes from the previous Weekly Meeting Log entry. Flag any user with 0 completed activities.

**Store per-user totals on the Weekly Meeting Log entry:** Aaron Activities, Lexie Activities, Tristen Activities, Ran Activities, Total Activities.

### Part B -- Chrome Lot Currency Check (catch-up forcing)

**Purpose:** surface, in one glance, how far behind the operating business is, so the meeting orients around *retiring* backlog rather than just re-planning. Present a small standing scorecard from data already pulled in Phase 0:

```
CL CURRENCY CHECK
| Signal                          | Count / Age | Trend vs last wk |
|---------------------------------|-------------|------------------|
| Todoist overdue (excl Shopping) |             |                  |
| Pipedrive overdue activities    |             |                  |
| Oldest open 1:1 (days)          |             |                  |
| CS deals 60+ days stale         |             |                  |
| Photographers w/ missing grade  |             |                  |
```

Then state the **active repair sprint** from Phase 2.1 `Active CL Sprint` (sequence: A Pipedrive+Todoist → B 1:1 cadence → C customer service → D photographer performance → E sales → Maintenance). Name the specific backlog slice this meeting will retire. This feeds the retire-a-slice rule at commit.

**FIELD CHECK — CL Ops:** `Aaron Activities`, `Lexie Activities`, `Tristen Activities`, `Ran Activities`, `Total Activities`.

### 2.6 CS Management (~10 min)

**Purpose:** Keep customer relationships healthy with structured 60-day check-in cadence and invoice accountability.

### Part A -- Check-in Cadence

1. **Cross-reference Knack & Pipedrive:** Current customers in Knack (field_464=Yes) vs. open CS deals in Pipedrive (pipeline 6). Flag mismatches.
2. **Staleness check:** Calculate days since last activity per CS deal. Flag any 60+ days overdue.
3. **HJD flag:** Any customer with field_1035 > 10 gets flagged.
4. **Health status review:** Check field_1601 for "Needs Attention" or "At Risk" accounts.
5. **Propose weekly check-in batch:** ~6-9 stops, prioritized by:
   - AT RISK / Needs Attention stage first
   - Days overdue (most stale first)
   - Account value
   - Grouped by geography where possible
6. **Delegation:** Assign stops to Tristen, Lexie, Ran, or Aaron based on deal ownership. Spread across the week.

### Part B -- Late Invoice Review

1. Pull customers where field_1410 (Invoice Follow Up Status) = "Account Manager Follow Up" or "Max Escalation"
2. Pull customers where field_1428 (Adjusted Late Invoices) > 3 OR field_1491 (Aged Invoices) > 1
3. Combine both lists (deduplicate). Review each flagged customer one by one.
4. Create Pipedrive activities (escalation follow-up) assigned to either Aaron or the account manager (field_1438) associated with the account.

**Invoice escalation trigger rule:**
- field_1410 = "Account Manager Follow Up" or "Max Escalation" --> always flag
- field_1428 (Adjusted Late Invoices) > 3 --> flag
- field_1491 (Aged Invoices) > 1 --> flag
- Any one of these triggers a review and Pipedrive activity

### Knack Customer Field Reference

| Field | Name | Trigger |
|-------|------|---------|
| field_464 | Current Customer | Filter active |
| field_1035 | High Job Days | Flag if > 10 |
| field_1601 | Customer Health Status | Flag Needs Attention / At Risk |
| field_1438 | Account Manager | Determine delegation |
| field_1410 | Invoice Follow Up Status | Flag "AM Follow Up" / "Max Escalation" |
| field_1428 | Adjusted Late Invoices | Flag if > 3 |
| field_1491 | Aged Invoices | Flag if > 1 |

### Pipedrive Reference

| ID | Meaning |
|----|---------|
| Pipeline 6 | CS Pipeline |
| Stage 28 | Needs Attention |
| Stage 61 | Happy |
| Stage 26 | AT RISK |
| User 18865844 | Aaron |
| User 20938631 | Tristen |
| User 22704318 | Lexie |
| User 19274648 | Ran |

**Outputs:** Pipedrive stop activities for check-ins. Pipedrive escalation activities for invoice issues. Knack/Pipedrive updates if account status changed (with approval).

### 2.7 Sales Management (~12 min)

**Purpose:** Drive new revenue and manage the full sales team's pipeline activity.

### Part A -- Aaron's Sales Activity

**A0 — Deal gaps cleanup (mandatory, before planning stops)**

1. Pre-pull Aaron-owned open deals in Sales (pipeline 1), CS (pipeline 6), and Social Media (pipeline 13) with **no `next_activity_date`** via Pipedrive MCP. Present as a numbered checklist.
2. Instruct Aaron to run CL Bot **`deal gaps`** in Teams ([`cl-bot.md`](../../systems/notion-guides/cl-bot.md)) and schedule every gap via the Adaptive Card **Schedule** buttons (+3 days default).
3. **Gate:** Do not plan new sales stops until gaps = 0. If Aaron explicitly defers a deal, log the deal name + reason in Phase 4 `Key Decisions` and exclude it from the gap count.

**A1 — Stale deal review (one-by-one)**

1. Pull Aaron-owned **Sales Pipeline 1** deals with the **Stale** label (same signal as bot `stale deals`). Fallback if label missing: open deals with no activity in 14+ days.
2. Present **one deal at a time** via `AskQuestion`: **Keep active** / **Move to cold pool (Pipeline 12)** / **Defer decision**.
3. **Cold sales pool** = Pipeline 12 **Not Actively Working** ([`pipedrive.md`](../../systems/pipedrive.md)). Each **Move to Pipeline 12** requires case-by-case Pipedrive approval before `pipedrive_update_deal` — never batch-execute.
4. Deferrals carry to next weekly plan; do not re-present deferred deals unless still stale.

**A2 — Plan the week**

1. Review Pipedrive sales pipeline (pipeline 1): new leads, remaining active stale deals, overdue activities
2. Plan Aaron's sales stops for the week (realistically 2-3)
3. Route planning: use Mapsly manually to cluster stops geographically
4. Review any rescheduled-3x activities -- decide: do, delegate, or drop
5. Accountability: activities completed last week vs. planned

### Part B -- Team Sales Oversight

1. Pull sales pipeline deals owned by Tristen, Lexie, and Ran
2. Review each team member's sales activity: deals in progress, overdue activities, stale leads
3. Flag any deals with no activity in 14+ days
4. Propose reassignments or follow-up nudges where needed
5. Create Pipedrive activities for team members as needed

**Outputs:** Pipedrive activities for Aaron's sales stops. Pipedrive activities or nudges for team sales. Todoist tasks for follow-ups.

### 2.8 People Management (~10 min)

**Purpose:** Keep team relationships healthy, catch photographer and staff performance issues early.

### Part A -- Photographer Performance Review

1. Pull all active photographers from Knack (`object_7`, field_33 = "active")
2. Flag any photographer meeting one or more criteria:
   - field_1267 (Call-ins Last Month) > 3
   - field_1362 (Average Issues Per Car) > 0.5
   - field_1446 (Time Off Requests Last Month) > 3
   - field_1338 (Performance Grade) contains "Below Expectations"
   - field_1338 (Performance Grade) is blank/missing
3. For each flagged photographer, pull the most recent comment from `object_57` where field_1006 = "Photographer" (matched via photographer connection)
4. Present each flagged photographer one by one with: name, flag reasons, metrics, and most recent photographer comment
5. Decide action: coaching conversation, written warning, schedule meeting, no action, etc.
6. If Performance Grade is missing, set it during the session via `knack_update_record` (with approval)
7. Create Todoist tasks for any decided actions

### Knack Photographer Field Reference (object_7)

| Field | Name | Trigger |
|-------|------|---------|
| field_30 | Name | Identity |
| field_33 | Status | Filter active |
| field_1267 | Call-ins Last Month | Flag if > 3 |
| field_1362 | Average Issues Per Car | Flag if > 0.5 |
| field_1446 | Time Off Requests Last Month | Flag if > 3 |
| field_1338 | Performance Grade | Flag if "Below Expectations" or missing |

### Knack Comments Reference (object_57)

| Field | Name |
|-------|------|
| field_857 | Comment text |
| field_858 | User who wrote it |
| field_860 | Date |
| field_1006 | Comment type (filter: "Photographer") |

### Part B -- Staff & 1:1 Management

1. Review people directory: who is overdue for a 1:1?
2. Hubstaff hours: pull last week's report. Flag under-hours, low activity %, zero-hour days.
3. Pick top 1-2 overdue 1:1s to schedule this week
4. Any delegation decisions from earlier phases that need to be communicated

**Outputs:** Todoist tasks for photographer actions. Calendar events for 1:1 meetings. Knack updates for missing performance grades (with approval). Teams messages if needed.

**Do not proceed to Phase 3 (Personal) until Phase 2 work block is complete.**

## Phase 3: Personal Life (~5 min)

**Present exactly these tables:**

**Table 3.1 — Parenting**

| Question | Answer | → Notion field |
|----------|--------|----------------|
| Parenting Health | Healthy / Unhealthy | `Parenting Health` |
| Custody schedule this week | | |
| Activities planned | | |
| Quality of recent time together | | |

**Table 3.2 — Relationship integration** *(see [dating.md](../../self/dating.md))*

| Signal | Answer |
|--------|--------|
| Present & integrated vs divided/hiding | |
| Eros clean / contaminated / divided | |
| Impact on morning keystone, sleep, Bus time | |
| Work/relationship boundary concerns | |
| Fuel flag from Table 1.5-E (two weeks running) | route to [eros.md](../../self/eros.md) daily container if yes |

**Table 3.3 — Compulsion scan**

| Pattern | Present? | Notes |
|---------|----------|-------|
| Apps / substances / avoidance / hidden space | yes / no | per [capacity-rules.md](../../systems/capacity-rules.md) |

**Table 3.4 — Personal enjoyment**

| Item | On calendar? |
|------|--------------|
| Purely fun (beyond Phase 1.5 social pre-commit) | |

**Table 3.5 — Admin health**

| Rating | 1-line actual last week | → Notion field |
|--------|-------------------------|----------------|
| Healthy / Unhealthy | | `Admin Health` |

**Table 3.6 — Unhealthy Values follow-up**

| Category (Unhealthy anywhere this week) | Discussion note |
|----------------------------------------|-----------------|
| (one row per Unhealthy category) | |

**Outputs:** Calendar blocks for personal time (use Personal Time Blocks calendar `10283d615faeb91862fc0ccd8f3ac216c7299a58f2196185e912be8f3e3cbe83@group.calendar.google.com`). Todoist reminders if needed.

## Phase 4: Commit (~5 min)

**Purpose:** Final review, capacity check, execute remaining actions, log everything.

1. **Summary table:** Everything planned across all phases (project selections, CS stops, sales stops, invoice escalations, photographer actions, 1:1s, personal items)
2. **Final capacity check:** Total planned hours vs. available hours. If total exceeds available, something must move. This is non-negotiable.
3. **Confirm "This Week" checkboxes:** Verify all selected Dev Projects have `This Week = true` and no deselected ones still have it checked.
4. **Store project KPIs on Weekly Meeting Log:** Write `Projects Completed` (count of projects marked Done this week) and `Projects In Progress` (count of projects with This Week checked for the new week).
5. **Store activity KPIs on Weekly Meeting Log:** Write `Aaron Activities`, `Lexie Activities`, `Tristen Activities`, `Ran Activities`, `Total Activities` (if not already set in Phase 2.5).
6. **Verify all FIELD CHECKs (REQUIRED):** Re-run Phase 1 (`1.check`), Phase 2 (`2.check` + CL ops), and confirm Phase 3 fields. Confirm nothing is blank without N/A + reason.
7. **Append per-user Pipedrive detail sections** to the Weekly Meeting Log page using `personal_notion_append_blocks`. Use the Pipedrive data already pulled in Phase 0 (completed activities from the past 7 days + all open activities per user). Append the following structure:

   ```
   ---
   # Team Activity Details

   ## Aaron Hoegenauer

   **Completed Activities (X total)**
   - [Stop] CS Check-in: Karma DFW -- Karma DFW CS -- Apr 8
   - [Task] Invoice follow-up -- iDrive1 CS -- Apr 10
   ...

   **Open / Overdue Activities (Y total)**
   - [OVERDUE] Follow up with Nabil -- Tradeline CS -- due Apr 7
   - [Call/Text] Follow up with Marios -- DFW Motorcars CS -- due Apr 14
   ...

   ---

   ## Lexie ...
   (same structure)

   ---

   ## Tristen ...
   (same structure)

   ---

   ## Ran ...
   (same structure)
   ```

   Each activity line: `[type] subject -- deal name -- date`. For completed activities, show `marked_as_done_time` date. For open activities, show `due_date`. Prefix overdue open activities with `[OVERDUE]`. If a user has 0 activities in a section, show "None" instead of an empty list. Use `---` dividers between users.

8. **Record life health ratings (REQUIRED):** Verify all 6 selects are set — `Mind Health` (→ `Spirituality Health` in Phase 4), `Fitness Health` (1.3), `Social Health` (1.5), `Work Health` (2.2), `Admin Health` (3.5), `Parenting Health` (3.1). Values: `Healthy` or `Unhealthy`.
9. **Update Values DB Health (with approval):** For each category where this week's rating differs from current Values DB Health, update via `personal_notion_update_page` on the category page in Values DB (`342f40c2-487b-80c5`).
10. **Record Starved Values:** Derive from life health ratings — set `Starved Values` multi_select to every category rated **Unhealthy** (Spirituality, Fitness, Work, Social, Admin, Parenting). Do not use a separate "felt off-track" question; health ratings are the source of truth.
11. **Confirm accomplishment fields (REQUIRED):** Verify Phase 2.2 wrote `Logged/Unlogged/Total Accomplishments Count`, `Focused Output Hours Estimate`, and `Accomplishments`. Backfill from habit summary if missing.
12. **Body comp already persisted.** Withings written in Phase 0 (`--days 28`). Don't re-run here.
13. **Execute remaining:** Create any Todoist/Calendar/Pipedrive/Notion items not yet committed during earlier phases.
14. **Log to Notion:** Finalize the Weekly Meeting Log entry (`322f40c2-487b-81bd`) with key decisions, action items, and plan summary.
15. **Update context files** if anything changed.

## Cross-Cutting Rules

- **Table contract per phase.** Phase 1 = Values → Mind → Fitness → Sleep → Social → `1.check`. Phase 2 = development (`2.1`–`2.4` + `2.check`) + CL ops (`2.5`–`2.8`). Phase 3 = personal tables. Ledger `current_step` determines which tables are in scope.
- **FIELD CHECK gates.** Run `1.check` before Phase 2 work; `2.check` after development; verify all in Phase 4 commit.
- **Retire-a-slice (catch-up forcing).** Name the slice in Phase 2.5 CL Currency Check; confirm at Phase 4 commit it was archived / scheduled / assigned an owner.
- **Route every item into a bucket.** Each surfaced item is Automated (n8n), Delegated (team 1:1s), or a Scheduled slice (calendar + Todoist mirror).
- **Capacity is non-negotiable.** If total planned work exceeds available hours minus 10-15% buffer, the system pushes back. Something must move.
- **Delegation by default.** For any task deferred 3+ times, suggest delegation before rescheduling. Use the delegation framework in `context/systems/capacity-rules.md`.
- **Max 3 Pipedrive activities per day.** Cap at sustainable levels.
- **Max 5 must-do Todoist tasks per day.** If morning briefing shows >5, defer.
- **Pipedrive accountability is #1 neglected area.** Always check activities completed vs. planned.
- **All data pulls happen in Phase 0 silently.** ~78 minutes is for discussion and decisions.
- **Quarterly docket is the source.** Weekly project selection pulls only from the current quarter's assigned projects. Don't ad-hoc backlog items.

## Outputs

- **Pre-Phase 0:** Monthly plan gate pass (or full monthly plan run + resume).
- **Phase 0:** Wellness + social + work data pulls; trend files; 4-week log history.
- **Phase 0b:** Data integrity table; remediation before Phase 1.
- **Phase 1:** Life review (values, mind, fitness, sleep, social) + targets on Weekly Meeting Log.
- **Phase 2:** All work — dev review, CL ops, CS, sales, people.
- **Phase 3:** Personal (parenting, relationship, compulsion, admin health).
- **Phase 4:** Full Weekly Meeting Log finalized + all FIELD CHECKs; Values DB sync (with approval).

## Failure modes & graceful degradation

- **Monthly Meeting Log entry missing for review month:** Hard stop — run monthly plan first (Pre-Phase 0). Do not proceed weekly-only.
- **Weekly data pull script missing:** Rely on parallel MCP pulls per Phase 0 list.
- **Withings sync / `sources.notion.error`:** Skip or note body-comp unavailable; use `--` for body-comp rows; note "Withings sync inactive" or "Withings sync needs attention" in scorecard/footer as specified in Phases 1–2.
- **`sources.health_sync.ok = false`:** Drop Recovery + Activity lines in Values Pulse fitness callout (Phase 1); append diagnostic note; render watch metrics as `--` in habit scorecard with "Health Sync data missing" in footer; do not abort the workflow.
- **`health_persist_recent` / Health Sync issues:** Note in footer; continue with MCP + archived Notion data where available.
- **Partial or null watch fields (RHR, HRV):** Render as `--` until Health Sync folders are enabled.

## See also

- `../../router.md`
- `../monthly-plan/SKILL.md` — monthly review cadence
- `../quarterly-plan/SKILL.md` — quarterly docket source for project selection
- `../../systems/cadences.md`
- `../../systems/capacity-rules.md`
- `../../systems/notion-databases.md`
- `../../systems/pipedrive.md`
- `../../systems/knack-fields.md`
- `../../systems/hubstaff.md`
- `../../systems/health-data.md`
- `../../self/values.md`, `../../self/current-priorities.md`
- `../../people/index.md`
- `../../work/chrome-lot/customer-service.md`, `../../work/chrome-lot/sales.md`, `../../work/chrome-lot/operations.md`
- `../../work/turbo-gear/overview.md`