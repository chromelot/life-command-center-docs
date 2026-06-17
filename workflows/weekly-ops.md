> **Source:** [`context/skills/weekly-ops/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-ops/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Weekly Ops — Chrome Lot Operations Review

## Table of contents

- [Trigger](#trigger)
- [Domain map — Dev Projects alignment](#domain-map-dev-projects-alignment)
- [Inputs](#inputs)
- [Execution Protocol (mandatory — read `context/workflow-execution.md` + `context/systems/workflow-output-contracts.md`)](#execution-protocol-mandatory-read-contextworkflow-executionmd-contextsystemsworkflow-output-contractsmd)
  - [Ledger step order (do not reorder)](#ledger-step-order-do-not-reorder)
  - [Table scope by step](#table-scope-by-step)
- [Interaction Style](#interaction-style)
- [Required Notion fields — index](#required-notion-fields-index)
- [Relationship to Weekly Plan](#relationship-to-weekly-plan)
- [Procedure](#procedure)
- [Pre-Phase 0: Planning Week Gate (`pre-0`)](#pre-phase-0-planning-week-gate-pre-0)
- [Phase 0: Data Pull (silent, before conversation)](#phase-0-data-pull-silent-before-conversation)
  - [Ops pulls included in briefing](#ops-pulls-included-in-briefing)
- [Phase 0b: Ops Briefing Gate (~3 min)](#phase-0b-ops-briefing-gate-3-min)
- [Phase 1: Work Summary (~8 min)](#phase-1-work-summary-8-min)
  - [Part A — Work volume (`1.1`, table 1-A first)](#part-a-work-volume-11-table-1-a-first)
  - [Part B — Busier customers (`1.1`, table 1-B second)](#part-b-busier-customers-11-table-1-b-second)
  - [Part C — Slower customers (`1.1`, table 1-C third)](#part-c-slower-customers-11-table-1-c-third)
  - [Part D — Pipedrive Activity Scorecard (`1.2`)](#part-d-pipedrive-activity-scorecard-12)
  - [Part C — Work Management (`1.3`)](#part-c-work-management-13)
- [Phase 2: Hiring, CS & Photographer Management (~25 min)](#phase-2-hiring-cs-and-photographer-management-25-min)
  - [Part A — Hiring (`2.1`)](#part-a-hiring-21)
  - [Part B — Check-in Cadence (`2.2`)](#part-b-check-in-cadence-22)
  - [Part C — High Job Days Review (`2.3`)](#part-c-high-job-days-review-23)
  - [Part D — Photographer Management (`2.4`)](#part-d-photographer-management-24)
- [Phase 3: Sales Management (~12 min)](#phase-3-sales-management-12-min)
  - [Part A — Aaron's Sales Activity (`3.1`)](#part-a-aarons-sales-activity-31)
  - [Part B — Team Sales Oversight (`3.2`)](#part-b-team-sales-oversight-32)
- [Phase 4: Finance & Admin (~8 min)](#phase-4-finance-and-admin-8-min)
  - [Part A — Late Invoice Review (`4.1`)](#part-a-late-invoice-review-41)
- [Phase 6: Post Production (~5 min)](#phase-6-post-production-5-min)
- [Phase 6-B: Social Media (~10 min)](#phase-6-b-social-media-10-min)
- [Phase 7: 1:1 Meetings (~5 min)](#phase-7-11-meetings-5-min)
- [Check: Ops FIELD CHECK (`check`)](#check-ops-field-check-check)
- [Commit (~5 min)](#commit-5-min)
  - [Commit procedure](#commit-procedure)
- [Cross-Cutting Rules](#cross-cutting-rules)
- [Outputs](#outputs)
- [Failure modes & graceful degradation](#failure-modes-and-graceful-degradation)
- [See also](#see-also)

---


<a id="trigger"></a>
## Trigger

This skill activates when Aaron says **"weekly ops"**, **"ops review"**, **"CL ops meeting"**, or **"operations meeting"**.

**Target duration:** ~50 min.

**Separate session** from Weekly Plan — typically a different day the same planning week. Weekly Plan covers life review + development planning; Weekly Ops covers CL operating execution (Pipedrive accountability, CS, sales, people).

<a id="domain-map-dev-projects-alignment"></a>
## Domain map — Dev Projects alignment

This session audits **`Create, and Audit Weekly Ops Plan`** and its sub-items in **Dev Projects** (`341f40c2-487b-80ac`). Each sub-item maps to a dedicated workflow phase:

| Dev Projects sub-item | Workflow phase | Ledger step(s) |
|---|---|---|
| *(cross-cutting)* Work summary + ops foundation | Phase 1 — Work Summary | `1.1`, `1.2`, `1.3` |
| Hiring | Phase 2 — Hiring | `2.1` |
| Customer Service | Phase 2 — CS Management | `2.2`, `2.3` |
| Photographer Management | Phase 2 — Photographer Management | `2.4` |
| Sales Management | Phase 3 — Sales Management | `3.1`, `3.2` |
| Finance & Admin | Phase 4 — Finance & Admin | `4.1` |
| Post Production | Phase 6 — Post Production | `6.1` |
| Social Media | Phase 6 — Social Media | `6.2` |
| 1:1 meetings | Phase 7 — 1:1 Meetings | `7.1` |

Department **Health** / **Priority** is confirmed or updated **in the phase for that department** — not a separate cross-cutting phase. See [`work-management.md`](../../work/chrome-lot/work-management.md) department→phase map.

Sub-items under **1:1 meetings** (e.g. remote transitions) are handled in Phase 7. **LESA Reshoot fixes** and other root-level Dev Projects are out of scope for this workflow unless surfaced in RED FLAGS. Dev Projects sub-item may still read **Workload & Hiring** — workflow phase title is **Hiring**.

<a id="inputs"></a>
## Inputs

Load via the router. Read these before starting:

- `context/systems/notion-databases.md` — DB IDs (Weekly Ops Meeting Log `379f40c2-487b-8130`, Weekly Meeting Log, Monthly Plan Log)
- `context/systems/pipedrive.md` — pipeline IDs, stages, user IDs, completion-time API quirks
- `context/systems/knack-fields.md` — Customer + Photographer field references
- `context/systems/hubstaff.md` — member IDs, weekly-report tool
- `context/systems/capacity-rules.md` — limits, overcommitment triggers, max PD activities/day
- `context/work/chrome-lot/customer-service.md` — Phase 2 CS logic
- `context/work/chrome-lot/sales.md` — Phase 3 sales logic
- `context/work/chrome-lot/operations.md` — Phase 2.3 photographer management logic
- `context/work/chrome-lot/social-media.md` — Phase 6.2 social pipeline + posting
- `context/work/chrome-lot/work-management.md` — Phase 1.3 daily records, backups, ops priority; in-phase department health
- `context/people/index.md` — delegation matrix, 1:1 tracking

<a id="execution-protocol-mandatory-read-contextworkflow-executionmd-contextsystemsworkflow-output-contractsmd"></a>
## Execution Protocol (mandatory — read `context/workflow-execution.md` + `context/systems/workflow-output-contracts.md`)

> **This skill's tables are the spec.** Same determinism standard as weekly plan — fixed headers, named sources, finish section before advancing.

1. **Date context (every turn, before any weekday or "this week" language):**
   ```
   node scripts/planning-dates.mjs [--today=YYYY-MM-DD] [--week-of=<ledger week_of>]
   ```
   Aaron may run weekly ops on **any day** — never assume session day = Monday. Use script output for review week, planning week, and weekday→ISO mapping. `Today's date` from `user_info` must match `--today` (CT). Planning week Monday must match the same week as Weekly Plan when both run in the same cycle.
2. **Init ledger** at session start: `node scripts/workflow-progress.mjs init --workflow weekly-ops` — omit `--week-of` to auto-set canonical planning Monday from today; if set manually, must be a Monday and should match `planning-dates.mjs`. Ops log title = `Week of <planning Monday>`.
3. **Notion log** — at `1.1`: `node scripts/workflow-notion-log.mjs create --ledger <path>`. After every `advance`: `workflow-notion-log sync`. Write phase fields per `context/systems/workflow-logs.md`. On `commit`: `workflow-notion-log complete`.
4. **Every turn:** `node scripts/workflow-progress.mjs status --workflow weekly-ops` — present only `current_step` (status includes date warnings if ledger `week_of` is wrong)
5. **Phase banner** on every user-facing message: `**[Weekly Ops · Phase X.Y — title]**` (ledger uses `X.1` / `X.2` for sub-steps)
6. **One sub-step per turn** — never bundle Phase 2.A + 2.B, or multiple flagged customers/invoices/deals in one turn when the skill says one-by-one
7. **Table contract is the spec** — each sub-step lists the **exact tables** to present (column headers fixed). Fill every cell from the named data source; use `—` when data is missing. Do not add metrics, sections, or discussion topics outside that step's tables. **Do not `advance` until the step's contract is complete** (e.g. all work summary tables in `1.1`; scorecard in `1.2`; every hiring pipeline issue in `2.1`; every HJD > 10 customer in `2.3`; every behind CS visit in `2.2`; every stale deal in `3.1`).
8. **One question per turn** — stale deals, invoice reviews, photographer actions, and delegation picks are separate turns
9. **Advance after complete:** `node scripts/workflow-progress.mjs advance --workflow weekly-ops --step <id>`
10. **Phase gate:** `node scripts/workflow-progress.mjs gate --workflow weekly-ops --phase check` before `commit` — `check` FIELD CHECK must pass
11. **Tangents:** fix/interrupt, then resume ledger `current_step` — do not skip ahead

<a id="ledger-step-order-do-not-reorder"></a>
### Ledger step order (do not reorder)

| Order | Step ID | User-facing? |
|-------|---------|----------------|
| 1 | `pre-0` | Yes — confirm planning week + optional Weekly Plan gate |
| 2 | `0` | Silent — ops data pull |
| 3 | `0b` | Yes — RED FLAGS + planning context (no fixes) |
| 4 | `1.1` | Yes — Phase 1-A work summary (volume + busier/slower customers) |
| 5 | `1.2` | Yes — Phase 1-B Pipedrive activity scorecard |
| 6 | `1.3` | Yes — Phase 1-C Work Management (daily records + ops priority) |
| 7 | `2.1` | Yes — Phase 2-A Hiring (job posts, ad health, pipeline actions) |
| 8 | `2.2` | Yes — Phase 2-B CS check-in cadence + behind CS visits |
| 9 | `2.3` | Yes — Phase 2-C High Job Days review + CS dept health |
| 10 | `2.4` | Yes — Phase 2-D Photographer Management (roster + perf behind + monitoring) |
| 11 | `3.1` | Yes — Phase 3-A Aaron sales (gaps → stale → overdue PD → plan week) |
| 12 | `3.2` | Yes — Phase 3-B team sales oversight + behind Pipedrive |
| 13 | `4.1` | Yes — Phase 4 — Finance & Admin (late invoices, one customer per turn) |
| 14 | `6.1` | Yes — Phase 6 — Post Production (QA / delivery backlog) |
| 15 | `6.2` | Yes — Phase 6 — Social Media (pipeline + posting currency) |
| 16 | `7.1` | Yes — Phase 7 — 1:1 meetings + late 1:1 visits |
| 17 | `check` | Yes — ops FIELD CHECK |
| 18 | `commit` | Yes — log rollup + Team Activity Details + approved writes |

**Notion log:** Create at start of `1.1` if not already created (`workflow-notion-log create`). Sync after every `advance`. Write phase fields per `context/systems/workflow-logs.md`.

<a id="table-scope-by-step"></a>
### Table scope by step

| Step | Present only |
|------|----------------|
| `pre-0` | Planning week confirmation table |
| `0b` | RED FLAGS summary + planning context table |
| `1.1` | Tables 1-A (work volume), 1-B (busier customers), 1-C (slower customers) — one table per turn |
| `1.2` | Table 1-D (activity scorecard) |
| `1.3` | Table 1.3-A (daily ops records) + Table 1.3-B (ops priority letter) |
| `2.1` | Hiring — job posts & ad health + applications by week — one pipeline issue per turn when flagged |
| `2.2` | CS behind-summary + check-in cadence table — one behind customer per turn when flagged |
| `2.3` | HJD review + CS dept health — one customer per turn when flagged |
| `2.4` | Photographer roster + perf behind + monitoring — one monitored photographer per turn |
| `3.1` | Aaron sales (gaps → stale deals one-by-one → overdue PD → plan week) |
| `3.2` | Team sales oversight table (stale + overdue PD per member) |
| `4.1` | Late invoice review — one customer per turn |
| `6.1` | Post production currency table |
| `6.2` | Social Media — P&L + per-account posting + pipeline gaps — one flagged account per turn |
| `7.1` | Late 1:1 visits table — one person per turn when overdue |
| `check` | Table check (FIELD CHECK) |
| `commit` | Commit checklist + summary table |

<a id="interaction-style"></a>
## Interaction Style

- **One question at a time.** Never present a wall of choices. Walk through decisions sequentially.
- **Confirm before executing.** Each phase presents proposed Pipedrive/Knack/Todoist/Notion actions, gets approval, then executes before moving on.
- **Exclude Shopping List** (Todoist project `6W36wRPXj8qC2RCc`) from all analysis.
- **Data integrity:** All Knack/Pipedrive/Notion/Todoist write operations require explicit user approval before execution (Todoist case-by-case — never batch or assume).

<a id="required-notion-fields-index"></a>
## Required Notion fields — index

Commit writes target **Weekly Ops Meeting Log** (`379f40c2-487b-8130-916d-eba9ce85134c`).

- **"Last Wk" activity totals** (Table 1-A) → prior entry from **Weekly Ops Log** via `weekly-ops-pull.mjs`; falls back to **Weekly Meeting Log** (`322f40c2-487b-81bd`) until Ops Log has history
- **Monthly Plan Log** (`344f40c2-487b-806d`) — `Priority Stack`, `Domains Parked` for context only (CL sprint retired)

**Gate rules:**
- Before **`commit`**: `check` FIELD CHECK must pass.
- Each phase delivers **only** its table contract (see per-phase **Present** blocks below).

<a id="relationship-to-weekly-plan"></a>
## Relationship to Weekly Plan

| Weekly Plan | Weekly Ops |
|-------------|------------|
| Life review + dev planning (~78 min) | CL ops execution (~40 min) |
| Weekly Meeting Log DB (`322f40c2-487b-81bd`) | Weekly Ops Meeting Log DB (`379f40c2-487b-8130`) |
| `Ops Minutes` habit KPI (retrospective time logged) | Pipedrive activity scorecard (forward accountability) |
| Phases 2.5–2.8 (legacy) | Phases 1–4 here |

Monthly plan reads **Weekly Ops Log** for team activity rollups (not Weekly Meeting Log) once the Ops Log has entries.

<a id="procedure"></a>
## Procedure

<a id="pre-phase-0-planning-week-gate-pre-0"></a>
## Pre-Phase 0: Planning Week Gate (`pre-0`)

**Purpose:** Anchor this session to the correct planning week and optionally verify Weekly Plan ran for the same week.

1. Run `node scripts/planning-dates.mjs` — capture `planning_monday`, `review_week_start`, `review_week_end`.
2. **Present exactly Table pre-0:**

| Field | Value |
|-------|-------|
| Planning week Monday | from `planning-dates.mjs` |
| Review week (last week) | Mon–Sun range |
| Session date (CT) | today |
| Ops log title (when created) | `Week of [planning Monday YYYY-MM-DD]` |

3. Confirm with Aaron: "This ops session is for planning week **[Monday date]** — correct?"
4. **Optional gate — Weekly Plan log:** Query **Weekly Meeting Log** (`322f40c2-487b-81bd`) for an entry whose Name = `Week of [planning Monday YYYY-MM-DD]`.
   - **If found:** Note "Weekly Plan log exists for this week" and continue.
   - **If missing:** Warn: "No Weekly Meeting Log for this planning week — dev/life frame may be unset. Proceed ops-only?" Aaron may continue (not a hard stop) or pause to run Weekly Plan first.
5. **Advance:** `pre-0`

<a id="phase-0-data-pull-silent-before-conversation"></a>
## Phase 0: Data Pull (silent, before conversation)

**Purpose:** Pull all ops data silently. No user-facing output until Phase 0b.

```
node scripts/weekly-ops-pull.mjs [--week-of YYYY-MM-DD]
```

**Output:** `output/weekly-ops-briefing-YYYY-MM-DD.md` (session date, CT). **Read this file** before Phase 0b and throughout Phases 1–4 — do not re-query ad-hoc unless mid-session refresh is needed.

The script wraps `weekly-data-pull.mjs` and prepends **planning context** (Monthly Log Priority Stack, prior Weekly Log activity KPIs for "Last Wk").

<a id="ops-pulls-included-in-briefing"></a>
### Ops pulls included in briefing

Via `weekly-data-pull.mjs` (parallel MCP where applicable):

1. **Todoist:** Overdue tasks (exclude Shopping List) — surfaced in Phase 4 Finance & Admin when admin-related
2. **Pipedrive:**
   - Sales pipeline (pipeline 1), CS pipeline (pipeline 6, full pull), activities (overdue + this week)
   - **Completed activity KPIs:** `pipedrive_get_activities` with `done: "1"` and `updated_since` = Monday of review week (RFC3339); filter client-side by `marked_as_done_time` within the 7-day window
   - **Open activities per user:** pull per-user (user_id: 18865844, 22704318, 20938631, 19274648) — omitting `user_id` only returns the authenticated user
3. **Knack Customers** (`object_2`): field_464 (Current Customer), field_1035 (HJD), field_1601 (Health Status), field_1438 (Account Manager), field_1410 (Invoice Follow Up Status), field_1428 (Adjusted Late Invoices), field_1491 (Aged Invoices)
4. **Knack Photographers** (`object_7`): field_33 (Status), field_1267 (Call-ins Last Month), field_1362 (Avg Issues Per Car), field_1446 (Time Off Requests Last Month), field_1338 (Performance Grade)
5. **Hubstaff:** Last week's hours per member via `hubstaff_get_weekly_report`
6. **CS health / PHOTOGRAPHER MANAGEMENT** sections (script-built flags → RED FLAGS)

**Reconcile completions first** — before Phase 1, note what's already been done since last ops session.

**Advance:** `0` (silent — no user message required, or brief "Data pull complete" only)

<a id="phase-0b-ops-briefing-gate-3-min"></a>
## Phase 0b: Ops Briefing Gate (~3 min)

**Purpose:** Surface RED FLAGS and planning context from the briefing. **Ops briefing only — attempt no fixes.** Do not run sync scripts, backfill Notion, or remediate data gaps here; name gaps and continue with `—` where needed.

1. Read `output/weekly-ops-briefing-YYYY-MM-DD.md` in full.
2. Extract the **RED FLAGS** section (from `weekly-data-pull.mjs` flags) and **Planning context** section (from `weekly-ops-pull.mjs`).
3. **Present exactly these tables:**

**Table 0b-A — RED FLAGS**

| # | Flag | Source |
|---|------|--------|
| 1 | | briefing RED FLAGS |
| … | | (one row per flag; if none: single row "No RED FLAGS") |

**Table 0b-B — Planning context**

| Source | Field | Value |
|--------|-------|-------|
| Monthly Plan Log (latest) | Priority Stack | truncated if long |
| Monthly Plan Log (latest) | Domains Parked | |
| Prior Weekly Meeting Log | Aaron / Lexie / Tristen / Ran / Total Activities | for Table 1-A "Last Wk" |
| Briefing | Todoist overdue count | excl Shopping List |
| Briefing | Pipedrive overdue activities | count |

4. State aloud: "Ops briefing only — no remediation this step. Flags carry into Phases 1–4."
5. **Advance:** `0b`

<a id="phase-1-work-summary-8-min"></a>
## Phase 1: Work Summary (~8 min)

**Purpose:** Open with **operational throughput** — how much work CL processed last week, which customers ramped up or slowed down, then team Pipedrive activity accountability for the review week.

**Data source:** `output/weekly-ops-briefing-YYYY-MM-DD.md` → **KNACK — OPERATIONAL METRICS** (`WORK SUMMARY`, `BUSIER CUSTOMERS`, `SLOWER CUSTOMERS` sections) + Pipedrive pulls for scorecard.

<a id="part-a-work-volume-11-table-1-a-first"></a>
### Part A — Work volume (`1.1`, table 1-A first)

**Present exactly Table 1-A — Work volume (review week):**

| Metric | Review wk | Prior wk | Δ |
|--------|-----------|----------|---|
| Jobs / cars processed | from briefing | from briefing | |
| Avg jobs / day | | | |
| High-volume days (8+ jobs) | list dates or `none` | — | |

<a id="part-b-busier-customers-11-table-1-b-second"></a>
### Part B — Busier customers (`1.1`, table 1-B second)

Customers with the largest **week-over-week increase** in job count (briefing **BUSIER CUSTOMERS**). Include top 5–8 rows; pad with `—` if fewer.

**Present exactly Table 1-B:**

| Customer | Jobs (review wk) | Jobs (prior wk) | Δ | Notes |
|----------|------------------|-----------------|----|-------|
| | | | | |

<a id="part-c-slower-customers-11-table-1-c-third"></a>
### Part C — Slower customers (`1.1`, table 1-C third)

Customers whose volume **dropped materially** (briefing **SLOWER CUSTOMERS** — prior wk ≥ 3 jobs and Δ ≤ −3 or ≥30% decline). Include top 5–8 rows.

**Present exactly Table 1-C:**

| Customer | Jobs (review wk) | Jobs (prior wk) | Δ | Notes |
|----------|------------------|-----------------|----|-------|
| | | | | |

Flag any busier/slower customer also appearing in CS RED FLAGS (HJD, health, invoices) for Phase 2.

**Advance `1.1` after all three tables.**

<a id="part-d-pipedrive-activity-scorecard-12"></a>
### Part D — Pipedrive Activity Scorecard (`1.2`)

Pull completed activities for the **review week** using `pipedrive_get_activities` with `done: "1"` and `updated_since` set to the Monday of the review week. Filter results client-side to activities where `marked_as_done_time` falls within the 7-day window.

**Present exactly Table 1-D:**

```
COMPLETED ACTIVITIES — WEEK-OVER-WEEK
| User    | Stop | Call/Text | Task | Meeting | Total | Last Wk |
|---------|------|-----------|------|---------|-------|---------|
| Aaron   |      |           |      |         |       |         |
| Lexie   |      |           |      |         |       |         |
| Tristen |      |           |      |         |       |         |
| Ran     |      |           |      |         |       |         |
| Total   |      |           |      |         |       |         |
```

"Last Wk" total comes from the **prior Weekly Meeting Log** entry (briefing planning context) until Weekly Ops Log has its own history. Flag any user with **0** completed activities.

**Hold for commit (Weekly Ops Log `379f40c2-487b-8130`):** Aaron Activities, Lexie Activities, Tristen Activities, Ran Activities, Total Activities.

**Advance:** `1.2`

<a id="part-c-work-management-13"></a>
### Part C — Work Management (`1.3`)

**Purpose:** Examine **each business day** in the review week at the business level — throughput, coverage stress, and which **backup** photographers were required — then set org **ops priority** (sales vs hiring vs service delivery) for the planning week.

**Data source:** Briefing **DAILY OPERATIONS RECORDS**, **BACKUPS REQUIRED**, **CL DEPARTMENTS**, **HIRING — KNACK HIRING APP** (ops priority signals). See [`work-management.md`](../../work/chrome-lot/work-management.md).

1. Present **Table 1.3-A — Daily operations records** — one row per day in the review week (from briefing; do not skip days with zero jobs if they were business days):

| Date | Jobs | Business level | Call-ins | Backups used | Unassigned | Notes |
|------|------|----------------|----------|--------------|------------|-------|

2. Summarize **backups required** for the week — which day(s), which backup-tier photographers (`field_1209` contains Backup), and whether coverage was adequate.

3. From briefing **CL DEPARTMENTS** + **HIRING ACTIVE** (Knack hiring app) + Phase 2.1 / Phase 3 context, recommend an **ops priority** for the planning week.

**Table 1.3-B — Ops priority this week** *(single-select — reply with letter)*

| | Option |
|---|--------|
| **A** | **Sales** — AM / pipeline / new business is the binding constraint |
| **B** | **Hiring** — roster / personnel / delegation is the binding constraint *(only if briefing **HIRING ACTIVE: yes**)* |
| **C** | **Service delivery** — field coverage / backups / call-ins dominate *(ops priority label unchanged)* |
| **D** | **Balanced** — no single domain owns the week |

If **HIRING ACTIVE: no**, do not offer **B** — choose A, C, or D only.

→ Hold for commit: Weekly Ops Log **`Ops Priority`** + narrative in **`Ops Summary`**.

**Advance:** `1.3`

<a id="phase-2-hiring-cs-and-photographer-management-25-min"></a>
## Phase 2: Hiring, CS & Photographer Management (~25 min)

**Purpose:** Open ops with **hiring pipeline health** when active — then customer relationships (check-in cadence, HJD) and field photographer management.

<a id="part-a-hiring-21"></a>
### Part A — Hiring (`2.1`)

**Purpose:** Review **active job posts**, the **health of each ad**, and application import cadence — decide actions to keep the pipeline healthy (repost jobs, step in on applicant review, or redelegate review). Covers **Hiring** Dev Project sub-item (may still be named Workload & Hiring in Notion).

**Data source:** Briefing **HIRING — KNACK HIRING APP** (open positions `object_2`, job ads `object_23` field_453, applications `object_4` field_113). See [`work-management.md`](../../work/chrome-lot/work-management.md).

1. Present **Table 2.1-A — Hiring status:**

| HIRING ACTIVE | Open positions | Review-wk apps | Pipeline flags (briefing) |
|---------------|----------------|----------------|---------------------------|

2. **If HIRING ACTIVE: no** — note "Hiring N/A — no open positions"; skip Tables 2.1-B–D; still confirm **Table 2.1-H** dept health if Aaron wants baseline confirm; **advance `2.1`**.

3. **If HIRING ACTIVE: yes** — present **Table 2.1-B — Active job posts & ad health** (one row per job ad linked to an open position):

| Position | Ad health (field_453) | Review-wk apps (position) | Healthy? |
|----------|----------------------|---------------------------|----------|

Flag: **Unhealthy** or blank ad health; open position with **no linked job ad** (briefing RED FLAG).

4. Present **Table 2.1-C — Applications by week** (field_113 import date):

| Week of (Mon) | Apps imported | Healthy? |
|---------------|---------------|----------|

Healthy = regular imports while hiring (`≥3`/wk good; `0` while active = unhealthy).

5. Review **one pipeline issue per turn** when any of: unhealthy/blank ad, missing ad on open req, zero-import week while hiring active, review week zero apps:

**Table 2.1-D — Hiring action** *(one row per turn)*

| Issue | Position / week | Evidence | Action proposed |
|-------|-----------------|----------|-----------------|

Lettered options per turn: **A** Repost / refresh job ad · **B** Step in — Aaron reviews applicants this week · **C** Redelegate applicant review (name owner + handoff) · **D** Monitor — no action · **E** Defer — log reason in `Key Decisions`

If **zero** pipeline issues: single row `Hiring pipeline healthy — no actions` and continue.

6. **Department health — Delegation & Hiring + Personnel Management:** Present **Table 2.1-H** — one row per department:

| Department | Health | Priority | Change? | Reason |
|------------|--------|----------|---------|--------|

Confirm or update each on CL Departments with approval (one department per turn when changing).

**Outputs:** Todoist tasks for repost/review handoffs; Process Street **Daily Hiring Follow Up** nudge if standing workflow stale (case-by-case approval).

**Advance:** `2.1`

<a id="part-b-check-in-cadence-22"></a>
### Part B — Check-in Cadence (`2.2`)

**Behind CS visits (mandatory):**

1. Pull open CS deals (pipeline 6) with **60+ days since last activity** — from briefing RED FLAGS / Pipedrive pull.
2. Pull **overdue CS activities** (due before today, not done) — group by owner.
3. Present **Table 2-A0 — Behind CS summary** before planning the batch:

| Signal | Count | Oldest (days) | Owner(s) |
|--------|-------|---------------|----------|
| CS deals 60+ days stale | | | |
| Overdue CS activities | | | |

4. Review **one behind customer per turn** when count > 0 (most stale first):

**Table 2-A1 — Behind CS visit** *(one row per turn)*

| Customer | Days since activity | Overdue activities | Health | Action proposed |
|----------|---------------------|--------------------|--------|-----------------|

If **zero** behind CS signals: single row `No behind CS flags` and continue to batch planning.

**Check-in batch planning:**

1. **Cross-reference Knack & Pipedrive:** Current customers in Knack (field_464=Yes) vs. open CS deals in Pipedrive (pipeline 6). Flag mismatches.
2. **Health status review:** Check field_1601 for "Needs Attention" or "At Risk" accounts.
3. **Propose weekly check-in batch:** ~6–9 stops, prioritized by:
   - Behind customers from Table 2-A1 not yet resolved
   - AT RISK / Needs Attention stage first
   - Days overdue (most stale first)
   - HJD > 10 (from Phase 2.3 list)
   - Account value
   - Grouped by geography where possible
4. **Delegation:** Assign stops to Tristen, Lexie, Ran, or Aaron based on deal ownership. Spread across the week.

**Present exactly Table 2-A — CS check-in batch:**

| # | Customer | Stage | Days since activity | HJD | Health | Owner | Assigned to |
|---|----------|-------|---------------------|-----|--------|-------|-------------|
| | | | | | | | |

Propose Pipedrive stop activities — **case-by-case approval** before create. Cap: **max 3 Pipedrive activities per day** across all phases.

**Advance:** `2.2`

<a id="part-c-high-job-days-review-23"></a>
### Part C — High Job Days Review (`2.3`)

**Purpose:** Dedicated HJD pass — customers going too long without photos (`field_1035` **display value in days**; flag **> 10**, watch **8–10**).

**Data source:** Briefing **CS HEALTH** → `HIGH JOB DAYS (>10)` and `HJD WATCH (8–10 days)` sections.

1. List all current customers with **HJD > 10** (sorted highest first).
2. Optionally note **HJD 8–10** watch list — no action required unless Aaron wants proactive outreach.
3. Review **one customer per turn** when HJD > 10: root cause (slow week vs. churn risk), tie to Table 1-B/1-C volume if relevant, propose CS stop or AM call.

**HJD rules:**
- Read **`field_1035` display** (days). Do **not** compare `field_1035_raw` (seconds).
- **> 10** → must review in this phase
- **8–10** → watch list only unless also AT RISK / Needs Attention

**Present one customer per turn — Table 2.3-B:**

| Customer | HJD (days) | AM | Volume Δ (Phase 1) | Health | Action proposed |
|----------|------------|-----|-------------------|--------|-----------------|

If **zero** customers HJD > 10: single row `No HJD flags` and continue.

**Knack reference:** `field_1035` High Job Days — see [`knack-fields.md`](../../systems/knack-fields.md).

**Department health — Customer Service:** From briefing **CL DEPARTMENTS**. Compare to Phase 2 evidence. Present **Table 2.3-H**:

| Health | Priority | Change? | Reason |
|--------|----------|---------|--------|

If **Health** or **Priority** changes: lettered confirm → **`personal_notion_update_page`** on CL Departments (`341f40c2-487b-80cc-976e-c0e86b79e3fa`) with approval. If unchanged: confirm and advance.

**Advance:** `2.3`

<a id="part-d-photographer-management-24"></a>
### Part D — Photographer Management (`2.4`)

**Purpose:** Field photographer roster, perf-meeting cadence, and active monitoring — covers **Photographer Management** Dev Project sub-item (formerly Service Delivery).

**Data source:** Briefing **PHOTOGRAPHER MANAGEMENT** (Knack `object_7` roster, `field_985` since 1:1, monitoring flags). See [`operations.md`](../../work/chrome-lot/operations.md).

1. Present **Table 2.4-C0 — Photographer summary** from briefing:

| Signal | Count |
|--------|-------|
| Active photographers (roster) | |
| Perf meetings behind (≥30 days since 1:1) | |
| Under active monitoring | |

2. Present **Table 2.4-C1 — Roster summary** (all active photographers from briefing):

| Name | Rank | Grade | Jobs (review wk) | Call-ins | Issues/car | Since 1:1 (days) | Monitoring |
|------|------|-------|------------------|----------|------------|------------------|------------|

3. Present **Table 2.4-C2 — Perf meetings behind** when count > 0:

| Photographer | Days since 1:1 | AM | Action proposed |
|--------------|----------------|-----|-----------------|

4. Review **one monitored photographer per turn** when active monitoring count > 0 — pull latest `object_57` comment (`field_1006` = Photographer) if action needed:

**Table 2.4-C3 — Monitored photographer** *(one row per turn)*

| Name | Flags | Grade | Latest comment | Action |
|------|-------|-------|----------------|--------|

If **zero** perf-behind and **zero** monitoring: note "Photographer roster confirmed — no flags" and continue.

**Department health — Photographer Management:** Present **Table 2.4-H** (same contract as Table 2.3-H). Confirm or update on CL Departments with approval.

**Advance:** `2.4`

<a id="phase-3-sales-management-12-min"></a>
## Phase 3: Sales Management (~12 min)

**Purpose:** Drive new revenue and clear **behind Pipedrive** debt — deal gaps, overdue activities, and stale sales deals — before planning the week.

<a id="part-a-aarons-sales-activity-31"></a>
### Part A — Aaron's Sales Activity (`3.1`)

**Behind Pipedrive (Aaron — mandatory):**

Track and clear before planning new stops:
- Open deals with **no `next_activity_date`** (deal gaps)
- **Overdue activities** on Aaron-owned deals (all pipelines)
- **Stale sales deals** (Stale label or 14+ days no activity)

**A0 — Deal gaps cleanup (mandatory, before planning stops)**

1. Pre-pull Aaron-owned open deals in Sales (pipeline 1), CS (pipeline 6), and Social Media (pipeline 13) with **no `next_activity_date`** via Pipedrive MCP. Present as a numbered checklist.
2. Instruct Aaron to run CL Bot **`deal gaps`** in Teams ([`cl-bot.md`](../../systems/notion-guides/cl-bot.md)) and schedule every gap via the Adaptive Card **Schedule** buttons (+3 days default).
3. **Gate:** Do not plan new sales stops until gaps = 0. If Aaron explicitly defers a deal, log the deal name + reason in commit `Key Decisions` and exclude it from the gap count.

**Present Table 3-A0 — Deal gaps:**

| # | Deal | Pipeline | Action |
|---|------|----------|--------|
| | | | Schedule / Defer |

**A0b — Overdue Pipedrive activities (Aaron)**

1. Pull Aaron-owned **open overdue activities** from briefing / Pipedrive (due before today, not done).
2. Present **Table 3-A0b — Overdue PD (Aaron)** — one activity per turn when count > 0:

| Activity | Deal | Pipeline | Days overdue | Action |
|----------|------|----------|--------------|--------|

If **zero** overdue: single row `No overdue PD flags` and continue.

**A1 — Stale deal review (one-by-one)**

1. Pull Aaron-owned **Sales Pipeline 1** deals with the **Stale** label (same signal as bot `stale deals`). Fallback if label missing: open deals with no activity in 14+ days.
2. Present **one deal at a time** — lettered options: **A** Keep active · **B** Move to cold pool (Pipeline 12) · **C** Defer decision.
3. **Cold sales pool** = Pipeline 12 **Not Actively Working** ([`pipedrive.md`](../../systems/pipedrive.md)). Each **Move to Pipeline 12** requires case-by-case Pipedrive approval before `pipedrive_update_deal` — never batch-execute.
4. Deferrals carry to next weekly ops; do not re-present deferred deals unless still stale.

**Present one stale deal per turn — Table 3-A1:**

| Deal | Pipeline | Days stale | Last activity | Action |
|------|----------|------------|---------------|--------|

**A2 — Plan the week**

1. Review Pipedrive sales pipeline (pipeline 1): new leads, remaining active stale deals, overdue activities
2. Plan Aaron's sales stops for the week (realistically 2–3)
3. Route planning: use Mapsly manually to cluster stops geographically
4. Review any rescheduled-3× activities — decide: do, delegate, or drop
5. Accountability: activities completed last week vs. planned

**Present Table 3-A2 — Aaron's week plan:**

| Day (optional) | Stop / activity | Deal | Notes |
|----------------|-----------------|------|-------|

**Advance:** `3.1`

<a id="part-b-team-sales-oversight-32"></a>
### Part B — Team Sales Oversight (`3.2`)

**Behind Pipedrive (team — mandatory):**

1. Pull sales pipeline deals owned by Tristen, Lexie, and Ran
2. Review each team member's sales activity: deals in progress, **overdue activities**, **stale leads (14+ days)**
3. Flag any deals with no activity in 14+ days
4. Propose reassignments or follow-up nudges where needed
5. Create Pipedrive activities for team members as needed (approval per activity)

**Present exactly Table 3-B — Team behind + oversight:**

| Team member | Active deals | Overdue activities | Stale (14+ d) | Proposed action |
|-------------|--------------|-------------------|---------------|-----------------|

**Outputs:** Pipedrive activities for Aaron's sales stops. Pipedrive activities or nudges for team sales. Todoist tasks for follow-ups (case-by-case approval).

**Department health — Account Management:** Present **Table 3-H** (same contract as Table 2.3-H). Confirm or update on CL Departments with approval.

**Advance:** `3.2`

<a id="phase-4-finance-and-admin-8-min"></a>
## Phase 4: Finance & Admin (~8 min)

**Purpose:** Invoice accountability and admin currency — covers **Finance & Admin** Dev Project sub-item.

**Data source:** `output/weekly-ops-briefing-YYYY-MM-DD.md` (Knack invoice fields, outstanding invoice aging, Todoist overdue admin tasks).

<a id="part-a-late-invoice-review-41"></a>
### Part A — Late Invoice Review (`4.1`)

1. Pull customers where field_1410 (Invoice Follow Up Status) = "Account Manager Follow Up" or "Max Escalation"
2. Pull customers where field_1428 (Adjusted Late Invoices) > 3 OR field_1491 (Aged Invoices) > 1
3. Combine both lists (deduplicate). Review each flagged customer **one by one**.
4. Create Pipedrive activities (escalation follow-up) assigned to either Aaron or the account manager (field_1438) — approval per activity.

**Invoice escalation trigger rule:**
- field_1410 = "Account Manager Follow Up" or "Max Escalation" → always flag
- field_1428 (Adjusted Late Invoices) > 3 → flag
- field_1491 (Aged Invoices) > 1 → flag
- Any one of these triggers a review and Pipedrive activity

**Present one customer per turn — Table 4-A:**

| Customer | AM | field_1410 | Adj Late | Aged | Action proposed |
|----------|-----|------------|----------|------|-----------------|

**Knack Customer Field Reference**

| Field | Name | Trigger |
|-------|------|---------|
| field_464 | Current Customer | Filter active |
| field_1438 | Account Manager | Determine delegation |
| field_1410 | Invoice Follow Up Status | Flag "AM Follow Up" / "Max Escalation" |
| field_1428 | Adjusted Late Invoices | Flag if > 3 |
| field_1491 | Aged Invoices | Flag if > 1 |

**Outputs:** Pipedrive escalation activities for invoice issues. Todoist admin follow-ups if needed (case-by-case approval).

**Department health — Admin:** Present **Table 4-H** (same contract as Table 2.3-H). Confirm or update on CL Departments with approval.

**Advance:** `4.1`

<a id="phase-6-post-production-5-min"></a>
## Phase 6: Post Production (~5 min)

**Purpose:** Editor/QA delivery backlog — covers **Post Production** Dev Project sub-item.

**Data source:** Briefing RED FLAGS + `weekly-data-pull.mjs` sections: **QA ISSUES LAST 7 DAYS**, outstanding invoice aging (editor billing context), any reshoot flags.

**Present exactly Table 6-A — Post production currency:**

| Signal | Count / detail | Trend vs last wk | Action proposed |
|--------|----------------|------------------|-----------------|
| Unreviewed QA issues (7d) | from briefing | | |
| QA issues by severity | from briefing | | |
| Open reshoot backlog | from briefing/Knack flags | | |
| Post-prod SLA breaches | — if not in pull | | |

Review flagged rows **one signal at a time** if any require action. Create Todoist tasks or Knack follow-ups with approval.

**Advance:** `6`

<a id="phase-6-b-social-media-10-min"></a>
## Phase 6-B: Social Media (~10 min)

**Purpose:** Social product **P&L** (billing vs expenses), **per-account posting coverage**, and pipeline currency — covers **Social Media** Dev Project sub-item. **Behind tracking:** active accounts with **zero posts in review week**, plus deal gaps / stale PD activity — one flagged account per turn.

**Data source:** Briefing **SOCIAL MEDIA — PIPELINE & POSTING** (P&L, active account posting table, Pipedrive pipeline 13 + Knack `object_65` / `object_21` / `object_25`). See [`social-media.md`](../../work/chrome-lot/social-media.md).

1. Present **Table 6-B0 — Social P&L** from briefing **SOCIAL P&L** + **SOCIAL LABOR DETAIL** sections:

| Metric | Review week | Monthly run-rate |
|--------|-------------|------------------|
| Active accounts | | |
| Billing | line items (`object_21` field_266) | confirmed monthly per active account (override → field_281 → field_459) |
| Labor (Han / Kadyn / Naids) | Hubstaff hours × Airtable Hourly Rate | review-week labor × 52/12 |
| Buffer subscription | monthly ÷ 52/12 | `SOCIAL_BUFFER_MONTHLY_USD` secret |
| Total expenses | labor + Buffer | labor est + Buffer |
| Margin | billing − expenses | billing − expenses |

Note when review-week **billing** line items are $0 — use monthly billing run-rate. Expenses always use **labor + Buffer** (not Knack unit cost).

1b. Present **Table 6-B0b — Billing by account** from briefing **SOCIAL BILLING BY ACCOUNT** (current month):

| PD account | Knack match | Monthly | Wk … (Mon-start) | MTD actual | Source |
|------------|-------------|---------|------------------|------------|--------|

Flag accounts with **Monthly = $0** or MTD actual far below expected weekly cadence.

2. Present **Table 6-B1 — Active account posting** (all active PD accounts — one row each):

| Customer | Stage | Owner | Posts (review wk) | Status |
|----------|-------|-------|-------------------|--------|

**Status:** `OK` if ≥1 post dated in review week (`field_975`); `MISSING` if zero. Review **one MISSING account per turn**:

**Table 6-B2 — Missing posts** *(one row per turn)*

| Customer | Stage | Owner | Action proposed |
|----------|-------|-------|-----------------|

Lettered options: **A** Schedule/clear backlog this week · **B** Customer paused — move PD stage / note · **C** Data gap — posts exist off-system · **D** Defer — log reason

If **zero** MISSING rows: single row `All active accounts posted in review week` and continue.

3. Present **Table 6-B3 — Pipeline summary** (deal currency):

| Signal | Count | Notes |
|--------|-------|-------|
| Inactive Sales | | |
| Mis-staged deals | | |
| Deal gaps (active, no next activity) | | |
| Stale 60+ days (active) | | |

4. Present **Table 6-B4 — Posts by week** (aggregate throughput):

| Week of (Mon) | Posts | Review week? |
|---------------|-------|--------------|

5. When **deal gaps** or **stale active** count > 0 — review **one account per turn** (after missing-post clears):

**Table 6-B5 — Behind social account** *(one row per turn)*

| Customer | Stage | Days stale / gap | Owner | Action proposed |
|----------|-------|------------------|-------|-----------------|

If **zero** gap, stale, and missing-post flags: note pipeline confirmed and continue to dept health.

**Department health — Social Media:** Present **Table 6.2-H** (same contract as Table 2.3-H). Confirm or update on CL Departments with approval.

**Advance:** `6.2`

<a id="phase-7-11-meetings-5-min"></a>
## Phase 7: 1:1 Meetings (~5 min)

**Purpose:** Relationship cadence and scheduled check-ins — covers **1:1 meetings** Dev Project sub-item (including sub-tasks like remote transitions). **Behind tracking:** overdue 1:1 visits live here — not in a central currency pass.

**Data source:** `context/people/index.md` (1:1 cadence + last-meeting dates) + briefing RED FLAGS if present + Dev Projects sub-items under **1:1 meetings** with `This Week` checked.

1. Pull everyone **overdue for a 1:1** per people directory cadence (days since last 1:1 vs. target interval).
2. Present **Table 7-A0 — Late 1:1 summary:**

| Person | Days overdue | Last 1:1 | Target cadence | Dev Project sub-item? |
|--------|--------------|----------|----------------|----------------------|

3. Review **one overdue person per turn** when count > 0 — schedule, delegate prep, or explicit defer with reason in `Key Decisions`.
4. Cross-check Dev Projects **1:1 meetings** sub-items flagged `This Week`
5. Pick top 1–2 overdue 1:1s (and any required transition meetings) to schedule this week
6. Any delegation decisions from earlier phases that need to be communicated in 1:1s

**Present one person per turn when overdue — Table 7-A1:**

| Person / topic | Days overdue | Action this week |
|----------------|--------------|------------------|

If **zero** overdue 1:1s: single row `No late 1:1 flags` and advance.

**Outputs:** Calendar events for 1:1 meetings. Teams messages if needed.

**Advance:** `7.1`

<a id="check-ops-field-check-check"></a>
## Check: Ops FIELD CHECK (`check`)

**Purpose:** Hard stop before commit. Verify all ops decisions are captured and KPI fields are ready to write.

**Present exactly Table check:**

| Group | Required fields / artifacts | Status |
|-------|----------------------------|--------|
| Phase 1 | Work volume + busier/slower customers (Tables 1-A–C) | |
| Phase 1 | Aaron/Lexie/Tristen/Ran/Total Activities computed | |
| Phase 1 | Daily ops records reviewed + Ops Priority chosen | |
| Phase 2 | Hiring job posts reviewed + pipeline actions (or N/A if not hiring) | |
| Phase 2 | Delegation & Hiring + Personnel Management dept health confirmed or updated | |
| Phase 2 | All behind CS visits reviewed (or none flagged) | |
| Phase 2 | CS check-in batch proposed (or N/A + reason) | |
| Phase 2 | All HJD > 10 customers reviewed (or none flagged) | |
| Phase 2 | Customer Service dept health confirmed or updated | |
| Phase 2 | Photographer roster + perf behind reviewed | |
| Phase 2 | All monitored photographers reviewed (or none flagged) | |
| Phase 2 | Photographer Management dept health confirmed or updated | |
| Phase 3 | Deal gaps = 0 (or deferrals logged) | |
| Phase 3 | Aaron overdue PD + stale deals reviewed | |
| Phase 3 | Aaron sales week plan | |
| Phase 3 | Team behind Pipedrive reviewed | |
| Phase 3 | Account Management dept health confirmed or updated | |
| Phase 4 | Invoice escalations decided | |
| Phase 4 | Admin dept health confirmed or updated | |
| Phase 6 | Post production signals reviewed | |
| Phase 6 | Social P&L reviewed (billing vs expenses) | |
| Phase 6 | All active accounts posting reviewed — MISSING accounts actioned or N/A | |
| Phase 6 | Social Media pipeline + deal gaps/stale reviewed (or N/A + reason) | |
| Phase 6 | Social Media dept health confirmed or updated | |
| Phase 7 | All late 1:1s reviewed (or none flagged) | |
| Phase 7 | 1:1s picked for scheduling | |
| Pending writes | Pipedrive / Knack / Todoist / Calendar / Notion list | |

**Do not proceed to `commit` until Table check passes** (or Aaron approves documented N/A per row).

Run: `node scripts/workflow-progress.mjs gate --workflow weekly-ops --phase check`

**Advance:** `check`

<a id="commit-5-min"></a>
## Commit (~5 min)

**Purpose:** Create Weekly Ops Log entry, final capacity check, execute remaining actions, append Team Activity Details, log everything.

**Weekly Ops Meeting Log DB:** `379f40c2-487b-8130-916d-eba9ce85134c` — see `notion-databases.md`.

<a id="commit-procedure"></a>
### Commit procedure

1. **Create Weekly Ops Log entry:** Name = `Week of [planning Monday YYYY-MM-DD]`, Meeting Date = today (CT). Use `personal_notion_create_database_entry` on Weekly Ops Meeting Log DB.
2. **Summary table:** Everything planned across Phases 1–7 (hiring actions, CS stops, behind items cleared, sales stops, invoice escalations, photographer actions, 1:1s)

**Table commit-A — Ops summary**

| Area | This week's commitments | Owner |
|------|------------------------|-------|
| Ops priority | from Phase 1.3 | Aaron |
| Backups / coverage | days + actions from 1.3 | |
| Hiring | job posts / review handoffs from 2.1 | |
| CS check-ins | count + assignees | |
| Behind CS cleared | count resolved / deferred | |
| Invoice escalations | count | |
| Aaron sales stops | count | |
| Behind PD (Aaron) | overdue + stale resolved | |
| Team sales nudges | | |
| Photographer actions | from Phase 2.4 | |
| Post production / QA | actions from 6.1 | |
| Social Media | P&L + posting + gap/stale actions from 6.2 | |
| Late 1:1s scheduled | | |
| Department health | all in-phase confirms/updates done | |

3. **Final capacity check:** Total planned ops hours (stops, calls, reviews) vs. available ops capacity for the week. If total exceeds realistic ops hours minus buffer, something must move. This is non-negotiable.
4. **Store activity KPIs on Weekly Ops Log:** Write `Aaron Activities`, `Lexie Activities`, `Tristen Activities`, `Ran Activities`, `Total Activities` (from Table 1-D).
5. **Store context fields on Weekly Ops Log:**
   - `Ops Priority` — select from Phase 1.3 (Sales / Hiring / Service delivery / Balanced)
   - `Ops Summary` — rich_text narrative from Table commit-A (include any department health changes)
   - `Key Decisions` — deal deferrals, cold-pool moves, behind-CS/PD/1:1 deferrals, photographer outcomes, department health updates
   - `Department Health Snapshot` — optional; only if Aaron wants a consolidated table — otherwise health lives in phase notes + Ops Summary
   - `Action Items` — bullet list of open follow-ups
6. **Append per-user Pipedrive detail sections** to the Weekly Ops Log page using `personal_notion_append_blocks`. Use Pipedrive data from Phase 0 (completed activities from review week + all open activities per user). Append:

   ```
   ---
   # Team Activity Details

   ## Aaron Hoegenauer

   **Completed Activities (X total)**
   - [Stop] CS Check-in: Karma DFW — Karma DFW CS — Apr 8
   - [Task] Invoice follow-up — iDrive1 CS — Apr 10
   ...

   **Open / Overdue Activities (Y total)**
   - [OVERDUE] Follow up with Nabil — Tradeline CS — due Apr 7
   - [Call/Text] Follow up with Marios — DFW Motorcars CS — due Apr 14
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

7. **Execute remaining:** Create any Todoist/Calendar/Pipedrive/Knack items not yet committed during Phases 1–7 (each with prior approval).
8. **Verify FIELD CHECK** — re-run Table check; confirm nothing is blank without N/A + reason.

**Advance:** `commit` — workflow complete.

<a id="cross-cutting-rules"></a>
## Cross-Cutting Rules

- **Table contract per phase.** Ledger `current_step` determines which tables are in scope. Phase 1 = work summary (1.1) → scorecard (1.2) → work management (1.3). Phase 2 = Hiring (2.1) → CS (2.2) → HJD + CS dept health (2.3) → photographer mgmt + dept health (2.4). Phase 3 = sales (3.1–3.2) + Account Management health. Phases 4–7 per domain with in-phase dept health where mapped. Phase 6 = post production (6.1) → social media (6.2).
- **Behind tracking in domain sections.** Unhealthy hiring pipeline → Phase 2.1. Stale CS visits → Phase 2.2. HJD → Phase 2.3. Photographer monitoring + perf behind → Phase 2.4. Stale sales + overdue Pipedrive → Phase 3.1–3.2. Social Media missing posts + gaps/stale → Phase 6.2. Late 1:1 visits → Phase 7.1.
- **FIELD CHECK gate.** `check` must pass before `commit`.
- **Route every item into a bucket.** Each surfaced item is Automated (n8n), Delegated (team 1:1s), or a Scheduled slice (calendar + Todoist mirror).
- **Capacity is non-negotiable.** If total planned ops work exceeds available ops hours minus 10–15% buffer, the system pushes back. Something must move.
- **Delegation by default.** For any task deferred 3+ times, suggest delegation before rescheduling. Use the delegation framework in `context/systems/capacity-rules.md`.
- **Max 3 Pipedrive activities per day.** Cap at sustainable levels across CS, sales, and escalations combined.
- **Max 5 must-do Todoist tasks per day** (P1/P2). If morning briefing shows >5, defer.
- **Pipedrive accountability is #1 neglected area.** Always check activities completed vs. planned (Table 1-D scorecard).
- **All data pulls happen in Phase 0 silently.** ~40 minutes is for discussion and decisions.
- **Ops briefing only in 0b.** Unlike Weekly Plan Phase 0b, do not attempt data remediation — flag and proceed.
- **Weekly Ops Log is separate from Weekly Meeting Log.** Do not write ops KPIs to the Weekly Meeting Log once Ops Log exists.

<a id="outputs"></a>
## Outputs

- **pre-0:** Planning week confirmed; optional Weekly Plan log gate result.
- **Phase 0:** `output/weekly-ops-briefing-YYYY-MM-DD.md` generated.
- **Phase 0b:** RED FLAGS + planning context presented; no fixes attempted.
- **Phase 1:** Work summary + activity scorecard + daily ops records + ops priority.
- **Phase 2:** Hiring pipeline actions + behind CS + HJD + photographer management decisions.
- **Phase 3:** Deal gaps cleared + overdue PD + stale deals + Aaron sales plan + team behind PD actions.
- **Phase 4:** Invoice escalation decisions.
- **Phase 6:** Post production / QA actions + Social Media pipeline/posting review.
- **Phase 7:** Late 1:1 review + scheduling decisions.
- **check:** Ops FIELD CHECK pass.
- **commit:** Weekly Ops Log entry + Team Activity Details + approved external writes.

<a id="failure-modes-and-graceful-degradation"></a>
## Failure modes & graceful degradation

- **Weekly Plan log missing for planning week:** Warn in pre-0; Aaron may proceed ops-only or pause for Weekly Plan.
- **weekly-ops-pull / weekly-data-pull script failure:** Fall back to parallel MCP pulls per Phase 0 list; note missing sections in Table 0b-A.
- **Prior Weekly Meeting Log missing activity KPIs:** Render "Last Wk" column as `—`; note in 0b footer.
- **Pipedrive MCP rate limits:** Present partial scorecard; retry per-user pulls; document gaps in `Key Decisions`.
- **Knack unavailable:** Skip photographer section (Phase 2.4); CS invoice fields from last briefing cache if present; otherwise defer Phase 4 with documented N/A. Post production (Phase 6) may defer if QA pull missing.

<a id="see-also"></a>
## See also

- `../../router.md`
- `../weekly-planning/SKILL.md` — life + dev (separate session, same planning week)
- `../monthly-plan/SKILL.md` — monthly plan reads Weekly Ops Log for team rollups
- `../../workflow-execution.md` — execution protocol
- `../../systems/cadences.md`
- `../../systems/capacity-rules.md`
- `../../systems/notion-databases.md`
- `../../systems/pipedrive.md`
- `../../systems/knack-fields.md`
- `../../systems/hubstaff.md`
- `../../people/index.md`
- `../../work/chrome-lot/customer-service.md`, `../../work/chrome-lot/sales.md`, `../../work/chrome-lot/operations.md`