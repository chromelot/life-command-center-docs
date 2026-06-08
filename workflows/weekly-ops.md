> **Source:** [`context/skills/weekly-ops/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/weekly-ops/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Weekly Ops — Chrome Lot Operations Review

## Trigger

This skill activates when Aaron says **"weekly ops"**, **"ops review"**, **"CL ops meeting"**, or **"operations meeting"**.

**Target duration:** ~40 min.

**Separate session** from Weekly Plan — typically a different day the same planning week. Weekly Plan covers life review + development planning; Weekly Ops covers CL operating execution (Pipedrive accountability, CS, sales, people).

## Inputs

Load via the router. Read these before starting:

- `context/systems/notion-databases.md` — DB IDs (Weekly Ops Meeting Log `379f40c2-487b-8130`, Weekly Meeting Log, Monthly Meeting Log)
- `context/systems/pipedrive.md` — pipeline IDs, stages, user IDs, completion-time API quirks
- `context/systems/knack-fields.md` — Customer + Photographer field references
- `context/systems/hubstaff.md` — member IDs, weekly-report tool
- `context/systems/capacity-rules.md` — limits, overcommitment triggers, max PD activities/day
- `context/work/chrome-lot/customer-service.md` — Phase 2 CS logic
- `context/work/chrome-lot/sales.md` — Phase 3 sales logic
- `context/work/chrome-lot/operations.md` — Phase 4 photographer review logic
- `context/people/index.md` — delegation matrix, 1:1 tracking

## Execution Protocol (mandatory — read `context/workflow-execution.md`)

1. **Date context (every turn, before any weekday or "this week" language):**
   ```
   node scripts/planning-dates.mjs [--today=YYYY-MM-DD] [--week-of=<ledger week_of>]
   ```
   Aaron may run weekly ops on **any day** — never assume session day = Monday. Use script output for review week, planning week, and weekday→ISO mapping. `Today's date` from `user_info` must match `--today` (CT). Planning week Monday must match the same week as Weekly Plan when both run in the same cycle.
2. **Init ledger** at session start: `node scripts/workflow-progress.mjs init --workflow weekly-ops` — omit `--week-of` to auto-set canonical planning Monday from today; if set manually, must be a Monday and should match `planning-dates.mjs`. Ops log title = `Week of <planning Monday>`.
3. **Every turn:** `node scripts/workflow-progress.mjs status --workflow weekly-ops` — present only `current_step` (status includes date warnings if ledger `week_of` is wrong)
4. **Phase banner** on every user-facing message: `**[Weekly Ops · Phase X — title]**` (use `X.A` / `X.B` for sub-steps within a phase)
5. **One sub-step per turn** — never bundle Phase 2.A + 2.B, or multiple flagged customers/invoices/deals in one turn when the skill says one-by-one
6. **Table contract is the spec** — each sub-step lists the **exact tables** to present (column headers fixed). Fill every cell from the named data source; use `—` when data is missing. Do not add metrics, sections, or discussion topics outside that step's tables.
7. **One question per turn** — stale deals, invoice reviews, photographer actions, and delegation picks are separate turns
8. **Advance after complete:** `node scripts/workflow-progress.mjs advance --workflow weekly-ops --step <id>`
9. **Phase gate:** `node scripts/workflow-progress.mjs gate --workflow weekly-ops --phase check` before `commit` — `check` FIELD CHECK must pass
10. **Tangents:** fix/interrupt, then resume ledger `current_step` — do not skip ahead

### Ledger step order (do not reorder)

| Order | Step ID | User-facing? |
|-------|---------|----------------|
| 1 | `pre-0` | Yes — confirm planning week + optional Weekly Plan gate |
| 2 | `0` | Silent — ops data pull |
| 3 | `0b` | Yes — RED FLAGS + planning context (no fixes) |
| 4 | `1` | Yes — CL Operations (scorecard + currency) |
| 5 | `2` | Yes — CS Management |
| 6 | `3` | Yes — Sales Management |
| 7 | `4` | Yes — People Management |
| 8 | `check` | Yes — ops FIELD CHECK |
| 9 | `commit` | Yes — log entry + Team Activity Details + approved writes |

### Table scope by step

| Step | Present only |
|------|----------------|
| `pre-0` | Planning week confirmation table |
| `0b` | RED FLAGS summary + planning context table |
| `1` | Table 1-A (activity scorecard), then Table 1-B (CL currency) — separate turns |
| `2.A` | CS check-in cadence tables |
| `2.B` | Late invoice review — one customer per turn |
| `3.A` | Aaron sales (gaps → stale → plan week) — one deal per turn for stale review |
| `3.B` | Team sales oversight table |
| `4.A` | Flagged photographers — one per turn |
| `4.B` | Staff / 1:1 / Hubstaff table |
| `check` | Table check (FIELD CHECK) |
| `commit` | Commit checklist + summary table |

## Interaction Style

- **One question at a time.** Never present a wall of choices. Walk through decisions sequentially.
- **Confirm before executing.** Each phase presents proposed Pipedrive/Knack/Todoist/Notion actions, gets approval, then executes before moving on.
- **Exclude Shopping List** (Todoist project `6W36wRPXj8qC2RCc`) from all analysis.
- **Data integrity:** All Knack/Pipedrive/Notion/Todoist write operations require explicit user approval before execution (Todoist case-by-case — never batch or assume).

## Required Notion fields — index

Commit writes target **Weekly Ops Meeting Log** (`379f40c2-487b-8130-916d-eba9ce85134c`).

- **"Last Wk" activity totals** (Table 1-A) → prior entry from **Weekly Ops Log** via `weekly-ops-pull.mjs`; falls back to **Weekly Meeting Log** (`322f40c2-487b-81bd`) until Ops Log has history
- **Active CL Sprint** → read from **Monthly Meeting Log** (`344f40c2-487b-806d-98b2-ef710856bd07`) latest entry (`Active CL Sprint` field)

**Gate rules:**
- Before **`commit`**: `check` FIELD CHECK must pass.
- Each phase delivers **only** its table contract (see per-phase **Present** blocks below).

## Relationship to Weekly Plan

| Weekly Plan | Weekly Ops |
|-------------|------------|
| Life review + dev planning (~78 min) | CL ops execution (~40 min) |
| Weekly Meeting Log DB (`322f40c2-487b-81bd`) | Weekly Ops Meeting Log DB (`379f40c2-487b-8130`) |
| `Ops Minutes` habit KPI (retrospective time logged) | Pipedrive activity scorecard (forward accountability) |
| Phases 2.5–2.8 (legacy) | Phases 1–4 here |

Monthly plan reads **Weekly Ops Log** for team activity rollups (not Weekly Meeting Log) once the Ops Log has entries.

## Procedure

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

## Phase 0: Data Pull (silent, before conversation)

**Purpose:** Pull all ops data silently. No user-facing output until Phase 0b.

```
node scripts/weekly-ops-pull.mjs [--week-of YYYY-MM-DD]
```

**Output:** `output/weekly-ops-briefing-YYYY-MM-DD.md` (session date, CT). **Read this file** before Phase 0b and throughout Phases 1–4 — do not re-query ad-hoc unless mid-session refresh is needed.

The script wraps `weekly-data-pull.mjs` and prepends **planning context** (Monthly Log `Active CL Sprint`, prior Weekly Log activity KPIs for "Last Wk").

### Ops pulls included in briefing

Via `weekly-data-pull.mjs` (parallel MCP where applicable):

1. **Todoist:** Overdue tasks (exclude Shopping List) — currency check
2. **Pipedrive:**
   - Sales pipeline (pipeline 1), CS pipeline (pipeline 6, full pull), activities (overdue + this week)
   - **Completed activity KPIs:** `pipedrive_get_activities` with `done: "1"` and `updated_since` = Monday of review week (RFC3339); filter client-side by `marked_as_done_time` within the 7-day window
   - **Open activities per user:** pull per-user (user_id: 18865844, 22704318, 20938631, 19274648) — omitting `user_id` only returns the authenticated user
3. **Knack Customers** (`object_2`): field_464 (Current Customer), field_1035 (HJD), field_1601 (Health Status), field_1438 (Account Manager), field_1410 (Invoice Follow Up Status), field_1428 (Adjusted Late Invoices), field_1491 (Aged Invoices)
4. **Knack Photographers** (`object_7`): field_33 (Status), field_1267 (Call-ins Last Month), field_1362 (Avg Issues Per Car), field_1446 (Time Off Requests Last Month), field_1338 (Performance Grade)
5. **Hubstaff:** Last week's hours per member via `hubstaff_get_weekly_report`
6. **CS health / photographer performance** sections (script-built flags → RED FLAGS)

**Reconcile completions first** — before Phase 1, note what's already been done since last ops session.

**Advance:** `0` (silent — no user message required, or brief "Data pull complete" only)

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
| Monthly Meeting Log (latest) | Active CL Sprint | A–E or Maintenance |
| Monthly Meeting Log (latest) | Priority Stack | truncated if long |
| Prior Weekly Meeting Log | Aaron / Lexie / Tristen / Ran / Total Activities | for Table 1-A "Last Wk" |
| Briefing | Todoist overdue count | excl Shopping List |
| Briefing | Pipedrive overdue activities | count |

4. State aloud: "Ops briefing only — no remediation this step. Flags carry into Phases 1–4."
5. **Advance:** `0b`

## Phase 1: CL Operations Review (~8 min)

**Purpose:** Pipedrive activity scorecard + CL operating currency. Drives retire-a-slice commitment for commit.

**Data source:** `output/weekly-ops-briefing-YYYY-MM-DD.md` + Phase 0 Pipedrive pulls. **Active CL Sprint** from Table 0b-B (Monthly Meeting Log).

### Part A — Pipedrive Activity Scorecard

Pull completed activities for the **review week** using `pipedrive_get_activities` with `done: "1"` and `updated_since` set to the Monday of the review week. Filter results client-side to activities where `marked_as_done_time` falls within the 7-day window.

**Present exactly Table 1-A:**

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

### Part B — Chrome Lot Currency Check (catch-up forcing)

**Purpose:** Surface, in one glance, how far behind the operating business is, so the meeting orients around *retiring* backlog rather than just re-planning.

**Present exactly Table 1-B:**

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

Then state the **active repair sprint** from `Active CL Sprint` (sequence: **A** Pipedrive+Todoist → **B** 1:1 cadence → **C** customer service → **D** photographer performance → **E** sales → **Maintenance**). Name the specific backlog slice this meeting will **retire** — feeds retire-a-slice rule at commit.

Ask via AskQuestion (single-select): "Which retire-a-slice target for this week?" Options derived from currency table + Active CL Sprint letter.

**Advance:** `1`

## Phase 2: CS Management (~10 min)

**Purpose:** Keep customer relationships healthy with structured 60-day check-in cadence and invoice accountability.

### Part A — Check-in Cadence (`2.A`)

1. **Cross-reference Knack & Pipedrive:** Current customers in Knack (field_464=Yes) vs. open CS deals in Pipedrive (pipeline 6). Flag mismatches.
2. **Staleness check:** Calculate days since last activity per CS deal. Flag any 60+ days overdue.
3. **HJD flag:** Any customer with field_1035 > 10 gets flagged.
4. **Health status review:** Check field_1601 for "Needs Attention" or "At Risk" accounts.
5. **Propose weekly check-in batch:** ~6–9 stops, prioritized by:
   - AT RISK / Needs Attention stage first
   - Days overdue (most stale first)
   - Account value
   - Grouped by geography where possible
6. **Delegation:** Assign stops to Tristen, Lexie, Ran, or Aaron based on deal ownership. Spread across the week.

**Present exactly Table 2-A — CS check-in batch:**

| # | Customer | Stage | Days since activity | HJD | Health | Owner | Assigned to |
|---|----------|-------|---------------------|-----|--------|-------|-------------|
| | | | | | | | |

Propose Pipedrive stop activities — **case-by-case approval** before create. Cap: **max 3 Pipedrive activities per day** across all phases.

### Part B — Late Invoice Review (`2.B`)

1. Pull customers where field_1410 (Invoice Follow Up Status) = "Account Manager Follow Up" or "Max Escalation"
2. Pull customers where field_1428 (Adjusted Late Invoices) > 3 OR field_1491 (Aged Invoices) > 1
3. Combine both lists (deduplicate). Review each flagged customer **one by one**.
4. Create Pipedrive activities (escalation follow-up) assigned to either Aaron or the account manager (field_1438) — approval per activity.

**Invoice escalation trigger rule:**
- field_1410 = "Account Manager Follow Up" or "Max Escalation" → always flag
- field_1428 (Adjusted Late Invoices) > 3 → flag
- field_1491 (Aged Invoices) > 1 → flag
- Any one of these triggers a review and Pipedrive activity

**Present one customer per turn — Table 2-B:**

| Customer | AM | field_1410 | Adj Late | Aged | Action proposed |
|----------|-----|------------|----------|------|-----------------|

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

**Advance:** `2`

## Phase 3: Sales Management (~12 min)

**Purpose:** Drive new revenue and manage the full sales team's pipeline activity.

### Part A — Aaron's Sales Activity (`3.A`)

**A0 — Deal gaps cleanup (mandatory, before planning stops)**

1. Pre-pull Aaron-owned open deals in Sales (pipeline 1), CS (pipeline 6), and Social Media (pipeline 13) with **no `next_activity_date`** via Pipedrive MCP. Present as a numbered checklist.
2. Instruct Aaron to run CL Bot **`deal gaps`** in Teams ([`cl-bot.md`](../../systems/notion-guides/cl-bot.md)) and schedule every gap via the Adaptive Card **Schedule** buttons (+3 days default).
3. **Gate:** Do not plan new sales stops until gaps = 0. If Aaron explicitly defers a deal, log the deal name + reason in commit `Key Decisions` and exclude it from the gap count.

**Present Table 3-A0 — Deal gaps:**

| # | Deal | Pipeline | Action |
|---|------|----------|--------|
| | | | Schedule / Defer |

**A1 — Stale deal review (one-by-one)**

1. Pull Aaron-owned **Sales Pipeline 1** deals with the **Stale** label (same signal as bot `stale deals`). Fallback if label missing: open deals with no activity in 14+ days.
2. Present **one deal at a time** via AskQuestion: **Keep active** / **Move to cold pool (Pipeline 12)** / **Defer decision**.
3. **Cold sales pool** = Pipeline 12 **Not Actively Working** ([`pipedrive.md`](../../systems/pipedrive.md)). Each **Move to Pipeline 12** requires case-by-case Pipedrive approval before `pipedrive_update_deal` — never batch-execute.
4. Deferrals carry to next weekly ops; do not re-present deferred deals unless still stale.

**A2 — Plan the week**

1. Review Pipedrive sales pipeline (pipeline 1): new leads, remaining active stale deals, overdue activities
2. Plan Aaron's sales stops for the week (realistically 2–3)
3. Route planning: use Mapsly manually to cluster stops geographically
4. Review any rescheduled-3× activities — decide: do, delegate, or drop
5. Accountability: activities completed last week vs. planned

**Present Table 3-A2 — Aaron's week plan:**

| Day (optional) | Stop / activity | Deal | Notes |
|----------------|-----------------|------|-------|

### Part B — Team Sales Oversight (`3.B`)

1. Pull sales pipeline deals owned by Tristen, Lexie, and Ran
2. Review each team member's sales activity: deals in progress, overdue activities, stale leads
3. Flag any deals with no activity in 14+ days
4. Propose reassignments or follow-up nudges where needed
5. Create Pipedrive activities for team members as needed (approval per activity)

**Present exactly Table 3-B:**

| Team member | Active deals | Overdue activities | Stale (14+ d) | Proposed action |
|-------------|--------------|-------------------|---------------|-----------------|

**Outputs:** Pipedrive activities for Aaron's sales stops. Pipedrive activities or nudges for team sales. Todoist tasks for follow-ups (case-by-case approval).

**Advance:** `3`

## Phase 4: People Management (~10 min)

**Purpose:** Keep team relationships healthy, catch photographer and staff performance issues early.

### Part A — Photographer Performance Review (`4.A`)

1. Pull all active photographers from Knack (`object_7`, field_33 = "active")
2. Flag any photographer meeting one or more criteria:
   - field_1267 (Call-ins Last Month) > 3
   - field_1362 (Average Issues Per Car) > 0.5
   - field_1446 (Time Off Requests Last Month) > 3
   - field_1338 (Performance Grade) contains "Below Expectations"
   - field_1338 (Performance Grade) is blank/missing
3. For each flagged photographer, pull the most recent comment from `object_57` where field_1006 = "Photographer" (matched via photographer connection)
4. Present each flagged photographer **one by one** with: name, flag reasons, metrics, and most recent photographer comment
5. Decide action: coaching conversation, written warning, schedule meeting, no action, etc.
6. If Performance Grade is missing, set it during the session via `knack_update_record` (with approval)
7. Create Todoist tasks for any decided actions (case-by-case approval)

**Present one photographer per turn — Table 4-A:**

| Name | Flags | Call-ins | Issues/car | Time off | Grade | Latest comment | Action |
|------|-------|----------|------------|----------|-------|----------------|--------|

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

### Part B — Staff & 1:1 Management (`4.B`)

1. Review people directory: who is overdue for a 1:1?
2. Hubstaff hours: pull last week's report. Flag under-hours, low activity %, zero-hour days.
3. Pick top 1–2 overdue 1:1s to schedule this week
4. Any delegation decisions from earlier phases that need to be communicated

**Present exactly Table 4-B:**

| Person | 1:1 overdue (days) | Hubstaff hrs | Activity % | Flags | Action this week |
|--------|-------------------|--------------|------------|-------|------------------|

**Outputs:** Todoist tasks for photographer actions. Calendar events for 1:1 meetings. Knack updates for missing performance grades (with approval). Teams messages if needed.

**Advance:** `4`

## Check: Ops FIELD CHECK (`check`)

**Purpose:** Hard stop before commit. Verify all ops decisions are captured and KPI fields are ready to write.

**Present exactly Table check:**

| Group | Required fields / artifacts | Status |
|-------|----------------------------|--------|
| Phase 1 | Aaron/Lexie/Tristen/Ran/Total Activities computed | |
| Phase 1 | Retire-a-slice target named | |
| Phase 2 | CS check-in batch proposed (or N/A + reason) | |
| Phase 2 | Invoice escalations decided | |
| Phase 3 | Deal gaps = 0 (or deferrals logged) | |
| Phase 3 | Aaron sales week plan | |
| Phase 4 | Flagged photographers reviewed | |
| Phase 4 | 1:1s picked for scheduling | |
| Pending writes | Pipedrive / Knack / Todoist / Calendar list | |

**Do not proceed to `commit` until Table check passes** (or Aaron approves documented N/A per row).

Run: `node scripts/workflow-progress.mjs gate --workflow weekly-ops --phase check`

**Advance:** `check`

## Commit (~5 min)

**Purpose:** Create Weekly Ops Log entry, final capacity check, execute remaining actions, append Team Activity Details, log everything.

**Weekly Ops Meeting Log DB:** `379f40c2-487b-8130-916d-eba9ce85134c` — see `notion-databases.md`.

### Commit procedure

1. **Create Weekly Ops Log entry:** Name = `Week of [planning Monday YYYY-MM-DD]`, Meeting Date = today (CT). Use `personal_notion_create_database_entry` on Weekly Ops Meeting Log DB.
2. **Summary table:** Everything planned across Phases 1–4 (CS stops, sales stops, invoice escalations, photographer actions, 1:1s, retire-a-slice target)

**Table commit-A — Ops summary**

| Area | This week's commitments | Owner |
|------|------------------------|-------|
| Retire-a-slice | from Phase 1.B | |
| CS check-ins | count + assignees | |
| Invoice escalations | count | |
| Aaron sales stops | count | |
| Team sales nudges | | |
| Photographer actions | | |
| 1:1s scheduled | | |

3. **Final capacity check:** Total planned ops hours (stops, calls, reviews) vs. available ops capacity for the week. If total exceeds realistic ops hours minus buffer, something must move. This is non-negotiable.
4. **Store activity KPIs on Weekly Ops Log:** Write `Aaron Activities`, `Lexie Activities`, `Tristen Activities`, `Ran Activities`, `Total Activities` (from Table 1-A).
5. **Store context fields on Weekly Ops Log:**
   - `Active CL Sprint` — snapshot from Monthly Log at session time
   - `Retire-a-Slice` — the named backlog slice from Phase 1.B
   - `Ops Summary` — rich_text narrative from Table commit-A
   - `Key Decisions` — deal deferrals, cold-pool moves, photographer outcomes
   - `Action Items` — bullet list of open follow-ups
6. **Retire-a-slice confirmation (REQUIRED):** Confirm the slice named in Phase 1.B is **scheduled, assigned an owner, or archived** — not merely re-listed. If not done this session, log explicit next action in `Action Items`.
7. **Append per-user Pipedrive detail sections** to the Weekly Ops Log page using `personal_notion_append_blocks`. Use Pipedrive data from Phase 0 (completed activities from review week + all open activities per user). Append:

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

8. **Execute remaining:** Create any Todoist/Calendar/Pipedrive/Knack items not yet committed during Phases 1–4 (each with prior approval).
9. **Verify FIELD CHECK** — re-run Table check; confirm nothing is blank without N/A + reason.

**Advance:** `commit` — workflow complete.

## Cross-Cutting Rules

- **Table contract per phase.** Ledger `current_step` determines which tables are in scope. Phase 1 = scorecard then currency. Phase 2 = CS cadence then invoices one-by-one. Phase 3 = gaps then stale one-by-one then team table. Phase 4 = photographers one-by-one then staff table.
- **FIELD CHECK gate.** `check` must pass before `commit`.
- **Retire-a-slice (catch-up forcing).** Name the slice in Phase 1.B CL Currency Check; confirm at commit it was archived / scheduled / assigned an owner.
- **Route every item into a bucket.** Each surfaced item is Automated (n8n), Delegated (team 1:1s), or a Scheduled slice (calendar + Todoist mirror).
- **Capacity is non-negotiable.** If total planned ops work exceeds available ops hours minus 10–15% buffer, the system pushes back. Something must move.
- **Delegation by default.** For any task deferred 3+ times, suggest delegation before rescheduling. Use the delegation framework in `context/systems/capacity-rules.md`.
- **Max 3 Pipedrive activities per day.** Cap at sustainable levels across CS, sales, and escalations combined.
- **Max 5 must-do Todoist tasks per day** (P1/P2). If morning briefing shows >5, defer.
- **Pipedrive accountability is #1 neglected area.** Always check activities completed vs. planned (Table 1-A).
- **All data pulls happen in Phase 0 silently.** ~40 minutes is for discussion and decisions.
- **Ops briefing only in 0b.** Unlike Weekly Plan Phase 0b, do not attempt data remediation — flag and proceed.
- **Weekly Ops Log is separate from Weekly Meeting Log.** Do not write ops KPIs to the Weekly Meeting Log once Ops Log exists.

## Outputs

- **pre-0:** Planning week confirmed; optional Weekly Plan log gate result.
- **Phase 0:** `output/weekly-ops-briefing-YYYY-MM-DD.md` generated.
- **Phase 0b:** RED FLAGS + planning context presented; no fixes attempted.
- **Phase 1:** Activity scorecard + CL currency + retire-a-slice target named.
- **Phase 2:** CS check-in batch + invoice escalation decisions.
- **Phase 3:** Deal gaps cleared + Aaron sales plan + team oversight actions.
- **Phase 4:** Photographer reviews + 1:1 scheduling decisions.
- **check:** Ops FIELD CHECK pass.
- **commit:** Weekly Ops Log entry + Team Activity Details + approved external writes.

## Failure modes & graceful degradation

- **Weekly Plan log missing for planning week:** Warn in pre-0; Aaron may proceed ops-only or pause for Weekly Plan.
- **weekly-ops-pull / weekly-data-pull script failure:** Fall back to parallel MCP pulls per Phase 0 list; note missing sections in Table 0b-A.
- **Monthly Meeting Log missing `Active CL Sprint`:** Use `Maintenance` default; flag for monthly plan backfill.
- **Prior Weekly Meeting Log missing activity KPIs:** Render "Last Wk" column as `—`; note in 0b footer.
- **Pipedrive MCP rate limits:** Present partial scorecard; retry per-user pulls; document gaps in `Key Decisions`.
- **Knack unavailable:** Skip photographer flags; CS invoice fields from last briefing cache if present; otherwise defer Phase 2.B and 4.A with documented N/A.

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