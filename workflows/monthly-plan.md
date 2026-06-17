> **Source:** [`context/skills/monthly-plan/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/monthly-plan/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Monthly Plan — SKILL

## Table of contents

- [Trigger](#trigger)
- [Month framing (non-negotiable)](#month-framing-non-negotiable)
- [Inputs](#inputs)
- [Execution Protocol (mandatory)](#execution-protocol-mandatory)
  - [Ledger step order (do not reorder)](#ledger-step-order-do-not-reorder)
  - [Present only (by step)](#present-only-by-step)
  - [Output contracts — Phases 3–11](#output-contracts-phases-311)
- [Interaction Style](#interaction-style)
- [Procedure](#procedure)
- [Phase 0: Data Pull (silent, before conversation)](#phase-0-data-pull-silent-before-conversation)
- [Phase 1: Wellness Trends (~3 min)](#phase-1-wellness-trends-3-min)
- [Phase 1b: Identity Check (~5 min)](#phase-1b-identity-check-5-min)
- [Phase 1c: Quarterly Plan Checkpoint (~5 min)](#phase-1c-quarterly-plan-checkpoint-5-min)
  - [Quarterly plan completeness gate](#quarterly-plan-completeness-gate)
  - [Quarterly progress review (run only when gate passes)](#quarterly-progress-review-run-only-when-gate-passes)
- [Phase 1d: Sustained Unhealthy Gate (~10 min, conditional)](#phase-1d-sustained-unhealthy-gate-10-min-conditional)
- [Phase 2: Fitness Review & Reprioritization (~5 min)](#phase-2-fitness-review-and-reprioritization-5-min)
- [Phase 3: Dev Work Review & Goals (~10 min)](#phase-3-dev-work-review-and-goals-10-min)
- [Phase 3b: Idea Roadmap Scrub (~8 min)](#phase-3b-idea-roadmap-scrub-8-min)
  - [Procedure (run once per Type, in this order: TG → CL → Personal)](#procedure-run-once-per-type-in-this-order-tg-cl-personal)
  - [Sanity guardrails](#sanity-guardrails)
- [Phase 4: Sales Progress & Goals (~10 min)](#phase-4-sales-progress-and-goals-10-min)
  - [Pipeline Health](#pipeline-health)
  - [Activity Scorecard](#activity-scorecard)
  - [Team Efficiency Analysis](#team-efficiency-analysis)
  - [Goal Setting](#goal-setting)
  - [Pipedrive Reference](#pipedrive-reference)
  - [Hubstaff Project Reference](#hubstaff-project-reference)
- [Phase 5: Personal Project Tracker Audit (~10 min)](#phase-5-personal-project-tracker-audit-10-min)
- [Phase 6: Financial Review (~10 min)](#phase-6-financial-review-10-min)
  - [Personal Finances](#personal-finances)
  - [Chrome Lot Business Finances](#chrome-lot-business-finances)
- [Phase 7: CS Health Monthly Review (~5 min)](#phase-7-cs-health-monthly-review-5-min)
- [Phase 8: People & Operations Trends (~5 min)](#phase-8-people-and-operations-trends-5-min)
- [Phase 8b: Work Domain Health Rating (~5 min)](#phase-8b-work-domain-health-rating-5-min)
- [Phase 9: Quarterly KPI Update (~5 min)](#phase-9-quarterly-kpi-update-5-min)
- [Phase 10: Trip Planning with Bus (~5 min)](#phase-10-trip-planning-with-bus-5-min)
- [Phase 11: Personal Time Off (~3 min)](#phase-11-personal-time-off-3-min)
- [Phase 11b: Planning Month Dev Projects (~5 min)](#phase-11b-planning-month-dev-projects-5-min)
- [Phase 12: Commit (~3 min)](#phase-12-commit-3-min)
- [Cross-Cutting Rules](#cross-cutting-rules)
- [Outputs](#outputs)
- [Failure modes & graceful degradation](#failure-modes-and-graceful-degradation)
- [See also](#see-also)

---


## Trigger {#trigger}

This skill activates when Aaron says "monthly plan", "monthly review", or "plan this month". Target duration: ~83 minutes.

## Month framing (non-negotiable) {#month-framing-non-negotiable}

The monthly plan is **forward-looking**. Aaron **plans the current calendar month** while **reviewing the prior calendar month**.

Example: session on 2026-06-05 → **plan June**, **review May**.

| Lens | Which month | Used in |
|------|-------------|---------|
| **Review** | Prior calendar month | Phases 1–2 wellness/fitness; Phases 4–8 business retros; Phase 12 Monthly Log metrics (the month being closed out) |
| **Planning** | Current calendar month | Phases 3–4 goals; Phases 10–11 Bus trip + PTO; Action Items committed ahead |

At session start, compute both from `Today's date` (America/Chicago):

- **Review month** = first day of prior month through last day of prior month
- **Planning month** = first day of current month through last day of current month

Phase 0 data pulls:

- Weekly Meeting Log entries with Meeting Date in **review month**
- Pipedrive/Knack/Hubstaff aggregates for **review month**
- `health_get_summary` slices: **review month** avg vs month before review month (MoM baseline)
- MoM baseline = prior **Monthly Plan Log** entry titled `Monthly Plan for [Review Month YYYY]` (when reviewing May and planning June, baseline is `Monthly Plan for May 2026`, which holds April retrospective KPIs)

Phase 12 writes two Notion artifacts:

1. **Monthly Plan Log** (one entry per session): Name = `Monthly Plan for [Planning Month YYYY]` — covers **review month** retrospective KPIs + **planning month** forward commitments in one record. `Month` relation = **planning month** Months record.
2. **Months DB plan log** (planning month hub): Append formatted session summary to the **planning month** Months page (see Phase 12 step 6). Do not skip even when the Monthly Plan Log entry already exists.

Months page title resolution (DB `121f40c2-487b-80f2`): prefer `[FullMonthName] [YYYY]` (e.g. `June 2026`); fall back to month name only (`May`, `June`) when automation used the short form.

## Inputs {#inputs}

Load via the router. Read these before starting:

- `context/systems/cadences.md` — cadence health check
- `context/systems/capacity-rules.md` — limits, overcommitment triggers
- `context/systems/notion-databases.md` — Weekly Meeting Log, Monthly Plan Log, Dev Projects, Quarterly Outcomes, Values, Workouts, Small Talk, Health Data DB schemas + IDs
- `context/systems/pipedrive.md` — pipeline IDs, user IDs, completion-time API quirks
- `context/systems/knack-fields.md` — Customer + Invoice field references for Phases 6, 7
- `context/systems/hubstaff.md` — member IDs, project IDs (3563292, 3563452 for sales efficiency), monthly hours
- `context/systems/health-data.md` — MCP for body comp + watch metrics
- `context/self/values.md` — six categories with Picture of Success (Phase 1b Identity Check)
- `context/self/current-priorities.md` — what's hot this quarter
- `context/people/index.md` — delegation, 1:1 tracking, roster (Phase 8 reconciliation)
- `context/work/chrome-lot/overview.md` + `customer-service.md` — Phases 6, 7
- `context/work/turbo-gear/overview.md` — Phase 3 dev work review
- `context/skills/quarterly-plan/SKILL.md` — Phase 1c quarterly gate + escalation path

## Execution Protocol (mandatory) {#execution-protocol-mandatory}

Read `context/workflow-execution.md`, `context/systems/workflow-output-contracts.md`, and `context/systems/workflow-logs.md`.

1. `node scripts/workflow-progress.mjs init --workflow monthly-plan`
2. Step `0` — silent Phase 0 pulls
3. Step `1.0` — `node scripts/workflow-notion-log.mjs create --ledger <path>` (Session Complete = Incomplete)
4. Every turn: `workflow-progress.mjs status` → present only `current_step` tables → Aaron confirms → `advance` → `workflow-notion-log sync`
5. Before step `2.1`: `gate --phase 2` (requires `1.check` pass)
6. Before step `12.0`: `gate --phase 12`
7. Step `12.0` — `workflow-notion-log complete`

### Ledger step order (do not reorder) {#ledger-step-order-do-not-reorder}

| Step | Skill phase | User-facing? |
|------|-------------|--------------|
| `0` | Phase 0 Data Pull | Silent |
| `1.0` | Create Monthly Plan Log entry shell | Yes |
| `1.1` | Phase 1 Wellness Trends | Yes — Table 1.1-A, 1.1-B |
| `1.2` | Phase 1b Identity Check | Yes — one category per turn |
| `1.3` | Phase 1c Quarterly gate | Yes — Table 1.3-A |
| `1.4` | Phase 1d Sustained unhealthy (conditional) | Yes or N/A |
| `1.check` | Phase 1 FIELD CHECK | Yes — Table 1.check |
| `2.1` | Phase 2 Fitness | Yes — Table 2.1-A |
| `3.1` | Phase 3 Dev goals | Yes |
| `3.2` | Phase 3b Idea scrub | Yes — one Type per turn |
| `4.1`–`11.2` | Phases 4–11b | Yes — per Present map below |
| `12.0` | Phase 12 Commit | Yes — Table 12.check + writes |

### Present only (by step) {#present-only-by-step}

| Step | Present exactly |
|------|-----------------|
| `1.1` | Table 1.1-A (wellness trajectory), then Table 1.1-B (life health trajectory) — one table per turn |
| `1.2` | Table 1.2-A (one Values category per turn) |
| `1.3` | Table 1.3-A (quarterly gate) |
| `1.4` | Table 1.4-A (sustained unhealthy) or skip with N/A |
| `1.check` | Table 1.check |
| `2.1` | Table 2.1-A (body comp + watch MoM) |
| `3.1` | Table 3.1-A (dev goals + quarterly progress) |
| `3.2` | Table 3.2-A — one roadmap Type (TG → CL → Personal) per turn |
| `4.1` | Table 4.1-A (activity scorecard), then Table 4.1-B (team efficiency) |
| `5.1` | Table 5.1-A (personal project audit — one decision per turn) |
| `6.1` | Table 6.1-A (personal finances), then Table 6.1-B (CL finances) |
| `7.1` | Table 7.1-A (CS health monthly) |
| `8.1` | Table 8.1-A (people & ops trends) |
| `8.2` | AskQuestion Chrome Lot Healthy/Unhealthy, then Turbo Gear |
| `9.1` | Table 9.1-A (KPI update — one Quarterly Outcome page per turn) |
| `10.1` | Table 10.1-A (Bus trip — one option per turn until confirmed) |
| `11.1` | Table 11.1-A (PTO blocks + coverage) |
| `11.2` | Table 11.2-A (domains parked) + Dev Project–linked priority stack |
| `12.0` | Table 12.check, then commit checklist |

### Output contracts — Phases 3–11 {#output-contracts-phases-311}

**Table 3.1-A — Dev goals** *(step `3.1`)*

| Field | Review month | Planning month | Source |
|-------|--------------|----------------|--------|
| Projects Done / In Progress / Not Started | | | Dev Projects |
| Shipped vs stalled | | | Dev Projects + Weekly Log |
| Deep work trajectory | | | Weekly Meeting Log |
| Quarterly on-track? | | | Quarterly Outcomes |
| Planning month Dev Projects (`🌙 Month`) | | | Phase 11b selection |

**Table 3.2-A — Idea scrub** *(step `3.2`, one Type per turn)*

| # | Idea | Created | Difficulty | Recommendation | Decision |
|---|------|---------|------------|----------------|----------|

**Table 4.1-A — Activity scorecard** *(step `4.1`)*

| User | Wk 1 | Wk 2 | Wk 3 | Wk 4 | Total | Last Mo | Trend |
|------|------|------|------|------|-------|---------|-------|

**Table 4.1-B — Team efficiency** *(step `4.1`, second turn)*

| User | Hubstaff hrs | Activities | Activities/hr | MoM trend | Flag |
|------|--------------|------------|---------------|-----------|------|

**Table 5.1-A — Personal project audit** *(step `5.1`)*

| Project | Status | Days stalled | Recommendation | Decision |
|---------|--------|--------------|----------------|----------|

**Table 6.1-A — Personal finances** *(step `6.1`)*

| Item | Status | Action |
|------|--------|--------|

**Table 6.1-B — CL finances** *(step `6.1`)*

| Metric | Review month | Last month | Trend |
|--------|--------------|------------|-------|

**Table 7.1-A — CS health** *(step `7.1`)*

| Metric | Value | MoM | Flag |
|--------|-------|-----|------|

**Table 8.1-A — People & ops** *(step `8.1`)*

| Person | Hubstaff hrs | Trend | 1:1 overdue? | Action |
|--------|--------------|-------|--------------|--------|

**Table 9.1-A — KPI update** *(step `9.1`, one outcome page per turn)*

| KPI | Target | Actual MTD | Pace | Action |
|-----|--------|------------|------|--------|

**Table 10.1-A — Bus trip** *(step `10.1`)*

| Option | Dates | Destination | Weather fit | Decision |
|--------|-------|-------------|-------------|----------|

**Table 11.1-A — PTO** *(step `11.1`)*

| Day off | Coverage | Handoff tasks | Calendar blocked? |
|---------|----------|---------------|-------------------|

**Table 11.2-A — Planning context** *(step `11.2`)*

| Field | Value |
|-------|-------|
| Dev Projects linked to planning month (`🌙 Month`, max 5 total) | |
| Domains Parked | |

**FIELD CHECK — Phase 1** *(Table 1.check)*

| Item | Pass / N/A |
|------|------------|
| Wellness trajectory presented | |
| Life health trajectory presented | |
| All 6 identity ratings captured (`monthly_life_health`) | |
| Quarterly gate passed or escalated | |
| Phase 1d resolved or N/A | |

**Do not proceed to `2.1` until Table 1.check passes.** Run `gate --phase 2`.

**FIELD CHECK — Commit** *(Table 12.check)*

| Item | Pass |
|------|------|
| Monthly Plan Log entry created (`Monthly Plan for [Planning Month YYYY]`) | |
| Planning month Dev Projects linked (`🌙 Month` on each selected record) + Domains Parked on Monthly Log | |
| Life + work health selects populated | |
| Team Activity Details appended | |
| Planning month Months page log appended | |
| Session Complete = Complete | |

## Interaction Style {#interaction-style}

- **One question at a time.** Never present a wall of choices. Walk through decisions sequentially.
- **Confirm before executing.** Each phase presents proposed actions, gets approval, then executes before moving on.
- **Exclude Shopping List** (Todoist project `6W36wRPXj8qC2RCc`) from all analysis.
- **Data integrity:** All Knack/Pipedrive write operations require explicit user approval before execution.
- **Monthly lens:** Forward-looking planning for the current month, backed by a structured review of the prior month. Not individual task triage — the weekly plan handles that.

## Procedure {#procedure}

## Phase 0: Data Pull (silent, before conversation) {#phase-0-data-pull-silent-before-conversation}

Compute **review month** and **planning month** from today's date (see Month framing above).

Refresh Withings body-comp into Notion first:

```
node "scripts/withings-sync.mjs" --days 60 --write
```

That populates the Health Data DB for the target month plus the prior month (so trend math works). Skip silently if the script errors -- Notion will still have whatever was previously synced.

Then archive recent watch data into the Health Data DB. Health Sync's Drive export is rolling ~30 days, so this step is what makes 60-day MoM math possible:

```
health_persist_recent({ days: 60 })
```

This upserts Steps, Heart Rate Avg/Max, Sleep stages, and Workout aggregates per day. The MCP returns `counts: { created, updated, skipped }`. Older days that fell out of Drive's window will have already been persisted by previous monthly/weekly runs -- they stay in Notion as the long-term archive. If counts.errors > 0, note in Phase 2 footer.

Run **`health_source_status`** and note gaps before conversation:

- `drive.ok` false → re-auth Google MCP, update `.cursor/mcp.json` health-data env
- `metrics_seen` missing `sleep` → Samsung Health → Settings → Health Connect → enable Sleep write
- `metrics_seen` missing `rhr` or `hrv` → enable Resting heart rate + Heart rate variability in Health Sync Drive export (see `context/systems/health-data.md` Phone setup checklist). MCP parsers are ready; folders must exist on phone.
- Sparse sleep (< 20 nights in review month after persist) → watch off-wrist or Health Connect sleep permission

Then call the **health-data MCP** for a 60-day consolidated pull:

```
health_get_summary({ days: 60 })
```

Use the returned `daily` (one row per date) to slice **review month** vs. the month before it for body-comp + recovery trend tables. Use `stats` and `trend` for at-a-glance summaries. Daily rows are now backed by the Notion Health Data DB archive (set by the persister above), so days older than Drive's window still have data. If `sources.health_sync.ok` is false, the day-of metrics for the most recent few days may be stale, but historical days from Notion still flow -- note "Health Sync sync inactive (using archived watch data)" in Phase 2 footer rather than aborting.

**Months DB gate (planning + review month records):**

Before other pulls, resolve both Months pages. Store `planning_month_page_id` and `review_month_page_id` for Phase 12.

1. Query **Months DB** (`121f40c2-487b-80f2`) for **planning month**: search title `[FullMonthName] [YYYY]` first, then month name only.
2. If no planning month record exists, create via `personal_notion_create_database_entry`:
   - `database_id`: `121f40c2-487b-80f2-8cbf-ca5d3d5c33c2`
   - `properties`: `{ "Month": { "title": [{ "text": { "content": "[FullMonthName] [YYYY]" } }] } }`
   - Set page icon 🗓️ via `personal_notion_update_page` after create.
3. Resolve **review month** Months page the same way (for retrospective context; Monthly Plan Log `Month` relation points to **planning month**).
4. If planning month `Previous Month` is empty and review month page exists, set `Previous Month` relation on planning month → review month page.

Then pull the rest of the data via MCP tools in parallel:

1. **Weekly Meeting Log DB** (`322f40c2-487b-81bd`): (a) all entries from **review month** (PHQ-2, GAD-2, energy scores, structured KPI fields) for intra-month trajectory; (b) **last 6 entries by Meeting Date** (may span month boundaries) for life-health streak detection in Phase 1d.
2. **Monthly Plan Log DB** (`344f40c2-487b-806d`): prior entry titled `Monthly Plan for [Review Month YYYY]` for **month-over-month comparison** (when reviewing May, baseline is `Monthly Plan for May 2026`). Also check whether `Monthly Plan for [Planning Month YYYY]` already exists (skip duplicate create in Phase 12 if present).
3. **Quarterly Meeting Log DB** (`344f40c2-487b-80ed`): entry linked to **current quarter** (Quarter Tracker `Current` = "Current") — completeness gate for Phase 1c
4. **Notion Workouts DB** (`127f40c2-487b-80ba`): **review month** workout log
5. **Dev Projects DB** (`341f40c2-487b-80ac`): all projects assigned to current quarter, full status sweep
6. **Quarterly Outcomes** (`341f40c2-487b-80c2`): current quarter's outcome pages for Phase 1c review + Phase 9 KPI update
7. **Values DB** (`342f40c2-487b-80c5`): current health statuses for all 6 categories
8. **Pipedrive**: sales pipeline velocity for **review month** (deals created, won, lost), CS pipeline snapshot
9. **Knack Customers** (`object_2`): field_464, field_1035, field_1601, field_1438, field_1428, field_1491
10. **Knack Invoices** (`object_18`): revenue, AR aging, collections for **review month** (field_131 date, field_191 amount, field_132 status, field_383 balance)
11. **Hubstaff**: **review month** hours per team member, broken down by project (especially projects 3563292 and 3563452 for sales efficiency)
12. **Google Calendar**: **planning month** major events, available weekends for trip/time off
13. **Small Talk DB** (`121f40c2-487b-802d`): all entries from **review month** (social interaction frequency)
14. **Health Data + Watch metrics**: Loaded via the `health_get_summary({ days: 60 })` call above. Use throughout Phase 1 (recovery/wellness) and Phase 2 (body-comp + sleep + RHR + HRV trends).

## Phase 1: Wellness Trends (~3 min) {#phase-1-wellness-trends-3-min}

**Purpose:** Spot intra-month trajectories and month-over-month drift in mental health and energy.

Present wellness metrics using the **dual-level trend format**:

**Intra-month trajectory** -- plot the ~4 Weekly Meeting Log entries in chronological order:

```
WELLNESS -- INTRA-MONTH TRAJECTORY
| Metric            | Wk 1 | Wk 2 | Wk 3 | Wk 4 | Avg  | Last Mo | Trend |
|-------------------|------|------|------|------|------|---------|-------|
| PHQ-2 Score       |      |      |      |      |      |         |       |
| GAD-2 Score       |      |      |      |      |      |         |       |
| Energy Rating     |      |      |      |      |      |         |       |
| Strength Sessions |      |      |      |      |      |         |       |
| Small Talk Count  |      |      |      |      |      |         |       |
| Spirit Minutes    |      |      |      |      |      |         |       |
| Deep Work Minutes |      |      |      |      |      |         |       |
| Journal Count     |      |      |      |      |      |         |       |
```

"Last Mo" column comes from the previous Monthly Plan Log entry. "Trend" shows the % delta and direction.

**Flag any metric where:**
- 3+ consecutive weeks of decline within the month
- Month-over-month delta exceeds 20% in either direction
- A target exists and the metric is below 80% of target

**Life Health Trajectory** — plot the last up-to-6 Weekly Meeting Log entries (chronological) using the life-health select properties (`Spirituality Health` … `Parenting Health`). For weeks before those properties existed, annotate `(no weekly snapshot — used Values DB)` and use current Values DB Health as fallback; **do not false-trigger** streak gates on inferred data.

```
LIFE HEALTH -- WEEKLY TRAJECTORY (last N weeks)
| Category     | Wk-4 | Wk-3 | Wk-2 | Wk-1 | Streak        |
|--------------|------|------|------|------|---------------|
| Spirituality | H/U  |      |      |      | N wks Unhealthy |
| Fitness      |      |      |      |      |               |
| Work         |      |      |      |      |               |
| Social       |      |      |      |      |               |
| Admin        |      |      |      |      |               |
| Parenting    |      |      |      |      |               |
```

**Streak computation:** For each category, count consecutive weeks rated Unhealthy ending at the most recent week with data. A streak of **3+** triggers Phase 1d.

Then proceed with qualitative assessment:
1. Values category check: pull Health statuses from Values DB (`342f40c2-487b-80c5`). Which of the 6 categories got attention and which got starved?
2. Social connectedness trend: Small Talk count this month vs. last month from Monthly Plan Log.
3. Dating health check (if active): net positive or net negative to energy and structure this month?
4. If depression or anxiety trended upward (3+ weeks or month-over-month), flag for proactive intervention (therapy session, med review, workload reduction)

**Outputs:** Observation notes. Task creation only if intervention needed.

## Phase 1b: Identity Check (~5 min) {#phase-1b-identity-check-5-min}

**Purpose:** Am I living in alignment with my values, or just checking boxes?

Read `context/self/values.md` for the 6 value categories and their Pictures of Success. For each category, do a binary honest assessment — **Healthy** or **Unhealthy** (same scale as weekly plan and Values DB). One `AskQuestion` per category.

1. **Spirituality** — practicing state management regularly?
2. **Fitness** — training consistently and connecting body to mind?
3. **Work** — showed up, built the platform, built relationships?
4. **Social** — building friendships and going toward people?
5. **Admin** — duties under control or piling up?
6. **Parenting** — investing in Matthew?

Cross-reference with the weekly life-health trajectory from Phase 1 and Values DB. Discrepancies (e.g. mostly Healthy weeks but Unhealthy monthly rating) are discussion points.

For any area rated **Unhealthy**, generate one specific action for **planning month**. Don't over-plan — one action per flagged area.

Store ratings in session state (`monthly_life_health`) for Phase 12 commit.

**Outputs:** `monthly_life_health` map captured. Surface flagged areas in relevant later phases.

## Phase 1c: Quarterly Plan Checkpoint (~5 min) {#phase-1c-quarterly-plan-checkpoint-5-min}

**Purpose:** Confirm the current quarter has a committed strategic frame before setting **planning month** goals. Monthly goals must ladder up to quarterly themes — not float independently.

**Data sources:** Quarterly Meeting Log (`344f40c2-487b-80ed`), Quarter Tracker (`121f40c2-487b-802e`), Quarterly Outcomes pages (`341f40c2-487b-80c2`), Dev Projects with 🍁 Quarter = current quarter.

### Quarterly plan completeness gate {#quarterly-plan-completeness-gate}

1. Identify **current quarter** from Quarter Tracker (`Current` select = "Current").
2. Query Quarterly Meeting Log for an entry whose Quarter relation = current quarter.
3. Pull all three Quarterly Outcomes pages (CL, TG, Personal) for current quarter. Check: theme callout present? KPI table has targets (not all blank)?
4. If **no Quarterly Meeting Log entry exists for the current quarter** OR **any of the three Quarterly Outcomes pages lack a committed theme** (no theme callout / empty strategic frame):
   - **STOP the monthly plan.** Do not proceed to Phase 2 or beyond.
   - Tell Aaron: "Q[n] quarterly plan is not committed. Run the quarterly plan first."
   - Activate `context/skills/quarterly-plan/SKILL.md` end-to-end. Resume the monthly plan only after Phase 14 of the quarterly session completes (Quarterly Meeting Log entry exists for the current quarter).
5. **Exception (Aaron override only):** If Aaron explicitly says "proceed without quarterly plan," log the gap in Key Misses and cap planning-month commitments to maintenance (no new strategic goals). Default is **stop**.

### Quarterly progress review (run only when gate passes) {#quarterly-progress-review-run-only-when-gate-passes}

Whether or not the full quarterly session happened, review what's committed **only after the gate passes**:

1. **Themes:** One-line recap per domain (CL / TG / Personal) from Quarterly Outcomes pages.
2. **Project pace:** Dev Projects assigned to current quarter — Done vs In Progress vs Not Started vs at-risk (Due Date passed or In Progress 30+ days with no sub-item movement).
3. **KPI pace preview:** Which KPI rows have targets? Any already >50% behind expected pace for the month we're in? (Full actuals update happens in Phase 9.)
4. **Alignment check:** Flag if proposed **planning month** priorities (surfaced in Phases 3–4) don't serve the quarterly theme — force a reconcile before committing.

**Outputs:** Quarterly alignment notes carried into Phases 3–4 and Phase 9. If gate failed, monthly plan **stops here** — no Phase 1d or Phase 2+. Escalation to full quarterly plan is mandatory unless Aaron explicitly overrides.

## Phase 1d: Sustained Unhealthy Gate (~10 min, conditional) {#phase-1d-sustained-unhealthy-gate-10-min-conditional}

**Triggers when any life category has been Unhealthy for 3+ consecutive weeks** (from Phase 1 life-health trajectory, including weeks spanning month boundaries).

When triggered:

1. **Pause** Phases 2–11 until this gate completes. Do not set planning-month goals while a sustained-unhealthy area lacks a substantive response.
2. Present the streak table + one-sentence evidence per flagged category (KPI data from weekly logs).
3. One `AskQuestion` per flagged category: name **one substantive change** for planning month — approach shift, boundary, delegation, or cut (not a task tweak).
4. Draft `Health Intervention Notes` for Phase 12 (category → change committed).
5. Resume monthly plan only after all flagged categories have a named change.

If no 3-week streaks: skip silently (~0 min).

## Phase 2: Fitness Review & Reprioritization (~5 min) {#phase-2-fitness-review-and-reprioritization-5-min}

**Purpose:** Identify consistency and intensity trends, not just last week's performance.

1. Total workout days in **review month** vs. target (16-20 days at 4-5/week). Compare against previous Monthly Plan Log entry (Strength Sessions + Cardio Sessions) and flag if >20% change.
2. Workout type distribution (Pull/Push/Legs/Cardio balance)
3. Intensity trend: plot weekly Strength/Cardio Sessions from Weekly Meeting Log -- improving, declining, or flat week-over-week?
4. Motorcycle rides in **review month** (check capacity-rules.md for last ride date)
5. **Body composition + recovery trend** -- use the `health_get_summary({ days: 60 })` result loaded in Phase 0. Slice `daily` into **review month** and the month before it, compute means, and present:

   ```
   BODY COMPOSITION + RECOVERY -- MONTH-OVER-MONTH
   | Metric                  | Review Mo Avg | Prior Mo Avg | Δ        | Direction |
   |-------------------------|-------------|-------------|----------|-----------|
   | Weight (lbs)            |             |             |          |           |
   | Body Fat (%)            |             |             |          |           |
   | Lean Mass (lbs)         |             |             |          |           |
   | Sleep (hours)           |             |             |          |           |
   | Heart Rate Avg (bpm)    |             |             |          |           |
   | Resting HR (bpm)        |             |             |          |           |
   | HRV (ms RMSSD)          |             |             |          |           |
   | Steps (per day)         |             |             |          |           |
   | Workout Active Min/wk   |             |             |          |           |
   | Weigh-in days           |             |             |          | (count of distinct days with weight in review mo) |
   | Sleep nights logged     |             |             |          | (count of distinct days with sleep_hours in review mo) |
   ```

   "Direction" interprets the delta given Aaron's current intent (recomp -> want lean up + fat % down; recovery -> want sleep up, RHR down, HRV up). Flag any of:
   - Weigh-in days < 12 (consistency concern)
   - |ΔWeight| > 4 lbs without a stated cut/bulk goal
   - Lean mass dropping while weight is stable (likely undereating during training)
   - Sleep avg < 6.5h (chronic sleep debt)
   - HR Avg up > 5 bpm month-over-month while training load is unchanged (recovery deficit signal)
   - RHR up > 5 bpm month-over-month (overtraining or illness signal)
   - HRV down > 15% month-over-month (recovery deficit)
   - Sleep nights logged < 20 (watch off-wrist or sync issue)

   Render `--` for any row whose underlying data is null. Common nulls today: Resting HR + HRV (folders not yet enabled in Health Sync). When the persister has been running for less than a full prior month, "Last Mo Avg" may be partial -- annotate with `(partial: <N> days)` in that cell.
6. Identify systemic barriers (if a particular day is always skipped, why?)
7. Adjust targets for **planning month** if needed (e.g., drop to 3/week during high-travel periods)

**Outputs:** Adjusted fitness targets for planning month. Todoist recurring task updates if schedule changes. Body-comp averages stored on the Monthly Plan Log in Phase 12 (review month metrics).

## Phase 3: Dev Work Review & Goals (~10 min) {#phase-3-dev-work-review-and-goals-10-min}

**Purpose:** Project completion progress against quarterly targets, not individual task triage.

1. Pull all Dev Projects assigned to the current quarter. How many are Done vs. In Progress vs. Not Started?
2. What shipped in **review month** vs. what stalled? Which projects had "This Week" set but didn't move?
3. Dev hours spent vs. budgeted (from capacity math, Business Development DB for deep work minutes). Plot weekly Deep Work Minutes from Weekly Meeting Log to show intra-month trajectory.
4. **Quarterly progress check**: Are we on track to complete the quarter's assigned projects? If not, which ones are at risk and why?
5. Non-TG dev: did personal and CL projects get attention, or consumed by ops?
6. Compare Projects Completed, Projects In Progress against previous Monthly Plan Log entry -- flag if >20% change in either direction.
7. Confirm **planning month** Dev Project candidates align with Phase 1c quarterly themes — final `🌙 Month` links happen in **Phase 11b** (not free-text goals).

**Outputs:** Dev Projects status updates. Candidate list for Phase 11b month linking. Todoist tasks if needed.

## Phase 3b: Idea Roadmap Scrub (~8 min) {#phase-3b-idea-roadmap-scrub-8-min}

**Purpose:** Sweep the upstream **Status = Idea** backlog for each roadmap (Turbo Gear, Chrome Lot, Personal). Promote ideas worth queuing, archive stale ones, hold the rest. Prevents the Idea status from becoming a graveyard and feeds the next quarterly plan with a curated candidate list.

**Data source:** Dev Projects DB (`341f40c2-487b-80ac`). For each Type in turn — Turbo Gear, Chrome Lot, Personal — query with `Status = Idea`, sort by created time ascending (oldest first).

### Procedure (run once per Type, in this order: TG → CL → Personal) {#procedure-run-once-per-type-in-this-order-tg-cl-personal}

1. Query Dev Projects with `Type = <current>` AND `Status = Idea`. Pull the title, created time, Due Date, 🧭 Value relation, and any sub-item count.
2. **AI pre-assessment** for each idea (silent — present results in step 3):
   - **Overlap:** Does this duplicate or directly relate to an existing project (any Status) in Dev Projects, CL Internal Projects, or active Quarterly Outcomes?
   - **Difficulty:** XS / S / M / L / XL based on title + any sub-items
   - **Delegation potential:** Could a team member execute this? (CL → Lexie/Tristen/Ran; TG → Pyakz/markwillcraft; Personal → likely no)
   - **Values alignment:** Which of the 6 categories does this serve? Flag if it doesn't clearly serve any.
   - **Recommendation:** Promote (→ Not started) / Archive / Delegate / Hold
3. Present the list to Aaron with recommendations:

   ```
   IDEA SCRUB — [Type] ([N] ideas)
   1. "[Idea name]" (created [date])
      Difficulty: [X], Serves: [Value], Delegation: [Y/N to whom]
      Overlaps with: [existing project or "none"]
      Recommendation: [Promote / Archive / Delegate / Hold]
   2. ...
   ```
4. Decide one idea at a time (use AskQuestion). For each:
   - **Promote:** Update Status `Idea → Not started`. Optionally set 🍁 Quarter relation if it should be considered for next quarter (see Phase 9). Optionally set Due Date.
   - **Delegate:** Route to the appropriate system (Work Notion CL Tasks for CL ops, Todoist for personal handoffs, Pipedrive activity for customer-facing). Then archive the Dev Projects idea entry with explicit approval.
   - **Archive:** Set Status `Idea → Done` (or trash the page) — explicit approval required.
   - **Hold:** Leave as-is for next month's scrub.
5. **Caps per Type per month:**
   - Promote at most **3 ideas to Not started**. If more look promising, defer to next month or surface them in the next quarterly plan.
   - No cap on archives — clearing dead ideas is encouraged.

### Sanity guardrails {#sanity-guardrails}

- If a Type has **>15 Idea-status entries**, flag it as inbox bloat and recommend a deeper triage outside the monthly cadence.
- If the same idea has been **Held for 3+ consecutive monthly scrubs**, force a Promote-or-Archive decision this round (no more holds).
- Do not promote an idea that overlaps with an active project — instead, append it as a sub-item to that project (or archive as duplicate).

**Outputs:** Status updates in Dev Projects (Idea → Not started / Done). Delegated tasks in target systems. Notes on held ideas (with hold count) for next-month review and for the quarterly plan candidate pool.

## Phase 4: Sales Progress & Goals (~10 min) {#phase-4-sales-progress-and-goals-10-min}

**Purpose:** Pipeline health, team performance, and efficiency -- not individual deal triage.

### Pipeline Health {#pipeline-health}
1. Pipedrive sales pipeline velocity: new deals, won deals, lost deals, conversion rate. Compare against previous Monthly Plan Log entry (CL Revenue, CL Customer Count, CL Churn) -- flag if >20% change.
2. Revenue this month vs. last month (Pipedrive deal values)
3. Stale deal audit: deals older than 60 days with no movement

### Activity Scorecard {#activity-scorecard}

Aggregate the per-user activity totals from the ~4 Weekly Meeting Log entries this month. Present as:

```
COMPLETED ACTIVITIES -- INTRA-MONTH TRAJECTORY
| User    | Wk 1 | Wk 2 | Wk 3 | Wk 4 | Total | Last Mo | Trend |
|---------|------|------|------|------|-------|---------|-------|
| Aaron   |      |      |      |      |       |         |       |
| Lexie   |      |      |      |      |       |         |       |
| Tristen |      |      |      |      |       |         |       |
| Ran     |      |      |      |      |       |         |       |
| Total   |      |      |      |      |       |         |       |
```

"Last Mo" comes from the previous Monthly Plan Log entry (Total Activities per user). Flag any user with declining week-over-week activity or >20% month-over-month change.

### Team Efficiency Analysis {#team-efficiency-analysis}
Cross-reference Hubstaff time data with activity totals per team member (Aaron, Tristen, Lexie, Ran):
- **Time invested**: Hubstaff hours on "In Person Sales & Customer Service" (project 3563292) + "Sales Related Office & Admin" (3563452) for the month
- **Results produced**: Pipedrive activities completed (from activity scorecard above), deals progressed/won, new deals created
- **Efficiency ratio**: activities per hour invested -- who is getting the most out of their time?
- Flag imbalances: someone spending lots of hours with few completed activities, or vice versa
- Compare month-over-month to spot improving or declining trends

### Goal Setting {#goal-setting}
- Who hit their activity targets in **review month**? Who fell short?
- Set **planning month** targets per person: sales stops, new deal targets, activity minimums
- Adjust territory or deal assignments if efficiency data warrants it

**Outputs:** Monthly sales goals per team member. Pipedrive pipeline cleanup (archive dead deals with approval). Todoist tasks for target tracking. Delegation/territory adjustments if needed.

### Pipedrive Reference {#pipedrive-reference}

| ID | Meaning |
|----|---------|
| Pipeline 1 | Sales Pipeline |
| User 18865844 | Aaron |
| User 20938631 | Tristen |
| User 22704318 | Lexie |
| User 19274648 | Ran |

### Hubstaff Project Reference {#hubstaff-project-reference}

| Project ID | Name |
|------------|------|
| 3563292 | In Person Sales & Customer Service |
| 3563452 | Sales Related Office & Admin |

## Phase 5: Personal Project Tracker Audit (~10 min) {#phase-5-personal-project-tracker-audit-10-min}

**Purpose:** Deep cleanup that weekly micro-scrubs can't do. Scoped to personal projects only — TG and CL projects are covered in Phase 3 and the weekly plan. Projects **kept active** for planning month should receive `🌙 Month` in Phase 11b (or here if already certain).

1. Pull Dev Projects with Type=Personal (`341f40c2-487b-80ac`, filter Type=Personal)
2. Flag stalled projects: "In progress" for 30+ days with no sub-step completion
3. Backlog review: present top backlog items (Status=Not started), bring 1-2 up to active
4. Archive truly dead projects (with approval)
5. Reorganize priority order if needed
6. Ensure every active project has actionable sub-steps
7. Life admin catch-up: any personal projects that keep getting deferred (house, finances, health, etc.)?

**Outputs:** Dev Projects status updates (archive, activate, reprioritize). Todoist tasks for newly activated work.

## Phase 6: Financial Review (~10 min) {#phase-6-financial-review-10-min}

**Purpose:** Monthly-only phase covering both personal finances and Chrome Lot business financials.

Data sources: Knack Invoices (`object_18`) for CL revenue/AR. QuickBooks for P&L and expenses (when MCP is configured -- see `guidelines/monthly-plan.md` for setup options). Until QuickBooks MCP is live, use manual input for P&L data.

### Personal Finances {#personal-finances}
1. Budget check: are you spending within targets? Any categories out of control?
2. Savings goal tracking: on pace for the month/quarter/year?
3. Big purchase planning: house timeline update, any upcoming large expenses?
4. Action items: adjust budget, move money, schedule financial tasks

### Chrome Lot Business Finances {#chrome-lot-business-finances}
1. Revenue this month vs. last month (Knack invoices -- field_191 amount, field_131 date)
2. Accounts receivable health: total outstanding balance, aging distribution
3. Collections progress: compare against last month's late invoice count (field_1428, field_1491)
4. Profitability check (QuickBooks when available): revenue minus expenses, margin trend
5. Any customers consistently late? Cross-reference with CS health from Phase 7.

**Outputs:** Todoist tasks for financial actions. Budget/savings adjustments noted. Observation notes on CL profitability trend.

## Phase 7: CS Health Monthly Review (~5 min) {#phase-7-cs-health-monthly-review-5-min}

**Purpose:** Churn trends, invoice aging direction, and account manager workload balance.

**Data sources:**
- Active customers + health: Knack `object_2`, filter `field_464 = Yes`, read `field_1601` (Customer Health Status)
- Invoice aging: Knack `object_18` (Invoices), aggregate by `field_132` (status) and date windows
- Account manager workload: Pipedrive activity counts by `owner_id` from Phase 4 (not Knack)
- Month-over-month comparison: previous Monthly Plan Log entry

1. Net customer change this month (new vs. churned). Compare CL Customer Count and CL Churn against previous Monthly Plan Log entry -- flag if >20% change.
2. Invoice aging trend: are unpaid/overdue invoice counts getting better or worse vs. last month? (First run: skip -- no comparison available.)
3. Customer health distribution shift: more Happy, fewer At Risk, or the reverse? Flag any customer with Unknown/missing health status -- create Todoist task to classify.
4. Account manager workload balance: pull Pipedrive activity totals by owner from Phase 4 data. Is one person overloaded (>250 activities/month)?
5. Any systemic CS issues to address (e.g., recurring complaints, geographic gaps)

**Outputs:** Observation notes. Adjust delegation assignments if workload is unbalanced. Todoist tasks for data hygiene (unclassified customers).

## Phase 8: People & Operations Trends (~5 min) {#phase-8-people-and-operations-trends-5-min}

**Purpose:** Monthly trends in team performance, not individual weekly flags.

**Data sources:**
- Hubstaff hours: `hubstaff_get_daily_activities` for the full month, aggregate by `user_id` against `context/people/index.md` mapping
- Team roster: `context/people/index.md` + `hubstaff_get_members` (cross-check: flag anyone in Hubstaff not in directory, or vice versa)
- 1:1 cadence: `Last Meeting` dates in `context/people/index.md`
- Photographer-specific metrics (call-in rates, issue rates, time off): **TBD -- no automated data source yet.** Pull from Work Notion CL Tasks or Process Street manually, or skip until a source exists.
- New photographers hired: count completed runs of the Process Street "New Hire Onboarding" workflow (template ID `g7V9_B1K7z7rqgLS8Q1GEg`) within the month. Call `processstreet_list_workflow_runs` with `template_id=g7V9_B1K7z7rqgLS8Q1GEg` and `status=Completed`, then filter on `audit.createdDate`.

1. Roster reconciliation: run `hubstaff_get_members` and compare against `context/people/index.md`. Update directory for any new hires or removed members before proceeding.
2. Hubstaff hours per team member: consistent, declining, or erratic? Compare against previous Monthly Plan Log entry -- flag if >20% change. (First run: skip comparison.)
3. Photographer performance (if data available): month-over-month call-in rates, issue rates, time off trends.
4. Any photographer/team member on a downward trajectory over 2+ months?
5. 1:1 consistency: check `Last Meeting` in `context/people/index.md` for every active team member. Flag anyone >45 days past their check-in cadence. Create Todoist tasks for the overdue 1:1s.
6. Hiring/firing decisions: anyone need to go? Need to hire?

**Outputs:** Todoist tasks for 1:1s and performance conversations. Hiring decisions noted. Updated `context/people/index.md` for any roster changes.

## Phase 8b: Work Domain Health Rating (~5 min) {#phase-8b-work-domain-health-rating-5-min}

**Purpose:** Rate Chrome Lot and Turbo Gear as Healthy or Unhealthy at the domain level. Results stored on Monthly Plan Log in Phase 12.

1. **Chrome Lot brief:** Summarize review-month signals from Phases 4 (sales), 6 (financial), and 7 (CS health) — revenue trend, churn, invoice aging, pipeline velocity.
2. **Turbo Gear brief:** Summarize from Phases 3 and 5 — dev velocity, features shipped, demos, deep work minutes.
3. Optional reference: surface CL Departments (`341f40c2-487b-80cc`) and TG Departments (`341f40c2-487b-80c9`) Health statuses as context — domain rating is the two business-level selects, not per-sub-department re-rating (quarterly deep read unchanged).
4. One `AskQuestion`: **Chrome Lot — Healthy or Unhealthy?**
5. One `AskQuestion`: **Turbo Gear — Healthy or Unhealthy?**

Store in session state (`monthly_cl_health`, `monthly_tg_health`) for Phase 12 commit.

**Outputs:** Domain health ratings captured for Phase 12.

## Phase 9: Quarterly KPI Update (~5 min) {#phase-9-quarterly-kpi-update-5-min}

**Purpose:** Monthly touchpoint for quarterly tracking. Update KPI actuals on each Quarterly Outcomes page.

**Data sources per KPI:**
- CL Photo/Social Revenue: Knack `object_18` invoices, aggregate by month
- CL Sales Calls: Pipedrive completed activity counts from Phase 4
- CL New Customers: Knack `object_2` created-date filter
- CL New Photographers: Process Street "New Hire Onboarding" workflow runs in the month. Template ID `g7V9_B1K7z7rqgLS8Q1GEg`. Use `processstreet_list_workflow_runs` with `template_id=g7V9_B1K7z7rqgLS8Q1GEg`, `status=Completed`, filter by `audit.createdDate`. Fall back to Work Notion CL Team DB creation date if Process Street is missing.
- TG MRR: **TBD -- no MCP source yet** (MongoDB or Stripe integration needed)
- TG Features Shipped / PRs Submitted: owner `torchcommercialmedia`. Active TG repos: `turbo-gear` (frontend), `turbo-gear-api` (backend), `turbo-gear-workspace`, `turbo-gear-feeder`, `turbo-gear-uploader`. For each, call `github_list_pull_requests` with `state=closed, per_page=100`, then filter client-side where `merged_at` is in the target month. Report totals by repo and by author. Treat any PR with non-null `merged_at` as "shipped" (vs. closed-without-merge).
- Personal KPIs: data source varies per row; skip if no source

1. Pull the current quarter from Quarter Tracker (`121f40c2-487b-802e-b7a8-c0ad8d74f5e7`, `Current` select = "Current")
2. Pull all Quarterly Outcomes pages (`341f40c2-487b-80c2-a871-eb600fd78e4d`) linked to the current quarter (all three: CL, TG, Personal)
3. For each outcome page, fetch the KPI table content.
4. **Empty-table escape hatch:** If the KPI cells are all blank (no targets set), do NOT attempt to populate actuals. Instead:
   - Create a Todoist task: "Set Q[n] KPI targets on [outcome page title]" due before the next monthly review
   - Skip to the next outcome page
5. If targets exist, for each KPI row:
   - Pull the month-to-date actual from the data source listed above
   - Update the correct Month 1/2/3 cell (requires manual Notion edit -- MCP can't write table cells)
   - Flag any KPI significantly behind pace (<50% of target by the end of month 2, for example)
6. If any area is off-track, discuss with Aaron: adjust target, accelerate effort, or accept the miss?

**Outputs:** Updated KPI actuals on Quarterly Outcomes pages (manual Notion edits). Todoist tasks for setting missing targets. Flagged areas for intervention.

## Phase 10: Trip Planning with Bus (~5 min) {#phase-10-trip-planning-with-bus-5-min}

**Purpose:** Ensure at least one quality outing with Matthew in **planning month**.

1. Check Google Calendar for available weekends/long weekends in **planning month**
2. Check weather forecast (seasonal -- "is it camping weather?")
3. If camping weather: suggest 1-3 day camping options within driving distance
4. If not: suggest alternatives (indoor attractions, day trips, museum/aquarium visits)
5. One-by-one: confirm dates, destination, and logistics
6. Create calendar events and packing/prep Todoist tasks

**Outputs:** Calendar events for trip. Todoist prep tasks.

## Phase 11: Personal Time Off (~3 min) {#phase-11-personal-time-off-3-min}

**Purpose:** Protect Aaron's recovery time — 2 days off in **planning month** minimum.

1. Review **planning month** calendar for 2 suitable days off (avoid heavy CS/sales weeks)
2. Identify coverage needs: who handles what when Aaron is out
3. Notify relevant people (Tristen, Lexie, Ran) via Teams or Todoist delegation
4. Block calendar days
5. Create handoff tasks in Todoist assigned to coverage people

**Outputs:** Calendar blocks. Todoist handoff tasks. Teams notification if needed.

## Phase 11b: Planning Month Dev Projects (~5 min) {#phase-11b-planning-month-dev-projects-5-min}

**Purpose:** Link **specific Dev Projects** to the **planning month** via `🌙 Month` (mirrors quarterly `🍁 Quarter`). Weekly plan reads these relations — not Priority Stack text.

**Required every monthly plan.** Store in session state (`monthly_dev_project_ids`, `monthly_domains_parked`) for Phase 12.

1. Pull open Dev Projects (current quarter, Status ≠ Done/Idea) grouped by Type. Present lettered multi-select — **max 5 total** across all domains (Personal + CL + TG combined).
2. On confirm (Notion approval): set `🌙 Month` → **planning month** on each selected project **and all open sub-items** under it (`scripts/sync-dev-projects-month.mjs --include-descendants`). Clear `🌙 Month` from projects removed from this month's plate (do not clear `🍁 Quarter`). Quarter-only work stays quarter-linked, not month-linked.
3. Create missing Dev Project records before linking.
4. **Table 11.2-B — Domains parked** *(multi-select — reply with letters)*

| | Option |
|---|--------|
| **A** | Turbo Gear |
| **B** | Chrome Lot New Business |
| **C** | External TG Demos |
| **D** | Personal Projects |
| **E** | Dating Outreach |
| **F** | Custody (hold) |
| **G** | Other |
| **H** | None — all domains active |

5. Confirm selections + parked with Aaron before Phase 12. **CL sprint format retired** — do not write `Active CL Sprint`. **Do not** write free-text Priority Stack as source of truth.

**Outputs:** `monthly_dev_project_ids`, `monthly_domains_parked` ready for Phase 12.

## Phase 12: Commit (~3 min) {#phase-12-commit-3-min}

**Purpose:** Final review, execute remaining actions, log everything.

1. **Summary table:** All monthly goals and decisions across phases
2. **Final review:** Anything unrealistic? Adjust before committing.
3. **Execute remaining:** Create any Todoist/Calendar/Pipedrive/Notion items not yet committed during earlier phases.
4. **Log to Monthly Plan Log** (`344f40c2-487b-806d`): Create (or update) entry for this session with:
   - Meeting Date = today (or session date), Month relation = **planning month**
   - Name = `Monthly Plan for [Planning Month YYYY]`
   - Wellness scores (PHQ-2, GAD-2, Energy -- from **review month** Weekly Meeting Log averages)
   - Habit KPIs for **review month** (Strength/Cardio Sessions, Small Talk Count, Spirit/Deep Work/Ops/Field Work Minutes, Journal Count -- summed from Weekly Meeting Log entries in review month)
   - Activity KPIs (Aaron Activities, Lexie Activities, Tristen Activities, Ran Activities, Total Activities -- summed from **review month** Weekly Meeting Log entries)
   - **Body composition** (from Phase 2): Weight Avg, Body Fat Avg, Lean Mass Avg (**review month** means from `health_get_summary` daily slice), plus Weight Delta, Body Fat Delta, Lean Mass Delta (review-month avg minus prior-month avg, signed). Omit any field where the underlying data is null.
   - **Watch metrics** (from Phase 2): Sleep Avg, Heart Rate Avg, Resting HR Avg, HRV Avg, Steps Avg (**review month** means from `health_get_summary` daily slice) plus Sleep Delta, Heart Rate Delta, Resting HR Delta, HRV Delta, Steps Delta (review-month avg minus prior-month avg, signed). Also Workout Active Minutes (**review month** total in minutes). Omit any field where the underlying data is null.
   - Project counts (Completed, In Progress, broken out by Personal/CL/TG for **review month**)
   - Business metrics for **review month** (CL Revenue, CL Customer Count, CL Churn, TG Demos Given, TG Features Shipped)
   - **Life health** (from Phase 1b): all 6 select properties (`Spirituality Health` … `Parenting Health`)
   - **Work domain health** (from Phase 8b): `Chrome Lot Health`, `Turbo Gear Health`
   - **Health Intervention Notes** (from Phase 1d, if gate fired)
   - **Planning context (REQUIRED):**
     - `Domains Parked` (multi_select) — from Phase 11b (`monthly_domains_parked`)
     - `Priority Stack` (rich_text) — **optional snapshot** auto-generated from linked Dev Project titles (not authoritative)
     - **`🌙 Month` on Dev Projects** — authoritative; written in Phase 11b via Notion API
   - `Starved Values` — derived from life-health selects (categories rated Unhealthy)
   - Key Wins, Key Misses, Action Items (Action Items commit **planning month** priorities)
   Compare against last month's entry to show trend direction.
5. **Append per-user Pipedrive detail sections** to the Monthly Plan Log page using `personal_notion_append_blocks`. Pull completed activities from **review month** (use `pipedrive_get_activities` with `done: "1"` and filter by `marked_as_done_time` within review month bounds). Also pull all open activities per user. Append the following structure:

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

   Each activity line: `[type] subject -- deal name -- date`. For completed activities, show `marked_as_done_time` date. For open activities, show `due_date`. Prefix overdue open activities with `[OVERDUE]`. If a user has 0 activities in a section, show "None" instead of an empty list. Use `---` dividers between users. Pull per-user using user IDs (Aaron: 18865844, Lexie: 22704318, Tristen: 20938631, Ran: 19274648).

6. **Write monthly plan log on planning month Months page** (`planning_month_page_id` from Phase 0):

   Append via `personal_notion_append_blocks` — **do not** `personal_notion_clear_page` (template toggles live on every Months page). Use proper markdown per notion-formatting rules.

   Required sections:

   ```
   # Monthly Plan Log — [Session Date YYYY-MM-DD]

   > Review **[Review Month YYYY]** · Plan **[Planning Month YYYY]**

   ## Review snapshot
   (2–4 sentences: wellness/focus headline, one win, one miss)

   ## [Review Month] KPI rollup
   Link to Monthly Plan Log entry: [Monthly Plan for Planning Month YYYY](notion URL from step 4)

   ## Planning commitments
   (Numbered list — same items as Action Items on the Monthly Plan Log)

   ## Planning month Dev Projects
   (Linked via `🌙 Month` — list titles + Notion URLs from Phase 11b)

   ## Domains parked
   (Same as `Domains Parked` multi-select — e.g., Turbo Gear)

   ## Quarterly alignment
   (One line per domain tying planning month to current quarter theme)

   ## Session notes
   (Bus trip / PTO decisions, deferred items, structural carryovers)
   ```

   If a prior `Monthly Plan Log — [date]` section exists on the page from an incomplete rerun, append a new dated section rather than clearing the page.

7. **Update context files** if anything changed (capacity rules, people directory, dev goals, etc.)

## Cross-Cutting Rules {#cross-cutting-rules}

- **Forward-looking monthly plan.** Plan the current month; review the prior month. Never treat the session as "last month's plan" unless today is still in that month and the prior month's log wasn't written.
- **Quarterly plan gate (hard stop).** Phase 1c must confirm a Quarterly Meeting Log entry for the current quarter and committed themes on all three Quarterly Outcomes pages. If not, **stop** and run `context/skills/quarterly-plan/SKILL.md` first. Resume monthly plan only after quarterly Phase 14 completes.
- **Quarterly alignment.** When the gate passes, monthly goals must trace to quarterly themes.
- **Monthly lens, not weekly.** Trends and goal-setting, not daily task triage.
- **One question at a time.** Never present a wall of choices.
- **Confirm before writing.** All Knack/Pipedrive/Notion updates require explicit approval.
- **Exclude Shopping List** from all Todoist analysis.
- **Capacity enforcement.** If monthly goals imply an unsustainable weekly load, flag and adjust before committing.

## Outputs {#outputs}

- **Phase 0:** Silent data refresh (Withings, persister, health MCP) and parallel MCP/database pulls.
- **Phase 1:** Wellness + life-health trajectory tables; observation notes; intervention tasks only if needed.
- **Phase 1b:** `monthly_life_health` ratings captured; flagged areas for later phases.
- **Phase 1c:** Quarterly alignment notes; escalation to full quarterly plan if gate failed.
- **Phase 1d:** Health Intervention Notes drafted when 3+ week unhealthy streaks fire; pauses Phases 2–11 until resolved.
- **Phase 2:** Fitness targets; optional Todoist recurring updates; body-comp for Phase 12 Monthly Log.
- **Phase 3:** Dev Projects updates; dev goals; Todoist if needed.
- **Phase 3b:** Idea Roadmap Scrub across TG / CL / Personal — Idea→Not started promotions (cap 3/Type), delegations, archives; held items annotated with hold count for next month.
- **Phase 4:** Sales goals; pipeline cleanup Todoist; delegation adjustments.
- **Phase 5:** Personal Dev Projects audit outcomes; Todoist for new work.
- **Phase 6:** Financial Todoist and observation notes.
- **Phase 7:** CS observations; delegation; data-hygiene Todoist.
- **Phase 8:** 1:1/hiring Todoist; updated `context/people/index.md`.
- **Phase 8b:** `monthly_cl_health` and `monthly_tg_health` captured for Phase 12.
- **Phase 9:** Manual KPI updates on Quarterly Outcomes; Todoist for empty targets; flags.
- **Phase 10–11:** Calendar events; Todoist prep and handoff tasks; Teams optional.
- **Phase 11b:** `🌙 Month` links on selected Dev Projects + Domains Parked captured (session state).
- **Phase 12:** New Monthly Plan Log entry (review month) with full KPI rollup + planning context fields + life/work health selects + Health Intervention Notes; Team Activity Details append; **planning month Months page plan log**; context file updates.

## Failure modes & graceful degradation {#failure-modes-and-graceful-degradation}

- **Withings sync errors:** Skip silently; Phase 0 notes; use prior Notion rows; Phase 2 table uses `--` where null.
- **`sources.health_sync.ok` false:** Note "Health Sync sync inactive (using archived watch data)" in Phase 2 footer; continue with archived Notion watch data.
- **`health_persist_recent` errors (`counts.errors > 0`):** Note in Phase 2 footer.
- **First-run month comparisons:** Invoice aging and some Hubstaff MoM comparisons skip when no prior baseline (per procedure).
- **QuickBooks / TG MRR / photographer automation:** Fall back to manual input or skip per Phase 6, Phase 9, Phase 8 notes.
- **KPI table empty in Notion:** Empty-table escape hatch — Todoist task, skip actuals for that page.

## See also {#see-also}

- `../../router.md`
- `../weekly-planning/SKILL.md`
- `../quarterly-plan/SKILL.md`
- `../../systems/cadences.md`, `../../systems/capacity-rules.md`, `../../systems/notion-databases.md`
- `../../systems/pipedrive.md`, `../../systems/knack-fields.md`, `../../systems/hubstaff.md`, `../../systems/health-data.md`
- `../../self/values.md`, `../../self/current-priorities.md`
- `../../people/index.md`
- `../../work/chrome-lot/overview.md`, `../../work/chrome-lot/customer-service.md`
- `../../work/turbo-gear/overview.md`
- `guidelines/monthly-plan.md` (QuickBooks / setup; referenced in Phase 6)