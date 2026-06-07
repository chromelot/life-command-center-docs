> **Source:** [`context/skills/weekly-planning/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-planning/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Weekly Planning — SKILL

## Trigger

This skill activates when Aaron says "weekly plan", "weekly meeting", "plan this week", "sprint planning", or "Monday review". Target duration: ~60 minutes.

## Inputs

Load via the router. Read these before starting:

- `context/systems/cadences.md` — cadence health check, Phase 0 schedule
- `context/systems/capacity-rules.md` — limits, overcommitment triggers, intervention protocol
- `context/systems/notion-databases.md` — every DB ID referenced below (Dev Projects, Weekly Meeting Log, source DBs, Values, Workouts, etc.)
- `context/systems/pipedrive.md` — pipeline IDs, stages, user IDs, completion-time API quirks
- `context/systems/knack-fields.md` — Customer + Photographer field references
- `context/systems/hubstaff.md` — member IDs, weekly-report tool
- `context/systems/health-data.md` — MCP architecture, expected fields
- `context/self/values.md` — six categories and current Health statuses (Phase 1 Values Pulse)
- `context/self/eros.md` — primary fuel doctrine (Phase 1 Fuel Check, Phase 7 integration)
- `context/self/dating.md` — relationship status + integration guardrails + risk surface (Phase 7)
- `context/self/current-priorities.md` — what's hot this quarter
- `context/people/index.md` — delegation matrix, 1:1 tracking
- `context/work/chrome-lot/customer-service.md` — Phase 4 logic
- `context/work/chrome-lot/sales.md` — Phase 5 logic
- `context/work/chrome-lot/operations.md` — Phase 6 photographer review logic
- `context/work/turbo-gear/overview.md` — TG strategic sequence (for Phase 3 project selection)

## Interaction Style

- **One question at a time.** Never present a wall of choices. Walk through decisions sequentially.
- **Confirm before executing.** Each phase presents proposed actions, gets approval, then executes before moving on.
- **Exclude Shopping List** (Todoist project `6W36wRPXj8qC2RCc`) from all analysis.
- **Data integrity:** All Knack/Pipedrive/Notion/Todoist write operations require explicit user approval before execution (Todoist case-by-case - never batch or assume).

## Operating model - this meeting is the control plane

This is the single place where all of Aaron's lines of effort are held at once, so they don't have to live in his head day to day. **Daily, Aaron carries only two things: the morning gate (grounded - meditation/yoga + physically set) and today's 1-3 tasks.** Everything surfaced in this meeting must be routed into exactly one of three buckets before the meeting ends - nothing leaves still living only in Aaron's head:

- **Automated** - handled by n8n (e.g., the 9am/5pm Pipedrive->Todoist sync that keeps CL currency).
- **Delegated** - pushed to the team through the 1:1 channels (Tristen / Lexie / Ran).
- **Scheduled slice** - a concrete task on this week's calendar + a Todoist mirror (per Phase 3 Step 6), for the things only Aaron can do.

**Reset-period note (active now):** the morning dev block runs ONE Chrome Lot operational repair sprint at a time, in order: (A) Pipedrive + Todoist currency, (B) team 1:1 cadence, (C) customer service, (D) photographer performance, (E) sales + sales management. Each is taken to a *path to currency* (archive the slipping stuff, schedule the cadence + assign owners, start the plan), then handed to this meeting to maintain. **Turbo Gear is parked** until the operating areas are on a path to currency. Each weekly meeting should name the currently active repair sprint and confirm progress.

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
4. **If entry exists:** Hold it for Phase 1a. Continue to Phase 0.

## Phase 0: Data Pull (silent, before conversation)

If the weekly data pull script exists, run it:
```
node "scripts/weekly-data-pull.mjs"
```

Also run the weekly habit + accomplishment summary script -- this is the canonical source for Phase 2 Part A (habits week-over-week), Part B (project counts), and Part B2 (Unlogged Accomplishments Sweep). It correctly reads formula-typed `Total Time` properties (Spirit/CLOps/FieldWork/BusinessDev) which `weekly-data-pull.mjs` does not touch:
```
node "scripts/weekly-habit-summary.mjs"
```
Output goes to `output/weekly-habits-YYYY-MM-DD.md`. Read that file -- DO NOT re-query the habit DBs ad-hoc, the script handles it.

Also pull Withings body-composition data and write it straight to Notion so the MCP query below sees fresh rows:
```
node "scripts/withings-sync.mjs" --days 14 --write
```
This refreshes the past 14 days of weigh-ins into the Health Data DB. If the script errors with missing credentials/token, skip it and note that body-comp data is unavailable for this week (the MCP will still return whatever Notion already has).

Then archive the past 14 days of Health Sync watch data into the Health Data DB so older periods accumulate beyond the ~30-day Drive rolling window:
```
health_persist_recent({ days: 14 })
```
This upserts Steps, Heart Rate Avg/Max, Sleep stages, and Workout aggregates per day. Today's row is intentionally skipped (partial-day data is misleading). Watch fields only -- never overwrites Withings body comp.

Then call the **health-data MCP** for one consolidated pull of body comp + watch metrics:
```
health_get_summary({ days: 14 })
```
This returns:
- `stats` -- 14-day mins/maxes/averages for weight, body fat, lean mass, sleep, heart-rate avg/max, RHR, HRV, steps, distance, workouts.
- `trend` -- last-7-vs-prior-7 deltas for the same fields.
- `daily` -- per-date rows so you can drop blanks/spikes.
- `sources.health_sync` -- if Health Sync's Drive export is stale or auth fails, watch metrics will be null but body comp from Notion still flows through. Note "Health Sync data missing" in the scorecard footer rather than aborting.

Use this single result throughout Phase 1 (Fitness pulse) and Phase 2 (Habit scorecard). **Do NOT** read `output/withings-*.json` directly anymore -- the MCP is now the source of truth.

Otherwise pull data via MCP tools in parallel:
1. **Todoist**: Overdue, this week, completion stats (exclude Shopping List)
2. **Dev Projects DB** (`341f40c2-487b-80ac`): All projects with `This Week = true` (last week's selections). All projects assigned to current quarter (via Quarter relation) with Status != Done.
3. **Weekly Meeting Log DB** (`322f40c2-487b-81bd`): Most recent entry (previous week) for **week-over-week comparison** in the habit scorecard.
3b. **Monthly Meeting Log DB** (`344f40c2-487b-806d`): Entry for **review month** (verified in Pre-Phase 0 gate) — used in Phase 1a monthly alignment. Also pull the prior month's entry if present for MoM context on any monthly KPI referenced during the session.
4. **Habit Scorecard source DBs** (all filtered by past 7 days):
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
12. **Health Data (body comp + watch)**: Loaded via the `health-data` MCP `health_get_summary({ days: 14 })` call above. Use the returned `stats`, `trend`, and `daily` objects throughout Phase 1 and Phase 2.

**Reconcile completions first** -- before presenting anything, check what's already been done since last plan and mark complete.

## Phase 1: Center (~5 min)

**Purpose:** Ground the week in self-awareness, catch warning signs early, adjust capacity before planning.

### Step 1: Manual Values Review
Prompt Aaron with a direct link to his Values database: [Values Database](https://www.notion.so/342f40c2487b80c5a2aee48ca48b4a20). Ask him to spend a few minutes reading through the 6 categories, then pause and wait for confirmation before continuing.

### Step 2: Wellness Screening
Present these one at a time using AskQuestion where available, otherwise one-question-per-message:

**PHQ-2 Depression Screen** (each scored 0-3, total 0-6):
- Over the last 2 weeks, how often have you had little interest or pleasure in doing things?
- Over the last 2 weeks, how often have you been feeling down, depressed, or hopeless?
Options: Not at all (0), Several days (1), More than half the days (2), Nearly every day (3)

**GAD-2 Anxiety Screen** (each scored 0-3, total 0-6):
- Over the last 2 weeks, how often have you felt nervous, anxious, or on edge?
- Over the last 2 weeks, how often have you been unable to stop or control worrying?
Options: Not at all (0), Several days (1), More than half the days (2), Nearly every day (3)

**Energy & Capacity**: Self-rating 1-10

### Step 3: Values Pulse
After the manual review, present each value category's **Health status** and **How to Spend Time** (time target) from the Values DB (`342f40c2-487b-80c5`), paired with the actual time-based KPI data from the source DBs (already pulled in Phase 0) for the past 7 days. Use this table format:

```
VALUES PULSE -- HEALTH + TIME TARGETS vs. ACTUAL
| Category      | Health     | Time Target                | Actual This Week        |
|---------------|------------|----------------------------|-------------------------|
| Spirituality  | Healthy    | (from "How to Spend Time") | X min (Spirit DB)       |
| Fitness       | Healthy    | (from "How to Spend Time") | X strength + X cardio   |
| Work          | Unhealthy  | (from "How to Spend Time") | X min (Business Dev DB) |
| Social        | Healthy    | (from "How to Spend Time") | X Small Talk entries    |
| Admin         | Healthy    | (from "How to Spend Time") | (qualitative)           |
| Parenting     | Healthy    | (from "How to Spend Time") | (qualitative)           |
```

The actual columns map to source DBs like this:
- **Spirituality** -> Spirit (`2aaf40c2-487b-8070`) Total Time sum
- **Fitness** -> Workouts (`127f40c2-487b-80ba`) count by type (Strength/Cardio) **plus a Body Comp + Recovery callout** built from the `health_get_summary({ days: 14 })` result. Format:
  - `Body comp: 7d avg <X> lbs (Δ<±Y> lbs) · Body Fat <X>% (Δ<±Y>%) · Lean <X> lbs (Δ<±Y> lbs)` (from `stats.weight_lbs.avg`, `trend.weight_lbs`, etc.)
  - `Recovery: Sleep avg <X.X>h (Δ<±Y>h) · HR avg <X> bpm (Δ<±Y>) · RHR avg <X> bpm (Δ<±Y>) · HRV avg <X> ms (Δ<±Y>)` (from `stats.sleep_hours.avg`, `trend.sleep_hours`, `stats.heart_rate_avg_bpm.avg`, `trend.heart_rate_avg_bpm`, `stats.resting_hr.avg`, `trend.resting_hr`, `stats.hrv_rmssd.avg`, `trend.hrv_rmssd`). RHR + HRV remain null until those Health Sync folders are enabled -- render them as `--` until then.
  - `Activity: Steps avg <X>/day (Δ<±Y>) · Workouts <N> sessions / <M> active min (Δ<±workout_active_minutes>)` (from `stats.steps.avg`, `trend.steps`, `stats.workout_count.total`, `stats.workout_active_minutes.total`, `trend.workout_active_minutes`)
  - If `sources.health_sync.ok = false`, drop the Recovery + Activity lines and append "(Health Sync data missing -- run `npm test` in `mcp/health-data` to diagnose)". If `sources.notion.error`, surface body-comp blank as "Withings sync needs attention".
- **Work** -> Business Development (`127f40c2-487b-803e`) Total Time sum + Chrome Lot Ops (`136f40c2-487b-80ba`) Total Time sum + Field Work (`237f40c2-487b-80ab`) Total Time sum (all three are formula-typed numeric properties)
- **Social** -> Small Talk (`121f40c2-487b-802d`) count
- **Admin, Parenting** -> no direct time KPI (qualitative reflection)

Then ask via AskQuestion (multi-select): "Which of these areas felt off-track or neglected this past week?"
Options: Spirituality, Fitness, Work, Social, Admin, Parenting, None -- all tracking

If any area is flagged, note it for the relevant phase later. Don't deep-dive here.

### Step 3b: Fuel Check (eros)
Per [eros.md](../../self/eros.md), the primary motivation source. Ask one question via AskQuestion:

> **Fuel check:** Is the charge running clean or contaminated this week? Aimed outward (work, presence, creation) or sliding into secret-seeking / consumption? Integrated or divided?

Options: Clean and aimed outward / Mixed / Contaminated — secret-seeking or consumption / Divided — hiding inside the relationship.

If "contaminated" or "divided" — note it for Phase 7 (do not deep-dive here). If flagged two weeks running (check the prior Weekly Meeting Log entry), escalate it as a Phase 7 intervention: name it directly and route back to the eros daily container.

### Step 4: Capacity Adjustment
If PHQ-2 >= 3 or GAD-2 >= 3 or Energy <= 4, collaborate on reducing planned load before continuing.

**Outputs:** Log scores to Weekly Meeting Log DB (`322f40c2-487b-81bd`). Adjust total available hours if needed.

## Phase 1a: Monthly Plan Review (~3 min)

**Purpose:** Ground the week in the committed monthly plan before tactical project selection.

Using the Monthly Meeting Log entry for **review month** (pulled in Pre-Phase 0 / Phase 0):

```
MONTHLY PLAN ALIGNMENT — [Planning Month YYYY]
| Signal | From Monthly Log |
|--------|------------------|
| Review month closed | [review month] — Key Wins / Key Misses (1 line each) |
| Action Items (planning month) | bullet summary from Action Items field |
| Starved Values | multi-select from entry, if any |
| Monthly KPIs | CL Revenue / customer count / TG demos / projects completed — only surface fields populated on the entry |
```

Ask via AskQuestion (multi-select): "Anything from the monthly plan that must get a slice **this** week?" Options: derive from Action Items themes (one option per distinct commitment) plus "None — monthly plan is on track as written."

**Outputs:** Flagged priorities carried into Phase 3 (One Thing check, capacity sanity) and Phase 8 commit. No Notion writes.

## Phase 2: Last Week Review (~10 min)

**Purpose:** Score last week's habits and review project progress before planning the new week.

### Part A -- Habit Scorecard

Compute from source databases (queried in Phase 0), NOT from Week Tracker rollups:

| Metric | Source DB | Calculation | Target |
|--------|-----------|-------------|--------|
| Strength Sessions | Workouts (`127f40c2-487b-80ba`) | Count where Type = Pull/Push/Legs/Deadlift/Full Body | 5/week |
| Cardio Sessions | Workouts (`127f40c2-487b-80ba`) | Count where Type = Cardio | -- |
| Small Talk Count | Small Talk (`121f40c2-487b-802d`) | Count entries | 3-5/week |
| Spirit Minutes | Spirit (`2aaf40c2-487b-8070`) | Sum Total Time | 105-140 min/week |
| Deep Work Minutes | Business Development (`127f40c2-487b-803e`) | Sum Total Time (formula = Calc. Minutes + Minutes; reader must handle formula-typed numeric properties) | -- |
| Ops Minutes | Chrome Lot Operations (`136f40c2-487b-80ba`) | Sum Total Time | Monitor (is this crowding out dev?) |
| Field Work Minutes | Field Work (`237f40c2-487b-80ab`) | Sum Total Time | -- |
| Journal Count | Journal (`99c9e393-812f-4d73`) | Count entries | -- |
| Weight Avg (lbs) | Health MCP `health_get_summary` | `stats.weight_lbs.avg` | -- |
| Body Fat Avg (%) | Health MCP `health_get_summary` | `stats.body_fat_pct.avg` | -- |
| Lean Mass Avg (lbs) | Health MCP `health_get_summary` | `stats.lean_lbs.avg` | Hold or grow |
| Sleep Avg (hours) | Health MCP `health_get_summary` | mean of `daily.slice(-7).sleep_hours` | 7-8 |
| HR Avg (bpm) | Health MCP `health_get_summary` | mean of `daily.slice(-7).heart_rate_avg_bpm` | -- |
| Resting HR Avg (bpm) | Health MCP `health_get_summary` | mean of `daily.slice(-7).resting_hr` (null until folder enabled) | -- |
| HRV Avg (ms RMSSD) | Health MCP `health_get_summary` | mean of `daily.slice(-7).hrv_rmssd` (null until folder enabled) | Hold or grow |
| Steps Avg (per day) | Health MCP `health_get_summary` | mean of `daily.slice(-7).steps` | 8000+ |
| Workout Active Minutes | Health MCP `health_get_summary` | sum of `daily.slice(-7).workout_active_minutes` | -- |

**Body-comp + watch rows are best-effort.** If `sources.notion.error` is set, render body-comp values as `--` and note "Withings sync inactive" in the scorecard footer. If `sources.health_sync.ok` is false, render watch values as `--` and note "Health Sync data missing" in the scorecard footer. Don't abort the rest of the scorecard either way.

Present as a scorecard table comparing this week's numbers against targets AND the previous week (from the previous Weekly Meeting Log entry pulled in Phase 0):

```
HABIT SCORECARD -- WEEK-OVER-WEEK
| Metric              | This Wk | Last Wk | Target   | Trend |
|---------------------|---------|---------|----------|-------|
| Strength Sessions   |         |         | 5/week   |       |
| Cardio Sessions     |         |         | --       |       |
| Small Talk Count    |         |         | 3-5/week |       |
| Spirit Minutes      |         |         | 105-140  |       |
| Deep Work Minutes   |         |         | --       |       |
| Ops Minutes         |         |         | Monitor  |       |
| Field Work Minutes  |         |         | --       |       |
| Journal Count       |         |         | --       |       |
| Weight Avg (lbs)    |         |         | --       |       |
| Body Fat Avg (%)    |         |         | --       |       |
| Lean Mass Avg (lbs) |         |         | Hold/grow|       |
| Sleep Avg (hours)   |         |         | 7-8      |       |
| HR Avg (bpm)        |         |         | --       |       |
| Resting HR Avg      |         |         | --       |       |
| HRV Avg (ms RMSSD)  |         |         | Hold/grow|       |
| Steps Avg/day       |         |         | 8000+    |       |
| Workout Active Min  |         |         | --       |       |
```

**For the body-comp rows:** "This Wk" = mean of `daily.slice(-7).weight_lbs` etc. (most-recent 7 days from the 14-day window). "Last Wk" = mean of `daily.slice(0, 7)` (older 7 days). "Trend" comes from `trend.weight_lbs` etc. (signed delta with one decimal, e.g., `-0.8 lbs`, `+0.3%`).

**For the watch rows (Sleep / HR / RHR / HRV / Steps / Workout):** Same approach -- compute "This Wk" from `daily.slice(-7)` and "Last Wk" from `daily.slice(0, 7)`. "Trend" comes from `trend.sleep_hours`, `trend.heart_rate_avg_bpm`, `trend.resting_hr`, `trend.hrv_rmssd`, `trend.steps`, `trend.workout_active_minutes`. RHR + HRV will stay `--` until those Health Sync folders are enabled in the app.

Flag anything significantly below target or showing a notable drop from last week.

**After presenting the scorecard, store these KPIs on the Weekly Meeting Log entry** created in Phase 1. Write the computed values to the structured number fields:
- Habit time fields: Strength Sessions, Cardio Sessions, Small Talk Count, Spirit Minutes, Deep Work Minutes, Ops Minutes, Field Work Minutes, Journal Count
- Body comp: Weight Avg, Body Fat Avg, Lean Mass Avg
- Watch metrics: Sleep Avg, Heart Rate Avg, Resting HR Avg, HRV Avg, Steps Avg, Workout Active Minutes

Omit any field where the underlying data is null. Use the most-recent 7-day mean from `daily.slice(-7)` for each watch metric (not the 14-day `stats.*.avg`, which would dilute toward last week's data).

### Part A2 -- Pipedrive Activity Scorecard

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

### Part A3 -- Chrome Lot Currency Check (catch-up forcing)

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

Then state the **active repair sprint** (per the reset sequence: A Pipedrive+Todoist currency -> B team 1:1 cadence -> C customer service -> D photographer performance -> E sales) and the specific backlog slice this meeting will retire. This feeds the retire-a-slice rule at commit.

### Part B -- Project Review

Pull Dev Projects where `This Week = true` (last week's selections, including sub-items):
1. Group items by top-level parent. For each parent, list selected sub-items + their statuses.
2. Ask Aaron for a quick narrative per parent: which sub-items got done, which didn't move, why?
3. **Sub-items are the unit of completion.** Mark Done sub-items with `Status = Done`. Their `This Week = true` stays as historical record OR can be unchecked -- Aaron's choice. Phase 3's manual selection is what defines next week's `This Week` set.
4. **Parent-level closure:** A parent is only marked `Status = Done` when ALL its sub-items are Done. Otherwise it stays In progress and continues across weeks naturally.
5. For sub-items that didn't move at all and were carryover from a previous week, decide: keep, defer, delegate, or drop. If deferred 3+ times, route through the delegation framework before keeping.

**Outputs:** Updated KPIs on Weekly Meeting Log. Sub-item Status updates where applicable. No need to manually clear `This Week` -- Aaron will reset/adjust during Phase 3 directly in Notion.

### Part B2 -- Unlogged Accomplishments Sweep

**Purpose:** Surface shipped work that isn't tracked in Dev Projects DB, so the weekly briefing reflects the *full* week's output. Aaron frequently ships meaningful work organically (bot deploys, n8n workflow updates, ad-hoc system fixes, infra changes) that never gets a Notion project entry. Without this sweep, those wins are invisible to the weekly plan and Aaron undercounts his own throughput, which distorts capacity calibration for the following week.

Pull from these sources (silent in Phase 0, present here):

1. **Code changes in workspace (last 7 days):**
   - **If workspace has `.git`:** run `git log --since="7 days ago" --pretty=format:'%h | %ad | %s' --date=short`. Group commits by directory/component. Highlight any commit cluster with 5+ commits or any commit touching a deploy script (`deploy-*.mjs`).
   - **If no git** (this workspace currently doesn't have `.git`): fall back to file-mtime scan via Node script:
     ```
     node -e "const {readdirSync,statSync}=require('fs');const path=require('path');const cutoff=Date.now()-7*86400000;function walk(d){const out=[];try{for(const e of readdirSync(d,{withFileTypes:true})){if(e.name.startsWith('.')||e.name==='node_modules')continue;const p=path.join(d,e.name);if(e.isDirectory())out.push(...walk(p));else{const s=statSync(p);if(s.mtimeMs>=cutoff)out.push({p,mt:s.mtimeMs});}}}catch{}return out;}const dirs=['scripts','n8n','context','mcp'];const all=dirs.flatMap(d=>walk(d));all.sort((a,b)=>b.mt-a.mt);for(const f of all)console.log(new Date(f.mt).toISOString().slice(0,10)+' | '+f.p);"
     ```
   - Group resulting files by top-level directory. Flag any cluster with 5+ files or any deploy/build artifact change.

2. **n8n workflow deployments / changes:**
   - Check `n8n/bots/cl-bot/deploy-teams-bot.mjs` (or stub `n8n/deploy-teams-bot.mjs`) and other `n8n/**/deploy-*.mjs` for mtime in the last 7 days.
   - Check `n8n/teams-app/build/*.zip` for new builds in the last 7 days.
   - Check the `n8n/` folder generally for new or modified workflow JSON files (last 7 days).

3. **Notion writes outside Dev Projects (last 7 days):**
   - Query Weekly Meeting Log, Monthly Meeting Log, Quarterly Outcomes, and other system-of-record DBs for entries created or substantially edited in the last 7 days.
   - This catches workflow runs (weekly/monthly/quarterly meeting logs themselves), KPI updates, and standalone documentation.

4. **Other system writes (best effort):**
   - Knack object creations the bot performed on Aaron's behalf (object_77 perf meetings, object_75 call-ins) — pull only if the source script has these counts.
   - Process Street workflows started/completed in the last 7 days.

**Present as:**

```
UNLOGGED ACCOMPLISHMENTS -- LAST 7 DAYS
| Source         | Count | Highlights                                              |
|----------------|------:|---------------------------------------------------------|
| Git commits    |    XX | <top 3 themes by commit volume>                         |
| n8n deploys    |    XX | <list deploy scripts touched + bot zip rebuilds>        |
| Notion writes  |    XX | <list system DBs with substantial new entries>          |
| Process Street |    XX | <workflows completed>                                   |
```

After presenting, **ask Aaron:** "Any of these represent a *meaningful shipped slice* that should be counted alongside the Dev Projects completions? If yes, name them — I'll fold them into the weekly accomplishment count."

**Crucial principle:** Don't try to auto-classify which commits/deploys are "real" accomplishments — surface the data, let Aaron flag what counts. The goal is to *catch* hidden work, not to grade it.

**Outputs:** Combined accomplishment count (Dev Projects Done + Aaron-flagged unlogged items) is the canonical "shipped this week" number for capacity calibration in Phase 3. Store as a rich-text bullet list on the Weekly Meeting Log under "Last Week Accomplishments."

## Phase 3: Project Selection for This Week (~12 min)

**Purpose:** Choose this week's focus projects from the current quarter's docket, accounting for carryover work.

**Why manual selection in Notion:** Most of Aaron's parent projects are multi-week containers with their own sub-item hierarchies already broken out in the Dev Projects DB. The natural unit of weekly work is a **specific sub-item**, not a whole parent. A Q&A-driven selection flow forces the agent to redundantly model that hierarchy in chat. Faster: Aaron toggles `This Week` directly on the projects/sub-items he wants in Notion, then the agent reads the result and confirms.

### Step 1: Pause and Direct to Notion
Tell Aaron: "Open the Dev Projects DB and toggle `This Week = true` on every project AND sub-item you intend to work on this week. Pick parents only when you mean the whole project; pick specific sub-items when you only want a slice. Carryover (anything still `This Week = true` from last week) is already accounted for. Tell me when you're done."

Direct link: [Dev Projects](https://www.notion.so/341f40c2487b80acae1fd344d334096c)

Wait for Aaron's confirmation before continuing. **Do NOT use AskQuestion to enumerate parents/sub-items**; that's what this step replaces.

### Step 2: Pull the Selection
Once confirmed, query the Dev Projects DB with `This Week = true` (page_size 100). Then:
- Group results by top-level parent (use the `Parent item` relation; if empty the item is itself a top-level parent).
- For each parent, show its `Type`, `Status`, `Completion`, `Due Date`, and the list of selected sub-items beneath it.
- Show a domain count summary: how many items per Type (Personal / Chrome Lot / Turbo Gear).

### Step 3: One Thing Check
Ask via AskQuestion: "Of everything you've marked, which **one** parent project (if you finish its slice this week) makes the most other stuff easier or irrelevant?" Single-select from the parents present. Flag the answer as the priority for the week and capture in the Weekly Meeting Log's Key Decisions.

### Step 4: Capacity Sanity Check
Total selected sub-items across all parents. Heuristic: each sub-item averages 2-4 hours of focused work. If total > 12 sub-items or projected hours > available capacity (~25h baseline minus trip/PTO time), recommend trimming a slice before proceeding. **Capacity is non-negotiable** -- if Aaron says "keep it all," push back once with a specific cut suggestion before deferring to him.

### Step 5: Confirm and Move On
No further Notion writes are needed in this phase -- Aaron has already set `This Week = true` directly. The agent's only write here is logging the priority project to the Weekly Meeting Log in Phase 8.

### Step 6: Personal Project Scheduling + Todoist Mirrors
Personal-type projects (custody modification, taxes / Bench->QuickBooks, personal dev, admin) lack the forcing function that Chrome Lot work gets from the 9am Pipedrive->Todoist sync and that Turbo Gear gets from the protected morning builder block. Left untracked, they drift. So for the **Personal** sub-items selected this week:
1. With Aaron, decide **when** in the week he'll work each one (day/slot), respecting capacity caps and protected blocks.
2. **Propose a Todoist mirror** for each (e.g., project Office / Same Day To Do) with the chosen due date so it surfaces in daily execution. Honor the 5-must-do/day cap.
3. **Custody is kept in bounds here** — scheduled as one or two concrete tasks in its planned slot, not left as a floating worry. This replaces any standalone recurring custody block. (Custody source: `context/family/custody/index.md`.)
4. **Todoist writes require case-by-case approval** (per `context/rules.md`) — propose the mirrors, create on Aaron's go.

**Note:** Todoist is NOT used for Chrome Lot or Turbo Gear Dev Project tracking — those stay at the Notion project level (CL also surfaces via the 9am Pipedrive->Todoist sync; TG lives in the morning builder block). **The Personal-project exception above (Step 6) is deliberate:** Personal items get scheduled + mirrored into Todoist because nothing else forces them.

**Key principle:** The weekly plan pulls from the *current quarter's* assigned projects, not the full roadmap/backlog. If the quarterly docket is empty or wrong, that's a signal to run the quarterly plan.

**Outputs:** Selection summary captured for Phase 8. Priority parent flagged. No Notion writes (Aaron set `This Week` manually in Step 1). **Personal-project work scheduled with Todoist mirrors (Step 6, created with approval).**

## Phase 4: CS Management (~10 min)

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

## Phase 5: Sales Management (~10 min)

**Purpose:** Drive new revenue and manage the full sales team's pipeline activity.

### Part A -- Aaron's Sales Activity

1. Review Pipedrive sales pipeline (pipeline 1): new leads, stale deals, overdue activities
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

## Phase 6: People Management (~10 min)

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

## Phase 7: Personal Life & Social (~5 min)

**Purpose:** Protect connection and enjoyment -- these don't get crowded out by work.

**Data pull:** Query Small Talk DB (`121f40c2-487b-802d`) for entries with Created Date in the past 7 days (already pulled in Phase 0). Count entries and note the most recent one.

1. **Parenting:** Custody schedule this week? Activities planned? Quality of recent time together?
2. **Social:** Report Small Talk count from past week and days since last entry.
   - Any casual social plans coming up? (friends, gym community, events, group activities)
   - If nothing planned and Small Talk count is low (<2 this week), suggest one low-effort thing
3. **Relationship integration check-in** (current status: in a relationship with Lexie — see [dating.md](../../self/dating.md)):
   - Present and integrated this week, or divided/hiding? (carry over any flag from Phase 1 Fuel Check)
   - Is the eros running clean — aimed outward and toward the relationship — or sliding toward secret-seeking?
   - Did the relationship support or erode the morning keystone, sleep, and Bus time?
   - Any work/relationship boundary concerns (fairness, operational distortion — she is also the ops manager)?
   - If the Phase 1 Fuel Check flagged contaminated/divided two weeks running, **name it directly here** and route back to the eros daily container ([eros.md](../../self/eros.md)).
4. **Compulsion scan:** Any obsessive patterns this week? (apps, substances, avoidance behaviors, re-forming a hidden space). Treat per the compulsive-transfer pattern in [capacity-rules.md](../../systems/capacity-rules.md) and the fuel-vs-compulsion line in [eros.md](../../self/eros.md).
5. **Personal enjoyment:** Anything purely fun on the calendar?

**Also check Values DB Health statuses.** If any category is Unhealthy, surface it here as a discussion point.

**Outputs:** Calendar blocks for personal time (use Personal Time Blocks calendar `10283d615faeb91862fc0ccd8f3ac216c7299a58f2196185e912be8f3e3cbe83@group.calendar.google.com`). Todoist reminders if needed.

## Phase 8: Commit (~5 min)

**Purpose:** Final review, capacity check, execute remaining actions, log everything.

1. **Summary table:** Everything planned across all phases (project selections, CS stops, sales stops, invoice escalations, photographer actions, 1:1s, personal items)
2. **Final capacity check:** Total planned hours vs. available hours. If total exceeds available, something must move. This is non-negotiable.
3. **Confirm "This Week" checkboxes:** Verify all selected Dev Projects have `This Week = true` and no deselected ones still have it checked.
4. **Store project KPIs on Weekly Meeting Log:** Write `Projects Completed` (count of projects marked Done this week) and `Projects In Progress` (count of projects with This Week checked for the new week).
5. **Store activity KPIs on Weekly Meeting Log:** Write `Aaron Activities`, `Lexie Activities`, `Tristen Activities`, `Ran Activities`, `Total Activities` (computed in Phase 2 Part A2).
6. **Append per-user Pipedrive detail sections** to the Weekly Meeting Log page using `personal_notion_append_blocks`. Use the Pipedrive data already pulled in Phase 0 (completed activities from the past 7 days + all open activities per user). Append the following structure:

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

7. **Record Starved Values:** If any value categories were flagged as off-track in Phase 1, log them as `Starved Values` on the Weekly Meeting Log entry (multi-select: Spirituality, Fitness, Work, Social, Admin, Parenting).
8. **Record Accomplishments (REQUIRED -- enables week-over-week throughput trend analysis):**
   - `Logged Accomplishments Count` = count of Dev Projects with Status -> Done in past 7 days (from Phase 2 Part B).
   - `Unlogged Accomplishments Count` = count of unlogged shipped slices Aaron flagged as real wins (from Phase 2 Part B2 sweep).
   - `Total Accomplishments Count` = Logged + Unlogged.
   - `Focused Output Hours Estimate` = best-effort total of focused work hours (logged Deep Work from Business Dev DB + Aaron-validated estimate of unlogged sweep effort). Be conservative -- this is a trend signal, not an exact measure.
   - `Accomplishments` (rich_text) = full narrative bullet list, grouped by Personal / Chrome Lot / Turbo Gear / Other-Infrastructure. Include both Dev Projects Done items and the Aaron-flagged unlogged slices. Format: `- [Domain] Item name (source: dev_projects | unlogged_sweep)`.
9. **Body comp already persisted.** Withings was written to Notion in Phase 0 (`withings-sync.mjs --days 14 --write`). Don't re-run it here.
10. **Execute remaining:** Create any Todoist/Calendar/Pipedrive/Notion items not yet committed during earlier phases.
11. **Log to Notion:** Finalize the Weekly Meeting Log entry (`322f40c2-487b-81bd`) with key decisions, action items, and plan summary.
12. **Update context files** if anything changed (people directory last-checkin dates, motorcycle ride date, etc.)

## Cross-Cutting Rules

- **Retire-a-slice (catch-up forcing).** Every weekly meeting must *retire* a defined slice of backlog from the active repair sprint - not just re-plan it. Name the slice in the CL Currency Check (Phase 2 Part A3) and confirm at commit (Phase 8) that it was archived / scheduled / assigned an owner. A meeting that only re-plans the same backlog has failed its core job during the reset period.
- **Route every item into a bucket.** Nothing leaves the meeting living only in Aaron's head - each surfaced item is Automated (n8n), Delegated (team 1:1s), or a Scheduled slice (calendar + Todoist mirror). See the Operating model section above.
- **Capacity is non-negotiable.** If total planned work exceeds available hours minus 10-15% buffer, the system pushes back. Something must move.
- **Delegation by default.** For any task deferred 3+ times, suggest delegation before rescheduling. Use the delegation framework in `context/systems/capacity-rules.md`.
- **Max 3 Pipedrive activities per day.** Cap at sustainable levels.
- **Max 5 must-do Todoist tasks per day.** If morning briefing shows >5, defer.
- **Pipedrive accountability is #1 neglected area.** Always check activities completed vs. planned.
- **All data pulls happen in Phase 0 silently.** The 60 minutes is for discussion and decisions.
- **Quarterly docket is the source.** Weekly project selection pulls only from the current quarter's assigned projects. Don't ad-hoc backlog items.

## Outputs

- **Pre-Phase 0:** Monthly plan gate pass (or full monthly plan run + resume).
- **Phase 0:** Silent pulls; reconciliation of completions; no user-facing deliverable beyond data readiness.
- **Phase 1:** PHQ-2, GAD-2, energy, and value-aligned pulse logged to Weekly Meeting Log; capacity adjustments noted.
- **Phase 1a:** Monthly alignment notes; flagged priorities for Phase 3 / Phase 8.
- **Phase 2:** Habit scorecard KPIs and Pipedrive activity totals written to Weekly Meeting Log; sub-item/project status updates in Dev Projects as decided.
- **Phase 3:** Selection summary and priority parent captured for Phase 8 (Aaron sets `This Week` in Notion).
- **Phase 4:** Pipedrive CS check-in and invoice escalation activities; optional Knack/Pipedrive updates with approval.
- **Phase 5:** Pipedrive sales activities; Todoist follow-ups as needed.
- **Phase 6:** Todoist, calendar, Knack (performance grade), Teams as applicable.
- **Phase 7:** Personal calendar blocks and Todoist reminders as needed.
- **Phase 8:** Full Weekly Meeting Log entry finalized; Team Activity Details appended; Starved Values; project counts; remaining Todoist/Calendar/Pipedrive/Notion items; context file updates if applicable.

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