> **Source:** [`context/skills/quarterly-plan/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/quarterly-plan/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Quarterly Plan — SKILL

## Trigger

This skill activates when Aaron says "quarterly plan", "strategy review", "quarterly review", or "big picture". **Target duration: 90-120 minutes.** Run in the first week of each quarter or when a major life/business shift occurs.

A lightweight **mid-quarter checkpoint** (~15 min, week 6) on "mid-quarter check" is a separate follow-up workflow (not covered by this SKILL).

## Inputs

Load via the router:

- `context/self/values.md` — six categories, Pictures of Success, current Health statuses
- `context/self/current-priorities.md` — what's hot this quarter (will be updated in Part E)
- `context/self/spiritual.md` + `fitness.md` + `social.md` + `parenting.md` + `admin.md` + `dating.md` — operational rules per category, used in Part C personal block
- `context/systems/notion-databases.md` — Quarter Tracker, Quarterly Outcomes, Dev Projects, Quarterly Meeting Log, Monthly + Weekly Meeting Logs, all source DBs
- `context/systems/pipedrive.md` + `knack-fields.md` + `hubstaff.md` + `health-data.md` — data sources for the numeric retrospective
- `context/systems/capacity-rules.md` — Part D capacity check, Part E intervention naming
- `context/systems/cadences.md` — Part E cadence calibration
- `context/people/index.md` — Part B people & relationships
- `context/work/chrome-lot/overview.md` + `team.md` — CL strategic block
- `context/work/turbo-gear/overview.md` + `architecture.md` + `contractors.md` — TG strategic block
- `context/family/custody/index.md` — Part B people & relationships

## Session framing

This is the rarest recurring planning session, so it carries the most weight. The goal is not to tactically pick projects and set KPIs — that's mostly the output. The goal is to produce strategic clarity: what matters, what's changed, what we're building toward over the next 2-3 quarters, and what we're deliberately saying no to.

## Execution Protocol (mandatory)

Read `context/workflow-execution.md`. Init `node scripts/workflow-progress.mjs init --workflow quarterly-plan` at start; one Part (A–E) sub-block per turn; advance ledger before proceeding.

## Procedure

## Project Assessment Rules (apply everywhere in this workflow)

When reviewing or planning projects in any phase:

1. **Never infer status from context or chat history.** Always read the project's actual Status and Completion rollup from the Dev Projects DB. If a project has open sub-items, it is not Done regardless of what prior messages implied.
2. **Use the `Completion` rollup** (percent of sub-items at Status=Done) as the primary indicator of real progress. A project at 80%+ Completion with remaining sub-items is a candidate for either (a) finishing the last mile or (b) scoping the remainder into its own follow-up project.
3. **Ask before guessing scope.** If a project's size, effort, or dependencies aren't clear from the Dev Projects data, ask Aaron before deciding whether to keep, drop, or add it. Offer AskQuestion with specific scope options.
4. **Set a Due Date on every project assigned to the current quarter.** The Dev Projects DB has a `Due Date` date property. In Phase 10, walk through each assigned project and either set the date or mark Off Track if the date can't be committed. Projects tagged to future quarters don't need a due date yet.

## Pre-Flight (silent)

Refresh Withings body-comp into Notion for the quarter:

```
node "scripts/withings-sync.mjs" --days 200 --write
```

That populates the Health Data DB for the outgoing quarter plus the prior quarter (so QoQ math works). Skip silently if it errors -- Notion still has whatever was previously synced.

Then archive recent watch data into the Notion Health Data DB. Health Sync's Drive export is rolling ~30 days, so this captures the most recent month into the long-term archive. Older days from the quarter are already in Notion from previous monthly/weekly persister runs:

```
health_persist_recent({ days: 60 })
```

(The persister is capped at 60 days because Drive doesn't reliably hold more than that. Quarterly history accumulates over time as monthly + weekly plans run their own persisters.)

Then call the **health-data MCP** for a 200-day consolidated pull:

```
health_get_summary({ days: 200 })
```

Use `daily` to slice "this quarter" vs. "prior quarter" for the body-comp + recovery rows in Phase 3. Daily rows are backed by the Notion Health Data DB archive, so historical days are still present even if Drive has rolled past them. If `sources.health_sync.ok` is false, only the most recent ~30 days might be slightly stale; older quarter data still flows from Notion. If the prior-quarter slice has fewer than ~45 days of watch data, annotate "(partial archive: <N> days)" on those rows -- the archive only goes as far back as we've been running the persister.

Pull data in parallel:
1. **Values DB** (`342f40c2-487b-80c5`): All 6 categories with Health statuses and full page content (Pictures of Success)
2. **Quarter Tracker** (`121f40c2-487b-802e`): Find outgoing quarter (Current=Previous or most recent) and current/new quarter. Also check whether future quarter records exist (Q[next+1], Q[next+2], Q[next+3]).
3. **Quarterly Outcomes** (`341f40c2-487b-80c2`): All pages linked to the outgoing quarter (Personal, CL, TG)
4. **Dev Projects** (`341f40c2-487b-80ac`): All projects by Type, across all quarter assignments (current, future, and unassigned backlog). This is the material for per-domain roadmap work.
5. **CL Departments** (`341f40c2-487b-80cc`): All departments with health, priority, and full page content (Pictures of Success)
6. **TG Departments** (`341f40c2-487b-80c9`): Same -- health, priority, and full page content
7. **Quarterly Meeting Log** (`344f40c2-487b-80ed`): Last 2 entries (outgoing quarter AND the one before it) for quarter-over-quarter trend comparison
8. **Weekly Meeting Log** (`322f40c2-487b-81bd`): All entries from the outgoing quarter for habit KPI averages
9. **Monthly Meeting Log** (`344f40c2-487b-806d`): All entries from the outgoing quarter (typically 3) for intra-quarter month-over-month trajectory analysis. Body-comp fields (Weight Avg / Body Fat Avg / Lean Mass Avg + Deltas) feed Phase 3's Personal Habits trajectory.
10. **Hubstaff** (recent 90 days): Time-by-project and by-team-member for Phase 6 time-reality check
11. **Health Data + Watch metrics**: Loaded via the `health-data` MCP `health_get_summary({ days: 200 })` call above. Used in Phase 3 for body-comp + recovery QoQ trend.

Orient: "We're closing Q[X] and planning Q[Y]. This session runs in five parts -- Look Back, Look Around, Look Forward Strategically, Plan Tactically, Commit. Target 90-120 minutes. I'll announce part transitions and flag time overruns."

---

## Part A -- Look Back (Retrospective) -- ~25 min

### Phase 1: Wide Retrospective (narrative, ~10 min)

Before touching numbers, ask via AskQuestion (one at a time or batched):

- What was the biggest positive surprise this quarter?
- What was the biggest negative surprise?
- What did you learn about yourself? About Chrome Lot? About Turbo Gear?
- What's one thing you're proud of?
- What's one thing you'd redo if you could?

These lessons get captured in the Quarterly Meeting Log `Lessons Learned` field at the end. Surface them before the data biases the reflection.

### Phase 2: Values Re-examination (~10 min)

Not a yes/no "have your values shifted?" check. For each of the 6 values in the Values DB:

1. Read the full Picture of Success page content aloud (display it to Aaron).
2. Diagnose current Health in one sentence with evidence from the quarter's Weekly Meeting Log entries.
3. Ask: "Is this Picture still right? Aspirational-not-lived? Drifted? Does anything need editing?"
4. If edits are named, write them via `personal_notion_update_page` immediately. Same for Health status changes.

End of Phase 2, ask: **"Which value is most at risk in Q[next]? Which is strongest and can support the others?"** Capture the answer -- it informs Part C.

### Phase 3: Numeric Retrospective (~5 min)

Run the dual-level trend tables, compressed. Display one table per domain:

```
PERSONAL HABITS -- INTRA-QUARTER TRAJECTORY
| Metric                | Mo 1 | Mo 2 | Mo 3 | Qtr Avg | Last Qtr | Trend |
|-----------------------|------|------|------|---------|----------|-------|
| Strength Sessions     |      |      |      |         |          |       |
| Spirit Minutes        |      |      |      |         |          |       |
| Small Talk Count      |      |      |      |         |          |       |
| Journal Count         |      |      |      |         |          |       |
| Weight Avg (lbs)      |      |      |      |         |          |       |
| Body Fat Avg (%)      |      |      |      |         |          |       |
| Lean Mass Avg (lbs)   |      |      |      |         |          |       |
| Sleep Avg (hours)     |      |      |      |         |          |       |
| HR Avg (bpm)          |      |      |      |         |          |       |
| Resting HR Avg        |      |      |      |         |          |       |
| HRV Avg (ms RMSSD)    |      |      |      |         |          |       |
| Steps Avg/day         |      |      |      |         |          |       |
| Workout Active Min/wk |      |      |      |         |          |       |
```

Body-comp Mo 1/2/3 cells come from each Monthly Meeting Log entry's Weight Avg / Body Fat Avg / Lean Mass Avg. Qtr Avg = mean across the three months (or recompute from the `health_get_summary({ days: 200 })` daily slice for the quarter if a monthly entry is missing the field). Last Qtr = previous Quarterly Meeting Log entry's Weight Avg / Body Fat Avg / Lean Mass Avg. Trend = QoQ delta with direction; for body comp, also annotate alignment with Aaron's stated cut/bulk/recomp intent for the outgoing quarter.

Sleep / HR / RHR / HRV / Steps / Workout Active Min cells follow the same pattern: prefer each Monthly Meeting Log entry's `Sleep Avg` / `Heart Rate Avg` / `Resting HR Avg` / `HRV Avg` / `Steps Avg` / `Workout Active Minutes` for Mo 1/2/3 cells. Fall back to recomputing from `daily` slice for any month missing those fields. Qtr Avg = mean across the quarter's daily rows. Last Qtr = previous Quarterly Meeting Log entry's matching field, OR recompute from the prior-quarter daily slice. Trend = QoQ delta. Render `--` for any row whose underlying data is null (Resting HR + HRV in particular until those Health Sync folders are enabled). If the prior-quarter slice covers fewer than ~45 days, annotate "(partial archive: <N> days)" beside that row.

```
CHROME LOT -- INTRA-QUARTER TRAJECTORY
| Metric              | Mo 1 | Mo 2 | Mo 3 | Qtr Avg | Last Qtr | Trend |
|---------------------|------|------|------|---------|----------|-------|
| CL Revenue          |      |      |      |         |          |       |
| CL Customer Count   |      |      |      |         |          |       |
| CL Churn            |      |      |      |         |          |       |
| Ops Minutes         |      |      |      |         |          |       |
| Field Work Minutes  |      |      |      |         |          |       |
```

```
TURBO GEAR -- INTRA-QUARTER TRAJECTORY
| Metric              | Mo 1 | Mo 2 | Mo 3 | Qtr Avg | Last Qtr | Trend |
|---------------------|------|------|------|---------|----------|-------|
| TG Features Shipped |      |      |      |         |          |       |
| TG Demos Given      |      |      |      |         |          |       |
| Deep Work Minutes   |      |      |      |         |          |       |
```

"Last Qtr" comes from the previous Quarterly Meeting Log entry. Flag any metric with 3 consecutive months of decline, >20% quarter-over-quarter delta, or below 80% of target.

Pick the **2-3 metrics that matter most for Q[next]** and name them explicitly. Everything else is noise this round.

### Phase 3b: Monthly Health Trend Review (~8 min)

Pull the **3 Monthly Meeting Log entries** from the outgoing quarter (chronological). Build domain-health and life-context tables:

```
WORK DOMAIN HEALTH -- INTRA-QUARTER
| Domain      | Mo1 | Mo2 | Mo3 | Streak        |
|-------------|-----|-----|-----|---------------|
| Chrome Lot  | H/U |     |     | N mos Unhealthy |
| Turbo Gear  | H/U |     |     |               |
```

Data source: `Chrome Lot Health` and `Turbo Gear Health` select properties on each Monthly Meeting Log entry. For months before those properties existed, annotate `(no monthly snapshot)` and **do not false-trigger** the strategy gate.

Also show life-category context (not a separate gate — monthly Phase 1d handles life):

```
LIFE HEALTH -- UNHEALTHY COUNT PER MONTH
| Month | Spirituality | Fitness | Work | Social | Admin | Parenting | Total Unhealthy |
|-------|--------------|---------|------|--------|-------|-----------|-----------------|
| Mo1   | H/U          |         |      |        |       |           | N/6             |
```

**Streak computation:** For each work domain, count consecutive months rated Unhealthy ending at Mo3. A streak of **3** (all months in the quarter) triggers the **Sustained Unhealthy Domain Gate** before Part C.

**End of Part A:** state elapsed time. Target was 33 min (was 25 min; +8 min for Phase 3b).

---

## Part B -- Look Around (Environment scan) -- ~15 min

### Phase 4: People & Relationships (~7 min)

Pull `context/people/index.md` and any overdue 1:1s from the Weekly Meeting Log entries.

- **Team**: Who's growing? Who's blocked? Who's underutilized? Who needs a harder conversation this quarter?
- **Personal relationships**: Kids, girlfriend/dating, social circle, mentors. Status of each. Anything needing reinvestment?
- **1:1 cadence**: Any that lapsed? What's the plan to rebuild?

Name 1-2 specific relationship actions that have to happen in Q[next].

### Phase 5: External Environment (~5 min)

What changed that affects the plan?

- Market / competitors / industry
- Knack product or pricing changes
- Economic context
- Family changes (kids' school, custody status, health)
- Legal proceedings
- Personal health changes

Capture as a one-paragraph narrative in the Quarterly Meeting Log entry at Phase 14.

### Phase 6: Capacity & Time Reality (~3 min)

Pull last quarter's Hubstaff totals and calendar analysis. Present:

- Where did hours actually go (by project / by domain / by activity type)?
- Where did you *intend* them to go?
- Where's the gap?

This surfaces the "I wanted Q[X] to be about TG but 60% of my hours went to CL field work" pattern before Part C tries to make allocation decisions.

**End of Part B:** state elapsed time. Target was 15 min.

---

## Part C -- Look Forward Strategically (per domain) -- ~35 min

**Pre-step (shared, ~2 min):** ensure Quarter Tracker (`121f40c2-487b-802e`) has records for Q[next+1], Q[next+2], Q[next+3]. Create any that are missing via `personal_notion_create_database_entry` so future-quarter project assignments in 7C/8C/9C have somewhere to land.

### Sustained Unhealthy Domain Gate (conditional, before domain blocks)

**Triggers when Chrome Lot or Turbo Gear was Unhealthy for all 3 months of the outgoing quarter** (from Phase 3b streak table).

When triggered for a domain:

1. **Pause** that domain's Part C block (Phase 8 for CL, Phase 9 for TG) until the gate completes.
2. Mandatory strategy rethink via `AskQuestion`: What approach failed? What gets cut, delegated, or restructured?
3. Name **1–3 structural changes** (not KPI tweaks) before committing Q[next] theme and roadmap for that domain.
4. Log outcomes in Quarterly Meeting Log `Lessons Learned` (Phase 14) and append a **Health Intervention Notes** block to the quarterly page body via `personal_notion_append_blocks`.
5. An Unhealthy domain **cannot receive "more projects"** in Part D until rethink is documented — cut or restructure first.

If no 3-month unhealthy streaks: proceed directly to domain blocks.

Each domain block follows the same shape: context recap → name a theme → walk the roadmap → name the no-list.

### Phase 7: Personal Strategic Forward-Look (~10 min)

**7A: Values recap (2 min).** Reference Phase 2. State which value is most at risk in Q[next] and which is strongest. Orient the Personal domain around these.

**7B: Personal Theme (2 min).** Via AskQuestion, name the Personal theme for Q[next] in one sentence. Examples:
- "Close out the legal/admin backlog so dev capacity is free in Q4."
- "Rebuild Spirituality after it got starved last quarter."
- "Protect family time through the custody modification."

This sentence gets written as a callout at the top of the Personal Quarterly Outcomes page in Phase 11.

**7C: Personal Roadmap (6 min).** Walk the Dev Projects roadmap view with Type=Personal -- all statuses, all quarter assignments. For each live project, decide:

- Assign to Q[next] (active this quarter)
- Assign to Q[next+1] / Q[next+2] / Q[next+3] (on the roadmap, not now)
- Clear quarter, Status=Idea (someday/maybe -- still possible, not scheduled)
- Archive (dead -- won't happen, stop carrying it)

Use the Project Assessment Rules at the top of this file. Every live project ends this phase with either a `🍁 Quarter` relation or an explicit decision to kill it.

**Personal No-List:** name 2-3 Personal things deliberately not happening this year. Examples: "Not starting any new hobby projects until custody is resolved." "Not doing any home renovations in 2026." These get written as a "Deliberate No-List" section on the Personal Quarterly Outcomes page in Phase 11.

### Phase 8: Chrome Lot Strategic Forward-Look (~12 min)

**8A: CL Department Deep Read (4 min).** Pull CL Departments (`341f40c2-487b-80cc`) and display each department's full Picture of Success page content. For each:

- Current Health (Healthy / Unhealthy / NA)
- Current Priority
- Is the Picture still right? Has the reality of the department moved past what's written?

Edit department pages via `personal_notion_update_page` if Pictures need to change. Update Health / Priority if they've moved.

**8B: CL Theme (2 min).** Via AskQuestion, name the CL theme for Q[next] in one sentence. Examples:
- "Rebuild team Pipedrive cadence and resolve the two Unhealthy departments."
- "Instrument CL revenue and churn so next quarter has real numbers."
- "Get Aaron out of field work."

Written as a callout at the top of the CL Quarterly Outcomes page in Phase 11.

**8C: CL Roadmap (6 min).** Walk the Dev Projects roadmap view with Type=Chrome Lot -- all statuses, all quarter assignments. Same decision tree as 7C: Q[next], a future quarter, someday/Idea, or archive. Every live project ends with a clear home.

**CL No-List:** name 2-3 CL things deliberately not happening this year. Examples: "No new service line expansion this year." "Not hiring another senior until the ones we have are back on track." Written on the CL Quarterly Outcomes page in Phase 11.

### Phase 9: Turbo Gear Strategic Forward-Look (~12 min)

**9A: TG Department Deep Read (4 min).** Pull TG Departments (`341f40c2-487b-80c9`). Same as 8A -- display Pictures of Success, diagnose Health and Priority, edit if reality has moved.

**9B: TG Theme (2 min).** Via AskQuestion, name the TG theme for Q[next] in one sentence. Examples:
- "Land the first three external demos."
- "Achieve Knack feature parity and stop shipping net-new internal features."
- "Build the external sales motion from zero."

Written as a callout at the top of the TG Quarterly Outcomes page in Phase 11.

**9C: TG Roadmap (6 min).** Walk the Dev Projects roadmap view with Type=Turbo Gear. Same decision tree as 7C/8C.

**TG No-List:** name 2-3 TG things deliberately not happening this year. Examples: "No new integrations until Knack parity is done." "Not building billing until first external customer commits." Written on the TG Quarterly Outcomes page in Phase 11.

**Roadmap tagging rules (apply in 7C, 8C, 9C):**
- Every live project ends with either a `🍁 Quarter` relation or an explicit kill decision.
- Projects moved to a future quarter are real commitments, not wishes -- only assign if you'd take the same project this quarter if capacity allowed.
- The no-lists are not aspirational -- name things you can feel yourself being tempted to do but shouldn't.

**End of Part C:** state elapsed time. Target was 35 min.

---

## Part D -- Plan the Quarter Tactically -- ~15 min

### Phase 10: Commit Projects to This Quarter (~10 min)

Candidates are the projects Phases 7C/8C/9C tagged to Q[next]. For each:

- Apply Project Assessment Rules (never infer completion, check `Completion` rollup, ask about scope).
- Present the full list across all three domains with count per domain.
- **Cross-domain capacity check.** If the combined total exceeds ~10 projects, projects move to Q[next+1]. Push the lowest-leverage ones first; keep the ones aligned with each domain's theme.
- Final pick: 2-4 per domain, 6-10 total.
- Set `Due Date` on every selected project via `personal_notion_update_page`.
- Carry-overs from the outgoing quarter that didn't complete: honest call -- keep, rescope, or drop. If kept without a clear due date, mark Off Track.

### Phase 11: Per-Domain KPI Targets + Instrumentation Gate (~5 min)

For each of the three Quarterly Outcomes pages (Personal, CL, TG):

1. Ensure the page exists for Q[next]; create or update via `personal_notion_update_page` / `personal_notion_create_database_entry`.
2. Write the per-domain theme from 7B/8B/9B as a callout at the top of the page.
3. Write the domain's Deliberate No-List as a section on the page (format: heading + bullet list).
4. Write the project list with Due Dates.
5. Set KPI targets relevant to the domain:
   - **Personal**: project completion rate, habit consistency goals, social interaction frequency
   - **CL**: revenue, customer count, churn, invoice aging, CS visit frequency, team Pipedrive activity
   - **TG**: features shipped, demos given, external customer conversations, deep work minutes

**Instrumentation gate.** For every KPI, confirm the data source exists. If instrumentation is missing (e.g., "CL Revenue" has no data source today), either (a) add a project to instrument it this quarter, or (b) drop the KPI. **No phantom targets.**

**End of Part D:** state elapsed time. Target was 15 min.

---

## Part E -- System & Commit -- ~15 min

### Phase 12: System & Cadence Calibration (~5 min)

Review the meta-system:
- Are daily morning/evening reviews happening consistently?
- Are micro-scrubs happening 3x/week?
- Is the weekly meeting happening? (Count Weekly Meeting Log entries in the outgoing quarter vs. expected ~13)
- Are monthly plans happening?
- Are n8n workflows running reliably?
- Is the people directory up to date?
- Any cadences that have lapsed?
- Are the Weekly Meeting Log KPI fields being populated consistently?

**New rule:** if a cadence collapsed last quarter, name the specific change this quarter that's supposed to fix it. Don't just note "weekly meetings fell off" -- name the intervention. If there's no intervention, it'll happen again.

Adjust system parameters if needed (capacity limits, cadence frequencies, rotation). If context files change, update them in Phase 14.

**Operations catalog refresh:** run `node scripts/generate-ops-catalog.mjs` then `node scripts/publish-ops-catalog-to-notion.mjs`. Review the **Health / staleness** section in Notion ([Systems Hub](https://app.notion.com/p/Life-Command-Center-Systems-Hub-377f40c2487b8142b5fdfd6707572d29)) or `output/ops-catalog.md` — flag any stale context files or broken cross-references for update in Phase 14.

### Phase 13: Calendar Stake-in-the-Ground (~5 min)

Block in Google Calendar (Aaron's calendar) for the full quarter:

- Weekly meeting slot (recurring, ~60 min)
- 3 monthly plan dates (~70 min each, first week of each month)
- 1 mid-quarter checkpoint at week 6 (~15 min -- lightweight "are we on track with the themes?")
- The Q[next+1] quarterly plan date, ~90 days out

These are the dates that get eaten first when things get busy. Block them now so they have to be explicitly moved.

### Phase 13b: Planning Context Capture (~3 min)

**Purpose:** Commit quarter-level priority stack and parked domains so monthly + weekly plans inherit reliable context.

**Required every quarterly plan.** Store in session state (`quarterly_priority_stack`, `quarterly_domains_parked`) for Phase 14.

1. **Draft Priority Stack** (max 5 numbered lines) from Part C themes + Part D project commits. Format: `N. [Domain] — [one-line focus]`.
2. **AskQuestion (multi-select):** "Which domains are **parked** for Q[next]?" Options: Turbo Gear | Chrome Lot New Business | External TG Demos | Personal Projects | Dating Outreach | Custody (hold) | Other | None — all domains active. Derive from no-lists and explicit pause decisions in Part C.
3. Confirm with Aaron before Phase 14.

### Phase 14: Commit & Log (~5 min)

1. Verify: all project assignments saved (quarter relations set), Due Dates on current-quarter projects, KPI targets on all three Outcomes pages, themes as callouts at top, no-lists as sections, department health statuses current, future Quarter Tracker records exist. **Also verify** Phase 13b planning context captured.
2. **Log to Quarterly Meeting Log** (`344f40c2-487b-80ed`) via `personal_notion_create_database_entry`. New entry linked to the outgoing quarter with:
   - Meeting Date, Quarter relation
   - Wellness averages (PHQ-2, GAD-2, Energy -- averaged from Weekly Meeting Log entries)
   - Habit KPI averages (Strength, Cardio, Small Talk, Spirit, Deep Work, Ops, Field Work, Journal)
   - **Body composition** (from Phase 3): Weight Avg, Body Fat Avg, Lean Mass Avg (means across the outgoing quarter from Monthly Meeting Log entries or the `health_get_summary` daily slice) plus Weight Delta, Body Fat Delta, Lean Mass Delta (this-quarter avg minus last-quarter avg, signed). Omit any field where the underlying data is null.
   - **Watch metrics** (from Phase 3): Sleep Avg, Heart Rate Avg, Resting HR Avg, HRV Avg, Steps Avg (quarter means) plus Sleep Delta, Heart Rate Delta, Resting HR Delta, HRV Delta, Steps Delta (this-quarter avg minus last-quarter avg, signed). Also Workout Active Minutes (quarter total in minutes). Omit any field where the underlying data is null. Resting HR + HRV will stay null until those Health Sync folders are enabled.
   - Project counts (Assigned, Completed, Carried Over, broken out by Personal/CL/TG)
   - Business metrics (CL Revenue, CL Customer Count, CL Churn, TG Demos Given, TG Features Shipped)
   - Starved Values, Key Wins, Key Misses
   - **Planning context (REQUIRED — monthly + weekly plans read these):**
     - `Priority Stack` (rich_text) — from Phase 13b (`quarterly_priority_stack`)
     - `Domains Parked` (multi_select) — from Phase 13b (`quarterly_domains_parked`)
   - **Lessons Learned** -- from Phase 1 narrative retro
   - **Next Quarter Focus** -- the three per-domain themes from 7B/8B/9B, one paragraph each
3. **Append per-user Pipedrive detail sections** to the Quarterly Meeting Log page using `personal_notion_append_blocks`. Pull completed activities from the past ~90 days (use `pipedrive_get_activities` with `done: "1"` and `updated_since` set to the 1st day of the outgoing quarter, then filter by `marked_as_done_time` within the quarter). Also pull all open activities per user. Append the following structure:

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

4. Update `context/self/values.md` if values or health statuses shifted (Health Status field stays in the Values DB; the markdown is the picture-of-success layer)
5. Update `context/self/current-priorities.md` to reflect the new quarter's focus (this is the rotating "what's hot now" file)
6. Update `context/systems/capacity-rules.md` if limits were adjusted
7. Update `context/people/index.md` if the team changed

**End of Part E:** state total elapsed time. Target was 90-120 min end-to-end.

---

## Key Rules

- This is the ONLY workflow where quarterly project assignments happen. Don't assign projects to quarters in daily or weekly reviews.
- This is the ONLY workflow that updates Values Pictures of Success and Department Pictures of Success. Monthly and weekly workflows can note drift but don't edit the pictures themselves.
- Each domain in Part C follows the same rhythm: **context recap → theme → roadmap → no-list.** Don't shortcut any step.
- The per-domain themes and no-lists are the strategic output. Projects and KPIs are downstream of them. If a project doesn't serve the theme, ask why it's on the list.
- Department success pictures function like values -- they define what "good" looks like for that department. Use them to evaluate whether projects are aligned.
- Focus on alignment with values and success targets, not just productivity.
- The goal is fewer, better-aligned projects -- not more. Reference "Do less, better."
- The quarterly docket drives everything downstream: weekly project selection pulls from it, monthly KPI updates measure against it.
- Carry-over projects from the previous quarter need honest assessment: keep, rescope, or drop. Default to drop if you can't name why it survived the quarter it was supposed to happen in.
- If the session is running short (e.g., 30 min at Part C start), don't compress Part C -- compress Part B. Parts A and C are the strategic core; everything else supports them.

## Outputs

- **Pre-Flight:** Withings refreshed; watch data persisted where possible; MCP `health_get_summary({ days: 200 })`; parallel data pulls oriented to Q transition.
- **Part A Phase 1–3:** Narrative Lessons Learned captured for Phase 14; Picture of Success updates applied in Notion; numeric retrospective tables; 2–3 named priority metrics for Q[next].
- **Part B Phase 4–6:** Relationship action commitments; environment narrative captured to Quarterly Meeting Log in Phase 14; Hubstaff/calendar gap surfaced.
- **Part C Phase 7–9:** Quarterly Outcomes scaffolding (themes, no-lists prepared for Phase 11); Dev Projects quarter assignments cleared for every live row; Quarter Tracker futures created where missing.
- **Part D Phase 10–11:** Current-quarter commits (6–10 projects), Due Dates, KPI targets with instrumentation gate; Quarterly Outcomes pages updated.
- **Part E Phase 13b:** Priority Stack + Domains Parked captured (session state).
- **Part E Phase 12–14:** Calendar infrastructure for quarter/cadences; Quarterly Meeting Log entry with full rollup + planning context fields + Team Activity Details; context updates (`values.md`, `current-priorities.md`, `capacity-rules.md`, `people/index.md`).

## Failure modes & graceful degradation

- **Withings / `health_get_summary` errors:** Skip silently for Withings preload; QoQ Phase 3 may require monthly log fallbacks only.
- **`sources.health_sync.ok` false:** Recent days may be slightly stale; older quarter flows from Notion; annotate archive partiality `(partial archive: <N> days)` where applicable (Pre-Flight and Phase 3).
- **`health_persist_recent` capped at 60 days:** Quarterly history cumulative via monthly + weekly cadence (already documented Pre-Flight).
- **Instrumentation gate:** KPIs without measurable sources drop or spawn instrumenting projects — no phantom targets (Phase 11).
- **Time pressure:** If short on time mid-session, defer Part B detail before shortening Part A or Part C (`Key Rules`).

## See also

- `../../router.md`
- `../weekly-planning/SKILL.md`
- `../monthly-plan/SKILL.md`
- `../../systems/notion-databases.md`, `../../systems/pipedrive.md`, `../../systems/knack-fields.md`, `../../systems/hubstaff.md`, `../../systems/health-data.md`
- `../../systems/capacity-rules.md`, `../../systems/cadences.md`
- `../../self/values.md`, `../../self/current-priorities.md`, `../../self/spiritual.md`, `../../self/fitness.md`, `../../self/social.md`, `../../self/parenting.md`, `../../self/admin.md`, `../../self/dating.md`
- `../../people/index.md`
- `../../family/custody/index.md`
- `../../work/chrome-lot/overview.md`, `../../work/chrome-lot/team.md`
- `../../work/turbo-gear/overview.md`, `../../work/turbo-gear/architecture.md`, `../../work/turbo-gear/contractors.md`