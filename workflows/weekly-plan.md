> **Source:** [`context/skills/weekly-planning/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-planning/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Weekly Planning — SKILL

## Trigger

This skill activates when Aaron says "weekly plan", "weekly meeting", "plan this week", "sprint planning", or "Monday review". Target duration: ~73 minutes (mind/body ~18 min + social ~8 min before work).

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
- `context/self/eros.md` — primary fuel doctrine (Phase 1 Fuel Check, Phase 9 integration)
- `context/self/dating.md` — relationship status + integration guardrails + risk surface (Phase 9)
- `context/self/social.md` — sarges, Small Talk targets, isolation signals (Phase 2)
- **Planning context (canonical):** Monthly + Quarterly Meeting Log fields — pulled in Phase 0; see Phase 3. `context/self/current-priorities.md` is fallback only.
- `context/people/index.md` — delegation matrix, 1:1 tracking
- `context/work/chrome-lot/customer-service.md` — Phase 6 logic
- `context/work/chrome-lot/sales.md` — Phase 7 logic
- `context/work/chrome-lot/operations.md` — Phase 8 photographer review logic
- `context/work/turbo-gear/overview.md` — TG strategic sequence (for Phase 5 project selection)

## Interaction Style

- **One question at a time.** Never present a wall of choices. Walk through decisions sequentially.
- **Confirm before executing.** Each phase presents proposed actions, gets approval, then executes before moving on.
- **Exclude Shopping List** (Todoist project `6W36wRPXj8qC2RCc`) from all analysis.
- **Data integrity:** All Knack/Pipedrive/Notion/Todoist write operations require explicit user approval before execution (Todoist case-by-case - never batch or assume).

## Required Notion fields by section

Every phase that ends with log writes must populate its fields **before leaving the phase** (or confirm at Phase 10 gate). Agent presents a one-line **FIELD CHECK** checklist at each section boundary.

| Section | Phase | Required Weekly Meeting Log fields |
|---------|-------|-----------------------------------|
| Mind & Body | 1.1 | `Strength Sessions`, `Cardio Sessions`, `Spirit Minutes`, `Journal Count`, `Weight Avg`, `Body Fat Avg`, `Lean Mass Avg`, `Sleep Avg`, `Heart Rate Avg`, `Steps Avg`, `Workout Active Minutes` (+ RHR/HRV if available) |
| Mind & Body | 1.2 | `Wake Time Std Dev Min`, `Sleep Schedule Rating` |
| Mind & Body | 1.3 | `Intentions Review` |
| Mind & Body | 1.4 | `PHQ-2 Score`, `GAD-2 Score`, `PHQ-2 Severity`, `GAD-2 Severity`, `Energy Rating` |
| Mind & Body | 1.5 | (session state `weekly_life_health` — written Phase 10) |
| Mind & Body | 1.6 | `Strength Target`, `Cardio Target`, `Sleep Target Hours`, `Target Wake Time`, `Week Intentions`, `Behavioral Adjustments` |
| Social | 2 | `Small Talk Count`, `Social Events Count`, `Social Review`, `Social Intentions Met`, `Social Priority`, `Social Intentions` |
| Work review | 4 | `Deep Work Minutes`, `Ops Minutes`, `Field Work Minutes`, `Aaron/Lexie/Tristen/Ran/Total Activities` |
| Work review | 4 | `Accomplishments`, `Logged/Unlogged/Total Accomplishments Count`, `Focused Output Hours Estimate` (at Phase 10 if not earlier) |
| Commit | 10 | `Spirituality/Fitness/Work/Social/Admin/Parenting Health`, `Starved Values`, `Projects Completed`, `Projects In Progress`, `Key Decisions`, `Action Items` + **verify all rows above** |

**Gate rule:** Before Phase 3 (planning context), Phases 1 + 2 field rows must be complete or explicitly marked N/A with reason in the log.

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
4. **If entry exists:** Hold it for Phase 3. Continue to Phase 0.

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

### Social pulls (with wellness; used in Phase 2)

- **Small Talk DB** (`121f40c2-487b-802d`): all entries from **last 7 days** — list Description + Created Date (habit summary has count; Phase 2 needs the list).
- **Google Calendar (last 7 days):** Pull Aaron's primary + Personal Time Blocks calendars. Flag events that look social: friend hangouts, fitness classes, group events, Meetup, dates (non-work). Count → `Social Events Count`.

### Work pulls (after wellness/social data; silent until Phase 4+)

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

## Phase 1: Mind & Body Review (~18 min)

**Purpose:** Complete spiritual, mental, and physical review and planning **before any work content.** Highly structured — same sections every week.

Create the **new week's** Weekly Meeting Log entry at the start of Phase 1 (Name = `Week of [next Monday YYYY-MM-DD]`, Meeting Date = today). All Phase 1 fields write to this entry; KPI numbers from **last week** are copied from habit summary + health MCP into the structured properties during Phase 1.1.

### 1.1 Habit Scorecard + Insights (~5 min)

Present **last week's** metrics (from `weekly-habits-*.md` + `health_get_summary` last-7-day slice + prior-week log). Include a **4-week trend sidebar** from `weekly-wellness-trends-*.md`.

```
MIND & BODY SCORECARD — LAST WEEK
| Metric              | Last Wk | 4-wk trend        | Target   | Insight |
|---------------------|---------|-------------------|----------|---------|
| Spirit Minutes      |         | wk-4→wk-1 sparkline from log | 105-140 | |
| Strength Sessions   |         |                   | 5/week (adjust in 1.6) | |
| Cardio Sessions     |         |                   | —        | |
| Sleep Avg (hours)   |         |                   | 7-8      | |
| Deep/REM (optional) |         | from daily sleep  | —        | |
| Weight / BF / Lean  |         |                   | hold/grow| |
| Steps Avg/day       |         |                   | 8000+    | |
| Journal Count       |         |                   | —        | |
| PHQ-2 / GAD-2 / Energy | from prior log if not yet re-screened | 4-wk from trends file | — | |
```

**Insights (required):** After the table, state 2-3 bullet observations — not just numbers. Examples: spirit minutes up but sleep still under 6h; strength hit target but cardio absent 3 weeks; weight trend favorable but energy score dropping.

Write last week's computed KPIs to the Weekly Meeting Log entry: Strength Sessions, Cardio Sessions, Spirit Minutes, Journal Count, Weight/Body Fat/Lean Avg, Sleep Avg, Heart Rate Avg, Resting HR Avg, HRV Avg, Steps Avg, Workout Active Minutes.

### 1.2 Sleep & Wake Schedule (~4 min)

**Purpose:** Flag erratic wake times and poor sleep quality before planning the week.

From `health_get_summary.daily` last 7 days with `wake_time` / `wake_minutes`:

```
SLEEP & WAKE — LAST 7 NIGHTS
| Date       | Sleep h | Wake time | Bedtime | Deep min | Awake min |
|------------|---------|-----------|---------|----------|-----------|
| (each day) |         |           |         |          |           |
```

Compute **wake time std dev** (minutes) across nights with data. Classify `Sleep Schedule Rating`:
- **Consistent** — σ ≤ 30 min
- **Moderate Variance** — σ 31–60 min
- **Erratic** — σ > 60 min
- **Unknown** — fewer than 4 nights with wake data

Present one insight line: e.g. "Wake times ranged 5:12–8:47 (σ 94 min) — schedule is erratic; sleep avg 5.3h is below target."

Store on Weekly Meeting Log: `Wake Time Std Dev Min`, `Sleep Schedule Rating`.

### 1.3 Last Week Intentions Review (~3 min)

Read **prior week's** `Week Intentions` and `Behavioral Adjustments` from `weekly-wellness-trends-*.md` (or prior log entry).

```
INTENTIONS REVIEW — PRIOR WEEK
| Intention (from last Week Intentions) | Evidence | Met? |
|---------------------------------------|----------|------|
| (each line)                           |          | ✓/~/✗ |
```

Compare against habit scorecard + Dev Projects completions. Ask Aaron one AskQuestion: overall — **Mostly met / Partially met / Missed / N/A (none set)**.

Write `Intentions Review` (rich_text) on the **new** week's log: bullet list of each intention + outcome.

### 1.4 Wellness Screening (~3 min)

**PHQ-2** (0-3 each, total 0-6) and **GAD-2** (0-3 each, total 0-6) — one question at a time via AskQuestion.

**Energy & Capacity** — 1-10 scale.

Derive severity selects (None / Mild / Moderate / Moderately Severe / Severe) per standard cutoffs. Write PHQ-2, GAD-2, Energy, severity selects to Weekly Meeting Log immediately.

### 1.5 Values Pulse + Life Health + Fuel (~4 min)

**Manual Values Review:** Link to [Values Database](https://www.notion.so/342f40c2487b80c5a2aee48ca48b4a20). Wait for confirmation.

Present Values Pulse table (spirituality/fitness focus — work metrics deferred to Phase 4; social deep-dive in Phase 2):

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
- **Fitness** -> Workouts count + Body Comp + Recovery callout from `health_get_summary({ days: 28 })`. Format:
  - `Body comp: 7d avg <X> lbs (Δ<±Y> lbs) · Body Fat <X>% (Δ<±Y>%) · Lean <X> lbs (Δ<±Y> lbs)` (from `stats.weight_lbs.avg`, `trend.weight_lbs`, etc.)
  - `Recovery: Sleep avg <X.X>h (Δ<±Y>h) · HR avg <X> bpm (Δ<±Y>) · RHR avg <X> bpm (Δ<±Y>) · HRV avg <X> ms (Δ<±Y>)` (from `stats.sleep_hours.avg`, `trend.sleep_hours`, `stats.heart_rate_avg_bpm.avg`, `trend.heart_rate_avg_bpm`, `stats.resting_hr.avg`, `trend.resting_hr`, `stats.hrv_rmssd.avg`, `trend.hrv_rmssd`). RHR + HRV remain null until those Health Sync folders are enabled -- render them as `--` until then.
  - `Activity: Steps avg <X>/day (Δ<±Y>) · Workouts <N> sessions / <M> active min (Δ<±workout_active_minutes>)` (from `stats.steps.avg`, `trend.steps`, `stats.workout_count.total`, `stats.workout_active_minutes.total`, `trend.workout_active_minutes`)
  - If `sources.health_sync.ok = false`, drop the Recovery + Activity lines and append "(Health Sync data missing -- run `npm test` in `mcp/health-data` to diagnose)". If `sources.notion.error`, surface body-comp blank as "Withings sync needs attention".
- **Work** -> Business Development (`127f40c2-487b-803e`) Total Time sum + Chrome Lot Ops (`136f40c2-487b-80ba`) Total Time sum + Field Work (`237f40c2-487b-80ab`) Total Time sum (all three are formula-typed numeric properties)
- **Social** -> Small Talk (`121f40c2-487b-802d`) count
- **Admin, Parenting** -> no direct time KPI (qualitative reflection)

**Life Health Rating:** One AskQuestion per category — Healthy or Unhealthy. Store in `weekly_life_health` for Phase 10 commit. Social evidence from Small Talk count is preliminary — Phase 2 deepens.

**Fuel Check (eros):** Per [eros.md](../../self/eros.md). If contaminated/divided two weeks running (check 4-week trends), flag for Phase 9.

**Capacity gate:** If PHQ-2 ≥ 3 or GAD-2 ≥ 3 or Energy ≤ 4, note reduced work capacity before Phase 4.

### 1.6 Adjustments & Weekly Targets (~3 min)

**Purpose:** Plan concrete mind/body adjustments for the **upcoming** week before touching work.

Propose based on 1.1–1.5 evidence (sleep erratic → fixed wake target; missed strength → realistic floor; low spirit → protected block):

1. **Workout targets** — AskQuestion: Strength sessions target (default 4–5)? Cardio sessions target (0–2)?
2. **Sleep targets** — Target wake time (e.g. `6:30 AM CT`) and sleep hours target (default 7–7.5h).
3. **Calendar adjustments** — Propose specific blocks: recovery time, long cardio session, early bedtime wind-down, PTO/rest half-day if wellness screening flagged high anxiety/low energy.
4. **Week Intentions** — 3–5 bullets (mind/body + max 1–2 personal; **social → Phase 2**; work → Phase 5).

Write to Weekly Meeting Log:
- `Strength Target`, `Cardio Target`, `Sleep Target Hours`, `Target Wake Time`, `Week Intentions`, `Behavioral Adjustments`

**Outputs:** Present **FIELD CHECK — Phase 1** against the Mind & Body rows in the table above. Mind/body plan captured on Weekly Meeting Log. **Do not proceed to Phase 2 until complete.**

## Phase 2: Social Review & Planning (~8 min)

**Purpose:** Review social connectedness **before work.** Per [social.md](../../self/social.md) — target 3–5 Small Talk entries/week; isolation signal is days since last entry, not dating status. Social may be knowingly **deprioritized** (parenting, girlfriend, work crunch) — capture that explicitly, don't treat it as failure.

Load: Small Talk entries (Phase 0), last week's calendar social events (Phase 0), prior week's `Social Intentions` + `Social Review` from `weekly-wellness-trends-*.md` or prior log.

### 2.1 Last Week — What Happened (~3 min)

```
SOCIAL REVIEW — LAST WEEK
| Signal | Count / Detail |
|--------|----------------|
| Small Talk entries | N (list each: date — description) |
| Calendar social events | N (list: date — event title) |
| Days since last Small Talk | N |
| 4-wk Small Talk trend | from weekly-wellness-trends |
```

Ask one open question: **"Any social contact last week that didn't get logged — gym conversations, work hangouts, time with Lexie/Bus friends, etc.?"** Add missed items to the narrative (not necessarily new DB rows unless Aaron wants them logged).

Write `Small Talk Count`, `Social Events Count`, `Social Review` (rich_text bullet narrative: what happened, what was missed, quality not just quantity).

### 2.2 Prior Social Intentions (~2 min)

If prior week had `Social Intentions`:

```
SOCIAL INTENTIONS REVIEW
| Intention | Evidence | Met? |
|-----------|----------|------|
| (each line from prior Social Intentions) | | ✓ / ~ / ✗ |
```

AskQuestion (single): **Mostly met / Partially met / Missed / Deprioritized / N/A**

Write `Social Intentions Met` select. If intentions were missed because Aaron chose parenting/work/Lexie time, use **Deprioritized** — not Missed.

### 2.3 Next Week — Goal & Priority (~2 min)

AskQuestion (single): **Social priority this week?**
- **Active** — pursuing 3–5 sarges / social day; pre-commit to events
- **Maintenance** — keep existing plans, no new outreach push
- **Deprioritized** — knowingly low; name why (parenting / girlfriend / work / recovery)

Based on priority + last week's evidence, propose **one concrete social goal** for the upcoming week. Examples:
- Active: "2 Small Talk entries + 1 fitness class pre-booked"
- Maintenance: "Show up to existing Saturday coffee plan"
- Deprioritized: "No social target — Bus week + custody email only"

Write `Social Priority` + `Social Intentions` (rich_text, 1–3 bullets for upcoming week).

### 2.4 Pre-Commit (conditional, ~1 min)

**Only when `Social Priority` = Active** (or Maintenance with zero events on calendar):

AskQuestion: **Pre-commit tactic this week?**
- Check Meetup for one event → calendar block
- Book a fitness class (gym schedule) → calendar block
- Schedule coffee/social day block (Personal Time Blocks calendar)
- Skip pre-commit — rely on organic opportunities
- N/A — deprioritized

Execute with approval: calendar events on Personal Time Blocks calendar (`10283d615faeb91862fc0ccd8f3ac216c7299a58f2196185e912be8f3e3cbe83@group.calendar.google.com`) or Todoist reminder. Append booked events to `Social Intentions`.

**Outputs:** Present **FIELD CHECK — Phase 2** against Social rows. **Do not proceed to Phase 3 (work/planning) until Phases 1 + 2 fields are complete.**

## Phase 3: Planning Context Review (~3 min)

**Purpose:** Ground the week in committed quarter + month context before tactical project selection. **Do not infer priorities from skill text or `current-priorities.md` when log fields are populated.**

Present from Phase 0 pulls:

```
PLANNING CONTEXT — week of [YYYY-MM-DD]
| Source | Field | Value |
|--------|-------|-------|
| Quarterly ([Q# YYYY]) | Priority Stack | numbered list from Quarterly Meeting Log |
| Quarterly | Domains Parked | multi-select (empty = nothing parked at quarter level) |
| Monthly ([planning month]) | Priority Stack | numbered list from review-month Monthly Log |
| Monthly | Domains Parked | multi-select — **authoritative for what's on pause this week** |
| Monthly | Active CL Sprint | select — current CL repair sprint (A–E or Maintenance) |
| Monthly | Action Items | bullet summary (planning month commitments) |
| Monthly | Key Wins / Misses | one line each from review month close-out |
```

**Enforcement rules (read aloud if violated):**
- If `Domains Parked` includes **Turbo Gear**, do not select TG Dev Projects in Phase 5 unless Aaron explicitly overrides this week.
- `Active CL Sprint` drives Phase 4 CL Currency Check — name the sprint from the log, not from memory.
- If `Priority Stack` or `Domains Parked` is empty on the Monthly Log, fall back to Quarterly Log, then `context/self/current-priorities.md`, and flag: "Planning context incomplete — rerun monthly Phase 11b next month."

Ask via AskQuestion (multi-select): "Anything from the planning context that must get a slice **this** week?" Options: one per `Priority Stack` line + one per distinct `Action Items` theme + "None — plan is on track as written."

**Outputs:** Flagged priorities carried into Phase 5 (One Thing check, capacity sanity) and Phase 10 commit. No Notion writes.

## Phase 4: Work Review (~10 min)

**Purpose:** Review last week's work output and operating currency. Mind/body scorecard is in Phase 1 — do not repeat.

### Part A -- Work Habit Metrics (last week)

| Metric | Last Wk | Notes |
|--------|---------|-------|
| Deep Work Minutes | | |
| Ops Minutes | | Monitor vs dev crowding |
| Field Work Minutes | | |
| Small Talk Count | | |

Write to Weekly Meeting Log if not already set.

### Part B -- Pipedrive Activity Scorecard

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

### Part C -- Chrome Lot Currency Check (catch-up forcing)

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

Then state the **active repair sprint** from the Monthly Log `Active CL Sprint` field (sequence: A Pipedrive+Todoist → B 1:1 cadence → C customer service → D photographer performance → E sales → Maintenance). Name the specific backlog slice this meeting will retire. This feeds the retire-a-slice rule at commit.

### Part D -- Project Review

Pull Dev Projects where `This Week = true` (last week's selections, including sub-items):
1. Group items by top-level parent. For each parent, list selected sub-items + their statuses.
2. Ask Aaron for a quick narrative per parent: which sub-items got done, which didn't move, why?
3. **Sub-items are the unit of completion.** Mark Done sub-items with `Status = Done`. Their `This Week = true` stays as historical record OR can be unchecked -- Aaron's choice. Phase 5's manual selection is what defines next week's `This Week` set.
4. **Parent-level closure:** A parent is only marked `Status = Done` when ALL its sub-items are Done. Otherwise it stays In progress and continues across weeks naturally.
5. For sub-items that didn't move at all and were carryover from a previous week, decide: keep, defer, delegate, or drop. If deferred 3+ times, route through the delegation framework before keeping.

**Outputs:** Sub-item Status updates where applicable. Aaron resets `This Week` during Phase 5 in Notion.

### Part E -- Unlogged Accomplishments Sweep

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

**Outputs:** Combined accomplishment count for capacity calibration in Phase 5. Store on Weekly Meeting Log `Accomplishments` at Phase 10.

## Phase 5: Project Selection for This Week (~12 min)

**Purpose:** Choose this week's focus projects from the current quarter's docket, accounting for carryover work.

**Why manual selection in Notion:** Most of Aaron's parent projects are multi-week containers with their own sub-item hierarchies already broken out in the Dev Projects DB. The natural unit of weekly work is a **specific sub-item**, not a whole parent. A Q&A-driven selection flow forces the agent to redundantly model that hierarchy in chat. Faster: Aaron toggles `This Week` directly on the projects/sub-items he wants in Notion, then the agent reads the result and confirms.

### Step 1: Pause and Direct to Notion
Tell Aaron: "Open the Dev Projects DB and toggle `This Week = true` on every project AND sub-item you intend to work on this week. Pick parents only when you mean the whole project; pick specific sub-items when you only want a slice. Carryover (anything still `This Week = true` from last week) is already accounted for. Tell me when you're done."

Direct link: [Dev Projects](https://www.notion.so/341f40c2487b80acae1fd344d334096c)

Wait for Aaron's confirmation before continuing. **Do NOT use AskQuestion to enumerate parents/sub-items**; that's what this step replaces.

### Step 2: Pull the Selection
Once confirmed, read the **"This Week — actionable slate"** section from `output/weekly-habits-YYYY-MM-DD.md` (already produced in Phase 0). Do not re-query raw `This Week` counts — the script filters Done items and Done-parent children. Then:
- Group results by top-level parent (use the `Parent item` relation; if empty the item is itself a top-level parent).
- For each parent, show its `Type`, `Status`, `Completion`, `Due Date`, and the list of selected sub-items beneath it.
- Show a domain count summary: how many items per Type (Personal / Chrome Lot / Turbo Gear).

### Step 3: One Thing Check
Ask via AskQuestion: "Of everything you've marked, which **one** parent project (if you finish its slice this week) makes the most other stuff easier or irrelevant?" Single-select from the parents present. Flag the answer as the priority for the week and capture in the Weekly Meeting Log's Key Decisions.

### Step 4: Capacity Sanity Check
Total selected sub-items across all parents. Heuristic: each sub-item averages 2-4 hours of focused work. If total > 12 sub-items or projected hours > available capacity (~25h baseline minus trip/PTO time), recommend trimming a slice before proceeding. **Capacity is non-negotiable** -- if Aaron says "keep it all," push back once with a specific cut suggestion before deferring to him.

### Step 5: Confirm and Move On
No further Notion writes are needed in this phase -- Aaron has already set `This Week = true` directly. The agent's only write here is logging the priority project to the Weekly Meeting Log in Phase 10.

### Step 6: Personal Project Scheduling + Todoist Mirrors
Personal-type projects (custody modification, taxes / Bench->QuickBooks, personal dev, admin) lack the forcing function that Chrome Lot work gets from the 9am Pipedrive->Todoist sync and that Turbo Gear gets from the protected morning builder block. Left untracked, they drift. So for the **Personal** sub-items selected this week:
1. With Aaron, decide **when** in the week he'll work each one (day/slot), respecting capacity caps and protected blocks.
2. **Propose a Todoist mirror** for each (e.g., project Office / Same Day To Do) with the chosen due date so it surfaces in daily execution. Honor the 5-must-do/day cap.
3. **Custody is kept in bounds here** — scheduled as one or two concrete tasks in its planned slot, not left as a floating worry. This replaces any standalone recurring custody block. (Custody source: `context/family/custody/index.md`.)
4. **Todoist writes require case-by-case approval** (per `context/rules.md`) — propose the mirrors, create on Aaron's go.

**Note:** Todoist is NOT used for Chrome Lot or Turbo Gear Dev Project tracking — those stay at the Notion project level (CL also surfaces via the 9am Pipedrive->Todoist sync; TG lives in the morning builder block). **The Personal-project exception above (Step 6) is deliberate:** Personal items get scheduled + mirrored into Todoist because nothing else forces them.

**Key principle:** The weekly plan pulls from the *current quarter's* assigned projects, not the full roadmap/backlog. If the quarterly docket is empty or wrong, that's a signal to run the quarterly plan.

**Outputs:** Selection summary captured for Phase 10. Priority parent flagged. **Personal-project work scheduled with Todoist mirrors (Step 6, created with approval).**

## Phase 6: CS Management (~10 min)

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

## Phase 7: Sales Management (~12 min)

**Purpose:** Drive new revenue and manage the full sales team's pipeline activity.

### Part A -- Aaron's Sales Activity

**A0 — Deal gaps cleanup (mandatory, before planning stops)**

1. Pre-pull Aaron-owned open deals in Sales (pipeline 1), CS (pipeline 6), and Social Media (pipeline 13) with **no `next_activity_date`** via Pipedrive MCP. Present as a numbered checklist.
2. Instruct Aaron to run CL Bot **`deal gaps`** in Teams ([`cl-bot.md`](../../systems/notion-guides/cl-bot.md)) and schedule every gap via the Adaptive Card **Schedule** buttons (+3 days default).
3. **Gate:** Do not plan new sales stops until gaps = 0. If Aaron explicitly defers a deal, log the deal name + reason in Phase 10 `Key Decisions` and exclude it from the gap count.

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

## Phase 8: People Management (~10 min)

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

## Phase 9: Personal Life (~5 min)

**Purpose:** Parenting, relationship integration, compulsion scan — **social connectedness is Phase 2.** Don't repeat Small Talk review here.

1. **Parenting:** Custody schedule this week? Activities planned? Quality of recent time together?
2. **Relationship integration check-in** (current status: in a relationship with Lexie — see [dating.md](../../self/dating.md)):
   - Present and integrated this week, or divided/hiding? (carry over any flag from Phase 1 Fuel Check)
   - Is the eros running clean — aimed outward and toward the relationship — or sliding toward secret-seeking?
   - Did the relationship support or erode the morning keystone, sleep, and Bus time?
   - Any work/relationship boundary concerns (fairness, operational distortion — she is also the ops manager)?
   - If the Phase 1 Fuel Check flagged contaminated/divided two weeks running, **name it directly here** and route back to the eros daily container ([eros.md](../../self/eros.md)).
3. **Compulsion scan:** Any obsessive patterns this week? (apps, substances, avoidance behaviors, re-forming a hidden space). Treat per the compulsive-transfer pattern in [capacity-rules.md](../../systems/capacity-rules.md) and the fuel-vs-compulsion line in [eros.md](../../self/eros.md).
4. **Personal enjoyment:** Anything purely fun on the calendar beyond what Phase 2 scheduled?

**Also check Values DB Health statuses.** If any category is Unhealthy, surface it here as a discussion point.

**Outputs:** Calendar blocks for personal time (use Personal Time Blocks calendar `10283d615faeb91862fc0ccd8f3ac216c7299a58f2196185e912be8f3e3cbe83@group.calendar.google.com`). Todoist reminders if needed.

## Phase 10: Commit (~5 min)

**Purpose:** Final review, capacity check, execute remaining actions, log everything.

1. **Summary table:** Everything planned across all phases (project selections, CS stops, sales stops, invoice escalations, photographer actions, 1:1s, personal items)
2. **Final capacity check:** Total planned hours vs. available hours. If total exceeds available, something must move. This is non-negotiable.
3. **Confirm "This Week" checkboxes:** Verify all selected Dev Projects have `This Week = true` and no deselected ones still have it checked.
4. **Store project KPIs on Weekly Meeting Log:** Write `Projects Completed` (count of projects marked Done this week) and `Projects In Progress` (count of projects with This Week checked for the new week).
5. **Store activity KPIs on Weekly Meeting Log:** Write `Aaron Activities`, `Lexie Activities`, `Tristen Activities`, `Ran Activities`, `Total Activities` (computed in Phase 4 Part B).
6. **Verify wellness + social fields (REQUIRED):** Cross-check **Required Notion fields by section** table — Phases 1, 2, and 4 rows must be populated. Explicitly confirm: `Week Intentions`, `Intentions Review`, targets, sleep fields, `Social Review`, `Social Intentions`, `Social Priority`, `Social Intentions Met`, `Small Talk Count`, `Social Events Count`.
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

8. **Record life health ratings (REQUIRED):** Write all 6 select properties from Phase 1.5 (`Spirituality Health`, `Fitness Health`, `Work Health`, `Social Health`, `Admin Health`, `Parenting Health`) on the Weekly Meeting Log entry. Values: `Healthy` or `Unhealthy`.
9. **Update Values DB Health (with approval):** For each category where Phase 1 rating differs from current Values DB Health, update via `personal_notion_update_page` on the category page in Values DB (`342f40c2-487b-80c5`).
10. **Record Starved Values:** Derive from life health ratings — set `Starved Values` multi_select to every category rated **Unhealthy** (Spirituality, Fitness, Work, Social, Admin, Parenting). Do not use a separate "felt off-track" question; health ratings are the source of truth.
11. **Record Accomplishments (REQUIRED -- enables week-over-week throughput trend analysis):**
   - `Logged Accomplishments Count` = count of Dev Projects with Status -> Done in past 7 days (from Phase 4 Part D).
   - `Unlogged Accomplishments Count` = count of unlogged shipped slices Aaron flagged as real wins (from Phase 4 Part E sweep).
   - `Total Accomplishments Count` = Logged + Unlogged.
   - `Focused Output Hours Estimate` = best-effort total of focused work hours (logged Deep Work from Business Dev DB + Aaron-validated estimate of unlogged sweep effort). Be conservative -- this is a trend signal, not an exact measure.
   - `Accomplishments` (rich_text) = full narrative bullet list, grouped by Personal / Chrome Lot / Turbo Gear / Other-Infrastructure. Include both Dev Projects Done items and the Aaron-flagged unlogged slices. Format: `- [Domain] Item name (source: dev_projects | unlogged_sweep)`.
12. **Body comp already persisted.** Withings written in Phase 0 (`--days 28`). Don't re-run here.
13. **Execute remaining:** Create any Todoist/Calendar/Pipedrive/Notion items not yet committed during earlier phases.
14. **Log to Notion:** Finalize the Weekly Meeting Log entry (`322f40c2-487b-81bd`) with key decisions, action items, and plan summary.
15. **Update context files** if anything changed.

## Cross-Cutting Rules

- **Mind/body + social before work.** Phases 0b–2 must complete before Phase 3 (planning context) or any work phase. No work discussion during Phases 1–2.
- **FIELD CHECK gates.** Present the section checklist from the table before leaving Phases 1, 2, and 10.
- **Retire-a-slice (catch-up forcing).** Name the slice in Phase 4 CL Currency Check; confirm at Phase 10 commit it was archived / scheduled / assigned an owner.
- **Route every item into a bucket.** Each surfaced item is Automated (n8n), Delegated (team 1:1s), or a Scheduled slice (calendar + Todoist mirror).
- **Capacity is non-negotiable.** If total planned work exceeds available hours minus 10-15% buffer, the system pushes back. Something must move.
- **Delegation by default.** For any task deferred 3+ times, suggest delegation before rescheduling. Use the delegation framework in `context/systems/capacity-rules.md`.
- **Max 3 Pipedrive activities per day.** Cap at sustainable levels.
- **Max 5 must-do Todoist tasks per day.** If morning briefing shows >5, defer.
- **Pipedrive accountability is #1 neglected area.** Always check activities completed vs. planned.
- **All data pulls happen in Phase 0 silently.** The 60 minutes is for discussion and decisions.
- **Quarterly docket is the source.** Weekly project selection pulls only from the current quarter's assigned projects. Don't ad-hoc backlog items.

## Outputs

- **Pre-Phase 0:** Monthly plan gate pass (or full monthly plan run + resume).
- **Phase 0:** Wellness + social + work data pulls; trend files; 4-week log history.
- **Phase 0b:** Data integrity table; remediation before Phases 1–2.
- **Phase 1:** Mind/body review + targets on Weekly Meeting Log.
- **Phase 2:** Social review + intentions + optional pre-commit (Meetup / fitness class).
- **Phase 3:** Planning context table; flagged priorities for Phase 5 / Phase 10.
- **Phase 4:** Work metrics, Pipedrive, CL currency, projects, accomplishments.
- **Phase 5:** Project selection + Todoist mirrors.
- **Phase 6–9:** CS, sales, people, personal (parenting/relationship/compulsion).
- **Phase 10:** Full Weekly Meeting Log finalized + FIELD CHECK all sections; Values DB sync (with approval).

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