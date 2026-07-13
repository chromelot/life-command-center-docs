> **Source:** [`context/skills/quarterly-plan/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/quarterly-plan/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Quarterly Plan — SKILL

## Table of contents

- [Trigger](#trigger)
- [Inputs](#inputs)
- [Session framing](#session-framing)
- [Execution Protocol (mandatory)](#execution-protocol-mandatory)
  - [Ledger step order](#ledger-step-order)
  - [Present exactly — Part A](#present-exactly-part-a)
  - [Present exactly — Parts C–E](#present-exactly-parts-ce)
  - [Present exactly — Part B (narrative → tables)](#present-exactly-part-b-narrative-tables)
- [Procedure](#procedure)
- [Project Assessment Rules (apply everywhere in this workflow)](#project-assessment-rules-apply-everywhere-in-this-workflow)
  - [Goal trajectory review (Part A look-back + before committing in Part D)](#goal-trajectory-review-part-a-look-back-before-committing-in-part-d)
  - [Behind → structured shift](#behind-structured-shift)
- [Pre-Flight (silent)](#pre-flight-silent)
- [Part A -- Look Back (Retrospective) -- ~25 min](#part-a-look-back-retrospective-25-min)
  - [Phase 1: Wide Retrospective (narrative, ~10 min)](#phase-1-wide-retrospective-narrative-10-min)
  - [Phase 2: Values Re-examination (~10 min)](#phase-2-values-re-examination-10-min)
  - [Phase 3: Numeric Retrospective (~5 min)](#phase-3-numeric-retrospective-5-min)
  - [Phase 3b: Monthly Health Trend Review (~8 min)](#phase-3b-monthly-health-trend-review-8-min)
- [Part B -- Look Around (Environment scan) -- ~15 min](#part-b-look-around-environment-scan-15-min)
  - [Phase 4: People & Relationships (~7 min)](#phase-4-people-and-relationships-7-min)
  - [Phase 5: External Environment (~5 min)](#phase-5-external-environment-5-min)
  - [Phase 6: Capacity & Time Reality (~3 min)](#phase-6-capacity-and-time-reality-3-min)
- [Part C -- Look Forward Strategically (per domain) -- ~35 min](#part-c-look-forward-strategically-per-domain-35-min)
  - [Sustained Unhealthy Domain Gate (conditional, before domain blocks)](#sustained-unhealthy-domain-gate-conditional-before-domain-blocks)
  - [Phase 7: Personal Strategic Forward-Look (~10 min)](#phase-7-personal-strategic-forward-look-10-min)
  - [Phase 8: Chrome Lot Strategic Forward-Look (~12 min)](#phase-8-chrome-lot-strategic-forward-look-12-min)
  - [Phase 9: Turbo Gear Strategic Forward-Look (~12 min)](#phase-9-turbo-gear-strategic-forward-look-12-min)
- [Part D -- Plan the Quarter Tactically -- ~15 min](#part-d-plan-the-quarter-tactically-15-min)
  - [Phase 10: Commit Projects to This Quarter (~10 min)](#phase-10-commit-projects-to-this-quarter-10-min)
  - [Phase 11: Per-Domain KPI Targets + Instrumentation Gate (~5 min)](#phase-11-per-domain-kpi-targets-instrumentation-gate-5-min)
- [Part E -- System & Commit -- ~15 min](#part-e-system-and-commit-15-min)
  - [Phase 12: System & Cadence Calibration (~5 min)](#phase-12-system-and-cadence-calibration-5-min)
  - [Phase 13: Calendar Stake-in-the-Ground (~5 min)](#phase-13-calendar-stake-in-the-ground-5-min)
  - [Phase 13b: Planning Context Capture (~3 min)](#phase-13b-planning-context-capture-3-min)
  - [Phase 14: Commit & Log (~5 min)](#phase-14-commit-and-log-5-min)
- [Key Rules](#key-rules)
- [Outputs](#outputs)
- [Failure modes & graceful degradation](#failure-modes-and-graceful-degradation)
- [See also](#see-also)

---


<a id="trigger"></a>
## Trigger

This skill activates when Aaron says "quarterly plan", "strategy review", "quarterly review", or "big picture". **Target duration: 90-120 minutes.** Run in the first week of each quarter or when a major life/business shift occurs.

A lightweight **mid-quarter checkpoint** (~15 min, week 6) on "mid-quarter check" is a separate follow-up workflow (not covered by this SKILL).

<a id="inputs"></a>
## Inputs

Load via the router:

- `context/self/values.md` — six categories, Pictures of Success, current Health statuses
- `context/self/current-priorities.md` — what's hot this quarter (will be updated in Part E)
- `context/self/spiritual.md` + `fitness.md` + `social.md` + `parenting.md` + `admin.md` + `dating.md` — operational rules per category, used in Part C personal block
- `context/systems/notion-databases.md` — Quarter Tracker (plan content lives on the page + `Plan Doc URL` PDF; Quarterly Outcomes DB retired 2026-07-11), Tasks, Projects, Quarterly Meeting Log, Monthly + Weekly Meeting Logs, all source DBs
- `context/systems/pipedrive.md` + `knack-fields.md` + `hubstaff.md` + `health-data.md` — data sources for the numeric retrospective
- `context/systems/capacity-rules.md` — Part D capacity check, Part E intervention naming
- `context/systems/cadences.md` — Part E cadence calibration
- `context/people/index.md` — Part B people & relationships
- `context/work/chrome-lot/overview.md` + `team.md` — CL strategic block
- `context/work/turbo-gear/overview.md` + `architecture.md` + `contractors.md` — TG strategic block
- `context/family/custody/index.md` — Part B people & relationships

<a id="session-framing"></a>
## Session framing

This is the rarest recurring planning session, so it carries the most weight. The goal is not to tactically pick projects and set KPIs — that's mostly the output. The goal is to produce strategic clarity: what matters, what's changed, what we're building toward over the next 2-3 quarters, and what we're deliberately saying no to.

<a id="execution-protocol-mandatory"></a>
## Execution Protocol (mandatory)

Read `context/workflow-execution.md`, `context/systems/workflow-output-contracts.md`, and `context/systems/workflow-logs.md`.

1. `node scripts/workflow-progress.mjs init --workflow quarterly-plan`
2. Step `0` — silent Pre-Flight pulls
3. Step `A.0` — `node scripts/workflow-notion-log.mjs create --ledger <path>`
4. Every turn: `status` → one ledger step → `advance` → `workflow-notion-log sync`
5. Run `gate --phase B|C|D|E` before crossing part boundaries
6. Step `E.4` — `workflow-notion-log complete`

<a id="ledger-step-order"></a>
### Ledger step order

| Step | Skill section |
|------|---------------|
| `0` | Pre-Flight (silent) |
| `A.0` | Create Quarterly Meeting Log shell |
| `A.1` | Part A Phase 1 — Wide retrospective (Table A.1-A, one question per turn) |
| `A.2` | Part A Phase 2 — Values (one value per turn) |
| `A.3` | Part A Phase 3 — Numeric tables (Personal, CL, TG — one table per turn) |
| `A.3b` | Part A Phase 3b — Work domain health trends |
| `A.check` | Part A FIELD CHECK |
| `B.1` | Phase 4 People & relationships — Table B.1-A |
| `B.2` | Phase 5 External environment — Table B.2-A |
| `B.3` | Phase 6 Capacity — Table B.3-A |
| `C.1` | Phase 7 Personal (7A→7B→7C in-step, one sub-table per turn) |
| `C.2` | Phase 8 Chrome Lot (8A→8B→8C) |
| `C.3` | Phase 9 Turbo Gear (9A→9B→9C) |
| `C.gate` | Sustained unhealthy domain gate (if triggered) |
| `D.1` | Phase 10 Project commits — Table D.1-A |
| `D.2` | Phase 11 KPI + instrumentation — Table D.2-A |
| `D.check` | Part D FIELD CHECK |
| `E.1` | Phase 12 Cadence calibration — Table E.1-A |
| `E.2` | Phase 13 Calendar blocks — Table E.2-A |
| `E.3` | Phase 13b Planning context — Table E.3-A |
| `E.4` | Phase 14 Commit & log — Table E.check |

<a id="present-exactly-part-a"></a>
### Present exactly — Part A

| Step | Present exactly |
|------|-----------------|
| `A.1` | Table A.1-A — one retrospective question per turn |
| `A.2` | Table A.2-A — one Values category per turn |
| `A.3` | Table A.3-A Personal, then A.3-B CL, then A.3-C TG — one per turn |
| `A.3b` | Table A.3b-A (work domain health streaks), Table A.3b-B (life unhealthy count) |
| `A.check` | Table A.check |

**Table A.1-A — Wide retrospective** *(step `A.1`)*

| # | Question | Answer |
|---|----------|--------|

**Table A.2-A — Values** *(step `A.2`, one category per turn)*

| Value | Health | Evidence | Picture still right? | Edits |
|-------|--------|----------|---------------------|-------|

**Table A.3-A / A.3-B / A.3-C** — Use Personal / CL / TG intra-quarter trajectory tables from Part A Phase 3 procedure.

**Table A.3b-A — Work domain health streaks** *(step `A.3b`)*

| Domain | Mo1 | Mo2 | Mo3 | Streak |
|--------|-----|-----|-----|--------|

<a id="present-exactly-parts-ce"></a>
### Present exactly — Parts C–E

| Step | Present exactly |
|------|-----------------|
| `C.1` | Part C Phase 7 sub-tables (7A→7B→7C) — one per turn |
| `C.2` | Part C Phase 8 sub-tables (8A→8B→8C) — one per turn |
| `C.3` | Part C Phase 9 sub-tables (9A→9B→9C) — one per turn |
| `C.gate` | Table C.gate (sustained unhealthy) or N/A |
| `D.1` | Table D.1-A — one project commit per turn |
| `D.2` | Table D.2-A (KPI targets + instrumentation gate) |
| `D.check` | Table D.check |
| `E.1` | Table E.1-A (cadence calibration) |
| `E.2` | Table E.2-A (calendar blocks) |
| `E.3` | Table E.3-A (planning context) |

**Table D.1-A — Project commits** *(step `D.1`)*

| Project | Quarter | Due Date | Capacity fit | Decision |
|---------|---------|----------|--------------|----------|

**Table D.2-A — KPI + instrumentation** *(step `D.2`)*

| Outcome | KPI | Target | Instrumented? | Gap |
|---------|-----|--------|---------------|-----|

<a id="present-exactly-part-b-narrative-tables"></a>
### Present exactly — Part B (narrative → tables)

**Table B.1-A — People & relationships** *(step `B.1`)*

| Area | Status | Action for Q[next] |
|------|--------|-------------------|
| Team — growing | | |
| Team — blocked | | |
| Personal relationships | | |
| 1:1 cadence gaps | | |

**Table B.2-A — External environment** *(step `B.2`)*

| Factor | Change this quarter | Impact |
|--------|---------------------|--------|
| Market / competitors | | |
| Knack / CL ops | | |
| Family / legal / health | | |

**Table B.3-A — Capacity reality** *(step `B.3`)*

| Domain | Hubstaff hrs | Intended hrs | Gap |
|--------|--------------|--------------|-----|
| Chrome Lot | | | |
| Turbo Gear | | | |
| Personal | | | |

**FIELD CHECK — Part A** *(step `A.check`)*

| Item | Pass |
|------|------|
| Narrative retro captured | |
| All 6 values reviewed | |
| Numeric trajectory tables presented | |
| Phase 3b domain health streaks computed | |

**FIELD CHECK — Commit** *(step `E.4`)*

| Item | Pass |
|------|------|
| Quarterly Meeting Log entry complete | |
| Quarter page has per-domain themes + no-lists + KPIs + PDF (`Plan Doc URL`) | |
| Priority Stack + Domains Parked from 13b | |
| Team Activity Details appended | |
| Context file updates (if any) done | |
| Session Complete = Complete | |

<a id="procedure"></a>
## Procedure

<a id="project-assessment-rules-apply-everywhere-in-this-workflow"></a>
## Project Assessment Rules (apply everywhere in this workflow)

When reviewing or planning projects in any phase:

1. **Never infer status from context or chat history.** Always read the project's actual Status and Completion rollup from the Tasks DB. If a project has open sub-items, it is not Done regardless of what prior messages implied.
2. **Use the `Completion` rollup** (percent of sub-items at Status=Done) as the primary indicator of real progress. A project at 80%+ Completion with remaining sub-items is a candidate for either (a) finishing the last mile or (b) scoping the remainder into its own follow-up project.
3. **Ask before guessing scope.** If a project's size, effort, or dependencies aren't clear from the Tasks data, ask Aaron before deciding whether to keep, drop, or add it. Offer AskQuestion with specific scope options.
4. **Set a Due Date on every project assigned to the current quarter.** The Tasks DB has a `Due Date` date property. In Phase 10, walk through each assigned project and either set the date or mark Off Track if the date can't be committed. Projects tagged to future quarters don't need a due date yet.
5. **Set the `Date` (start+end) on every committed project** — this populates the [Roadmap — Timeline](https://notion.so/39bf40c2487b8188bc05d953ac5d8802) and forces sequencing. Link each committed project to its **`🥅 Goals`** (or mark it conscious no-goal maintenance).

<a id="goal-trajectory-review-part-a-look-back-before-committing-in-part-d"></a>
### Goal trajectory review (Part A look-back + before committing in Part D)

Roadmap planning is **goal-driven**. Using the [Goals — Milestones](https://notion.so/39bf40c2487b810d905fdb9585ceddf9) + [Projects by Goal](https://notion.so/39bf40c2487b8120be53c40f6730fb45) views (and the Goals `Progress` + `Behind Projects` rollups), walk each active Goal:

- **On track?** Signal = `Behind Projects` > 0, or `Target Date` approaching with low `Progress`.
- **Targets too, not just projects.** Review each goal's **habit targets** (adherence across the quarter) and **output targets** (current vs target — trending *toward or away*). A goal can look fine on projects while failing the number or the habit — treat a lagging output/habit exactly like a Behind project (re-prioritize or adjust). See `horizon-roadmap.md` → Goal composition.
- **If off track — make ONE conscious decision** (lettered table, per goal): **(A) Re-prioritize** — pull the lagging project(s) into the next month/sprint, bump Priority, or re-date earlier; or **(B) Adjust the goal** — move `Target Date`, cut scope (Success Metric), or drop it. This keeps goals **challenging but achievable per the active plan** — never let a goal silently miss. Record the decision on the Goal + the Quarterly Meeting Log.

<a id="behind-structured-shift"></a>
### Behind → structured shift

Surface `Behind` projects first (the `Behind` formula). Each gets exactly one call — **reslot** (push its `Date`; re-sequence), **cut/pause** (`⏸ Pause`), or **fund** (raise Priority + pull its Tasks into the month). If a reslot pushes a project past its Goal's `Target Date`, escalate to the goal decision above. Nothing slips without a decision.

<a id="pre-flight-silent"></a>
## Pre-Flight (silent)

Refresh Withings body-comp into Notion for the quarter:

```
node "scripts/withings-sync.mjs" --days 200 --write
```

That populates the Health Data DB for the outgoing quarter plus the prior quarter (so QoQ math works). Skip silently if it errors -- Notion still has whatever was previously synced.

Watch metrics are now archived **continuously** in the Notion Health Data DB via the **HC Webhook → n8n** pipeline — **`health_persist_recent` is retired** (2026-07-12; the old Health Sync→Drive→CSV→MCP writer is deprecated). Do **not** run it. `withings-sync` above still writes body comp. Older quarter days are already archived in Notion. If watch data is stale, check the **Health Sync Watchdog** / HC Webhook app on the phone.

(The persister is capped at 60 days because Drive doesn't reliably hold more than that. Quarterly history accumulates over time as monthly + weekly plans run their own persisters.)

Then call the **health-data MCP** for a 200-day consolidated pull:

```
health_get_summary({ days: 200 })
```

Use `daily` to slice "this quarter" vs. "prior quarter" for the body-comp + recovery rows in Phase 3. Daily rows are backed by the Notion Health Data DB archive, so historical days are still present even if Drive has rolled past them. If `sources.health_sync.ok` is false, only the most recent ~30 days might be slightly stale; older quarter data still flows from Notion. If the prior-quarter slice has fewer than ~45 days of watch data, annotate "(partial archive: <N> days)" on those rows -- the archive only goes as far back as we've been running the persister.

Pull data in parallel:
1. **Values DB** (`342f40c2-487b-80c5`): All 6 categories with Health statuses and full page content (Pictures of Success)
2. **Quarter Tracker** (`121f40c2-487b-802e`): Find outgoing quarter (Current=Previous or most recent) and current/new quarter. Also check whether future quarter records exist (Q[next+1], Q[next+2], Q[next+3]).
3. **Outgoing Quarter Tracker page**: the prior quarter's per-domain plan sections (theme / no-list / KPIs) written on its page (the retired Quarterly Outcomes DB's role)
4. **Projects** (`394f40c2-487b-815f`) + **Tasks** (`341f40c2-487b-80ac`): initiatives by Domain across quarter assignments (current, future, unassigned). Material for per-domain roadmap work. *(Browsing views: [Projects by Quarter](https://notion.so/39bf40c2487b816f9d31cfa7e2dec05f), [Projects — Active](https://notion.so/39bf40c2487b815e8e1cf4915f8212ef), [Goals](https://notion.so/39bf40c2487b813dbffbec3adaa277de).)*
5. **Departments** (`39bf40c2-487b-816d-97a3-f6f870b3b6e1`): unified dept health — filter `Domain = Chrome Lot` / `Turbo Gear`; pull Health, Priority, KPI/Target, `🥅 Goal`, `🚀 Fix Projects`, and Picture-of-Success page content
7. **Quarterly Meeting Log** (`344f40c2-487b-80ed`): Last 2 entries (outgoing quarter AND the one before it) for quarter-over-quarter trend comparison
8. **Weekly Meeting Log** (`322f40c2-487b-81bd`): All entries from the outgoing quarter for habit KPI averages
9. **Monthly Plan Log** (`344f40c2-487b-806d`): All entries from the outgoing quarter (typically 3) for intra-quarter month-over-month trajectory analysis. Body-comp fields (Weight Avg / Body Fat Avg / Lean Mass Avg + Deltas) feed Phase 3's Personal Habits trajectory.
10. **Hubstaff** (recent 90 days): Time-by-project and by-team-member for Phase 6 time-reality check
11. **Health Data + Watch metrics**: Loaded via the `health-data` MCP `health_get_summary({ days: 200 })` call above. Used in Phase 3 for body-comp + recovery QoQ trend.

Orient: "We're closing Q[X] and planning Q[Y]. This session runs in five parts -- Look Back, Look Around, Look Forward Strategically, Plan Tactically, Commit. Target 90-120 minutes. I'll announce part transitions and flag time overruns."

---

<a id="part-a-look-back-retrospective-25-min"></a>
## Part A -- Look Back (Retrospective) -- ~25 min

<a id="phase-1-wide-retrospective-narrative-10-min"></a>
### Phase 1: Wide Retrospective (narrative, ~10 min)

Before touching numbers, ask via AskQuestion (one at a time or batched):

- What was the biggest positive surprise this quarter?
- What was the biggest negative surprise?
- What did you learn about yourself? About Chrome Lot? About Turbo Gear?
- What's one thing you're proud of?
- What's one thing you'd redo if you could?

These lessons get captured in the Quarterly Meeting Log `Lessons Learned` field at the end. Surface them before the data biases the reflection.

<a id="phase-2-values-re-examination-10-min"></a>
### Phase 2: Values Re-examination (~10 min)

Not a yes/no "have your values shifted?" check. For each of the 6 values in the Values DB:

1. Read the full Picture of Success page content aloud (display it to Aaron).
2. Diagnose current Health in one sentence with evidence from the quarter's Weekly Meeting Log entries.
3. Ask: "Is this Picture still right? Aspirational-not-lived? Drifted? Does anything need editing?"
4. If edits are named, write them via `personal_notion_update_page` immediately. Same for Health status changes.

End of Phase 2, ask: **"Which value is most at risk in Q[next]? Which is strongest and can support the others?"** Capture the answer -- it informs Part C.

<a id="phase-3-numeric-retrospective-5-min"></a>
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

Body-comp Mo 1/2/3 cells come from each Monthly Plan Log entry's Weight Avg / Body Fat Avg / Lean Mass Avg. Qtr Avg = mean across the three months (or recompute from the `health_get_summary({ days: 200 })` daily slice for the quarter if a monthly entry is missing the field). Last Qtr = previous Quarterly Meeting Log entry's Weight Avg / Body Fat Avg / Lean Mass Avg. Trend = QoQ delta with direction; for body comp, also annotate alignment with Aaron's stated cut/bulk/recomp intent for the outgoing quarter.

Sleep / HR / RHR / HRV / Steps / Workout Active Min cells follow the same pattern: prefer each Monthly Plan Log entry's `Sleep Avg` / `Heart Rate Avg` / `Resting HR Avg` / `HRV Avg` / `Steps Avg` / `Workout Active Minutes` for Mo 1/2/3 cells. Fall back to recomputing from `daily` slice for any month missing those fields. Qtr Avg = mean across the quarter's daily rows. Last Qtr = previous Quarterly Meeting Log entry's matching field, OR recompute from the prior-quarter daily slice. Trend = QoQ delta. Render `--` for any row whose underlying data is null (Resting HR + HRV in particular until those Health Sync folders are enabled). If the prior-quarter slice covers fewer than ~45 days, annotate "(partial archive: <N> days)" beside that row.

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

<a id="phase-3b-monthly-health-trend-review-8-min"></a>
### Phase 3b: Monthly Health Trend Review (~8 min)

**Life-category Score trend:** pull the Weekly Meeting Log per-category **`* Score`** fields (1–5) across the quarter's ~13 weeks and show each category's average + direction — the quarter-level view of sustained life-health trends (feeds the sustained-unhealthy gate).

Pull the **3 Monthly Plan Log entries** from the outgoing quarter (chronological). Build domain-health and life-context tables:

```
WORK DOMAIN HEALTH -- INTRA-QUARTER
| Domain      | Mo1 | Mo2 | Mo3 | Streak        |
|-------------|-----|-----|-----|---------------|
| Chrome Lot  | H/U |     |     | N mos Unhealthy |
| Turbo Gear  | H/U |     |     |               |
```

Data source: `Chrome Lot Health` and `Turbo Gear Health` select properties on each Monthly Plan Log entry. For months before those properties existed, annotate `(no monthly snapshot)` and **do not false-trigger** the strategy gate.

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

<a id="part-b-look-around-environment-scan-15-min"></a>
## Part B -- Look Around (Environment scan) -- ~15 min

<a id="phase-4-people-and-relationships-7-min"></a>
### Phase 4: People & Relationships (~7 min)

Pull `context/people/index.md` and any overdue 1:1s from the Weekly Meeting Log entries.

- **Team**: Who's growing? Who's blocked? Who's underutilized? Who needs a harder conversation this quarter?
- **Personal relationships**: Kids, girlfriend/dating, social circle, mentors. Status of each. Anything needing reinvestment?
- **1:1 cadence**: Any that lapsed? What's the plan to rebuild?

Name 1-2 specific relationship actions that have to happen in Q[next].

<a id="phase-5-external-environment-5-min"></a>
### Phase 5: External Environment (~5 min)

What changed that affects the plan?

- Market / competitors / industry
- Knack product or pricing changes
- Economic context
- Family changes (kids' school, custody status, health)
- Legal proceedings
- Personal health changes

Capture as a one-paragraph narrative in the Quarterly Meeting Log entry at Phase 14.

<a id="phase-6-capacity-and-time-reality-3-min"></a>
### Phase 6: Capacity & Time Reality (~3 min)

Pull last quarter's Hubstaff totals and calendar analysis. Present:

- Where did hours actually go (by project / by domain / by activity type)?
- Where did you *intend* them to go?
- Where's the gap?

This surfaces the "I wanted Q[X] to be about TG but 60% of my hours went to CL field work" pattern before Part C tries to make allocation decisions.

**End of Part B:** state elapsed time. Target was 15 min.

---

<a id="part-c-look-forward-strategically-per-domain-35-min"></a>
## Part C -- Look Forward Strategically (per domain) -- ~35 min

**Pre-step (shared, ~2 min):** ensure Quarter Tracker (`121f40c2-487b-802e`) has records for Q[next+1], Q[next+2], Q[next+3]. Create any that are missing via `personal_notion_create_database_entry` so future-quarter project assignments in 7C/8C/9C have somewhere to land.

<a id="sustained-unhealthy-domain-gate-conditional-before-domain-blocks"></a>
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

<a id="phase-7-personal-strategic-forward-look-10-min"></a>
### Phase 7: Personal Strategic Forward-Look (~10 min)

**7A: Values recap (2 min).** Reference Phase 2. State which value is most at risk in Q[next] and which is strongest. Orient the Personal domain around these.

**7B: Personal Theme (2 min).** Via AskQuestion, name the Personal theme for Q[next] in one sentence. Examples:
- "Close out the legal/admin backlog so dev capacity is free in Q4."
- "Rebuild Spirituality after it got starved last quarter."
- "Protect family time through the custody modification."

This sentence gets written as a callout at the top of the **Personal** section on the Quarter Tracker page in Phase 11.

**7C: Personal Projects (6 min).** Walk the Tasks DB for the personal-side domains — Domain ∈ {Systems, Workshop, Admin} -- all statuses, all quarter assignments. (Systems = deep-work infrastructure; Workshop = capped QoL/hobby; Admin = life/legal.) For each live project, decide:

- Assign to Q[next] (active this quarter)
- Assign to Q[next+1] / Q[next+2] / Q[next+3] (on the roadmap, not now)
- Clear quarter, Status=Idea (someday/maybe -- still possible, not scheduled)
- Archive (dead -- won't happen, stop carrying it)

Use the Project Assessment Rules at the top of this file. Every live project ends this phase with either a `🍁 Quarter` relation or an explicit decision to kill it.

**Personal No-List:** name 2-3 Personal things deliberately not happening this year. Examples: "Not starting any new hobby projects until custody is resolved." "Not doing any home renovations in 2026." These get written as a "Deliberate No-List" section under **Personal** on the Quarter Tracker page in Phase 11.

<a id="phase-8-chrome-lot-strategic-forward-look-12-min"></a>
### Phase 8: Chrome Lot Strategic Forward-Look (~12 min)

**8A: CL Department Deep Read (4 min).** Pull the **Departments** DB (`39bf40c2-487b-816d-97a3-f6f870b3b6e1`, filter `Domain = Chrome Lot`) and display each department's Picture of Success page content. For each — this is the **KPI-driven re-assessment**:

- Pull the dept's **KPI vs Target** (from the dashboard / Pipedrive / Knack) and set **Health** (Healthy / Unhealthy / Critical / Stalled) + **Last Assessed**.
- Confirm/refresh Priority and the Picture of Success if reality moved.
- **Fix loop** — for every non-Healthy dept: either confirm its linked **`🚀 Fix Projects`** are on track (via the goal-trajectory review) or **create a fix-Project now** (Domain = Chrome Lot) linked to the dept + a **`🥅 Goal`** ("Get [Dept] to Healthy by [date]"). Never leave a Critical/Stalled dept without an owner-project.

Edit dept pages via `personal_notion_update_page`.

**8B: CL Theme (2 min).** Via AskQuestion, name the CL theme for Q[next] in one sentence. Examples:
- "Rebuild team Pipedrive cadence and resolve the two Unhealthy departments."
- "Instrument CL revenue and churn so next quarter has real numbers."
- "Get Aaron out of field work."

Written as a callout at the top of the **Chrome Lot** section on the Quarter Tracker page in Phase 11.

**8C: CL Projects (6 min).** Walk the Tasks DB with Domain=Chrome Lot -- all statuses, all quarter assignments. Same decision tree as 7C: Q[next], a future quarter, someday/Idea, or archive. Every live project ends with a clear home.

**CL No-List:** name 2-3 CL things deliberately not happening this year. Examples: "No new service line expansion this year." "Not hiring another senior until the ones we have are back on track." Written under **Chrome Lot** on the Quarter Tracker page in Phase 11.

<a id="phase-9-turbo-gear-strategic-forward-look-12-min"></a>
### Phase 9: Turbo Gear Strategic Forward-Look (~12 min)

**9A: TG Department Deep Read (4 min).** Pull the **Departments** DB (filter `Domain = Turbo Gear`). Same as 8A — KPI vs Target → set Health + Last Assessed, and spawn/verify `🚀 Fix Projects` (linked to a `🥅 Goal`) for every non-Healthy dept.

**9B: TG Theme (2 min).** Via AskQuestion, name the TG theme for Q[next] in one sentence. Examples:
- "Land the first three external demos."
- "Achieve Knack feature parity and stop shipping net-new internal features."
- "Build the external sales motion from zero."

Written as a callout at the top of the **Turbo Gear** section on the Quarter Tracker page in Phase 11.

**9C: TG Projects (6 min).** Walk the Tasks DB with Domain=Turbo Gear. Same decision tree as 7C/8C.

**TG No-List:** name 2-3 TG things deliberately not happening this year. Examples: "No new integrations until Knack parity is done." "Not building billing until first external customer commits." Written under **Turbo Gear** on the Quarter Tracker page in Phase 11.

**Project assignment rules (apply in 7C, 8C, 9C):**
- Every live project ends with either a `🍁 Quarter` relation or an explicit kill decision.
- Projects moved to a future quarter are real commitments, not wishes -- only assign if you'd take the same project this quarter if capacity allowed.
- The no-lists are not aspirational -- name things you can feel yourself being tempted to do but shouldn't.

**End of Part C:** state elapsed time. Target was 35 min.

---

<a id="part-d-plan-the-quarter-tactically-15-min"></a>
## Part D -- Plan the Quarter Tactically -- ~15 min

<a id="phase-10-commit-projects-to-this-quarter-10-min"></a>
### Phase 10: Commit Projects to This Quarter (~10 min)

Candidates are the projects Phases 7C/8C/9C tagged to Q[next]. For each:

- Apply Project Assessment Rules (never infer completion, check `Completion` rollup, ask about scope).
- Present the full list across all three domains with count per domain.
- **Cross-domain capacity check.** If the combined total exceeds ~10 projects, projects move to Q[next+1]. Push the lowest-leverage ones first; keep the ones aligned with each domain's theme.
- Final pick: 2-4 per domain, 6-10 total.
- Set `Due Date` on every selected project via `personal_notion_update_page`.
- Carry-overs from the outgoing quarter that didn't complete: honest call -- keep, rescope, or drop. If kept without a clear due date, mark Off Track.

<a id="phase-11-per-domain-kpi-targets-instrumentation-gate-5-min"></a>
### Phase 11: Per-Domain KPI Targets + Instrumentation Gate (~5 min)

**The Quarterly Outcomes DB is retired** — the quarterly plan is written **directly on the Q[next] Quarter Tracker page** (`121f40c2-487b-802e`), one section per domain, then rendered to a printable PDF (Phase 14). No separate outcomes DB.

For each domain (Personal, CL, TG), write its section on the **Q[next] Quarter Tracker page**. If re-running the same quarter, clear the prior plan sections first (delete blocks from the first `— Q[next]` domain heading onward), then rebuild:

1. Domain heading — H2 `<Domain> — Q[next]`.
2. Theme callout (from 7B/8B/9B) directly under the heading.
3. **Deliberate No-List** — H3 + bullet list.
4. **KPIs** — H3 + table (KPI / Target / Instrumented? / Gap), with the project list + Due Dates as context.
5. Set KPI targets relevant to the domain:
   - **Personal**: project completion rate, habit consistency goals, social interaction frequency
   - **CL**: revenue, customer count, churn, invoice aging, CS visit frequency, team Pipedrive activity
   - **TG**: features shipped, demos given, external customer conversations, deep work minutes

**Per-goal Targets.** Beyond domain KPIs, set/refresh each active goal's **habit + output Targets** — the goal's own instrumentation (e.g. "gym 4×/wk", "deadlift ≥ 500", "MRR ≥ $40k", "body fat ≤ 12%"). These feed the goal dashboard's KPI wheels and the trajectory chart. Apply the same instrumentation gate below to them: a target with no data source is dropped or gets an instrumenting project. See `horizon-roadmap.md` → Goal composition.

**Instrumentation gate.** For every KPI, confirm the data source exists. If instrumentation is missing (e.g., "CL Revenue" has no data source today), either (a) add a project to instrument it this quarter, or (b) drop the KPI. **No phantom targets.**

**End of Part D:** state elapsed time. Target was 15 min.

---

<a id="part-e-system-and-commit-15-min"></a>
## Part E -- System & Commit -- ~15 min

<a id="phase-12-system-and-cadence-calibration-5-min"></a>
### Phase 12: System & Cadence Calibration (~5 min)

Review the meta-system:
- Are daily morning/evening reviews happening consistently?
- Is the weekly meeting happening? (Count Weekly Meeting Log entries in the outgoing quarter vs. expected ~13)
- Are monthly plans happening?
- Are n8n workflows running reliably?
- Is the people directory up to date?
- Any cadences that have lapsed?
- Are the Weekly Meeting Log KPI fields being populated consistently?

**New rule:** if a cadence collapsed last quarter, name the specific change this quarter that's supposed to fix it. Don't just note "weekly meetings fell off" -- name the intervention. If there's no intervention, it'll happen again.

Adjust system parameters if needed (capacity limits, cadence frequencies, rotation). If context files change, update them in Phase 14.

**Operations catalog refresh:** run `node scripts/generate-ops-catalog.mjs` then `node scripts/publish-ops-catalog-to-notion.mjs`. Review the **Health / staleness** section in Notion ([Systems Hub](https://app.notion.com/p/Life-Command-Center-Systems-Hub-377f40c2487b8142b5fdfd6707572d29)) or `output/ops-catalog.md` — flag any stale context files or broken cross-references for update in Phase 14.

<a id="phase-13-calendar-stake-in-the-ground-5-min"></a>
### Phase 13: Calendar Stake-in-the-Ground (~5 min)

Block in Google Calendar (Aaron's calendar) for the full quarter:

- Weekly meeting slot (recurring, ~60 min)
- 3 monthly plan dates (~70 min each, first week of each month)
- 1 mid-quarter checkpoint at week 6 (~15 min -- lightweight "are we on track with the themes?")
- The Q[next+1] quarterly plan date, ~90 days out

These are the dates that get eaten first when things get busy. Block them now so they have to be explicitly moved.

<a id="phase-13b-planning-context-capture-3-min"></a>
### Phase 13b: Planning Context Capture (~3 min)

**Purpose:** Commit quarter-level priority stack and parked domains so monthly + weekly plans inherit reliable context.

**Required every quarterly plan.** Store in session state (`quarterly_priority_stack`, `quarterly_domains_parked`) for Phase 14.

1. **Draft Priority Stack** (max 5 numbered lines) from Part C themes + Part D project commits. Format: `N. [Domain] — [one-line focus]`.
2. **AskQuestion (multi-select):** "Which domains are **parked** for Q[next]?" Options: Turbo Gear | Chrome Lot New Business | External TG Demos | Personal Projects | Dating Outreach | Custody (hold) | Other | None — all domains active. Derive from no-lists and explicit pause decisions in Part C.
3. Confirm with Aaron before Phase 14.

<a id="phase-14-commit-and-log-5-min"></a>
### Phase 14: Commit & Log (~5 min)

1. Verify: all project assignments saved (quarter relations set), Due Dates on current-quarter projects, and all three domain sections written on the **Q[next] Quarter Tracker page** (theme callout + Deliberate No-List + KPIs table each), department health statuses current, future Quarter Tracker records exist. **Also verify** Phase 13b planning context captured.
1b. **Render the printable PDF (REQUIRED):** `node scripts/quarterly-plan-summary.mjs --quarter="Q[next]"` — renders the Quarter page's domain sections to a PDF, uploads to Plan Records/quarterly/, and sets **`Plan Doc URL`** on the Quarter record (same pattern as the weekly plan's Phase 4b). Aaron approves the production write.
2. **Log to Quarterly Meeting Log** (`344f40c2-487b-80ed`) via `personal_notion_create_database_entry`. New entry linked to the outgoing quarter with:
   - Meeting Date, Quarter relation
   - Wellness averages (PHQ-2, GAD-2, Energy -- averaged from Weekly Meeting Log entries)
   - Habit KPI averages (Strength, Cardio, Small Talk, Spirit, Deep Work, Ops, Field Work, Journal)
   - **Body composition** (from Phase 3): Weight Avg, Body Fat Avg, Lean Mass Avg (means across the outgoing quarter from Monthly Plan Log entries or the `health_get_summary` daily slice) plus Weight Delta, Body Fat Delta, Lean Mass Delta (this-quarter avg minus last-quarter avg, signed). Omit any field where the underlying data is null.
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

<a id="key-rules"></a>
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

<a id="outputs"></a>
## Outputs

- **Pre-Flight:** Withings refreshed; watch data persisted where possible; MCP `health_get_summary({ days: 200 })`; parallel data pulls oriented to Q transition.
- **Part A Phase 1–3:** Narrative Lessons Learned captured for Phase 14; Picture of Success updates applied in Notion; numeric retrospective tables; 2–3 named priority metrics for Q[next].
- **Part B Phase 4–6:** Relationship action commitments; environment narrative captured to Quarterly Meeting Log in Phase 14; Hubstaff/calendar gap surfaced.
- **Part C Phase 7–9:** Per-domain plan scaffolding (themes, no-lists prepared for Phase 11); Tasks quarter assignments cleared for every live row; Quarter Tracker futures created where missing.
- **Part D Phase 10–11:** Current-quarter commits (6–10 projects), Due Dates, KPI targets with instrumentation gate; per-domain sections written on the Quarter Tracker page + printable PDF rendered (`Plan Doc URL`).
- **Part E Phase 13b:** Priority Stack + Domains Parked captured (session state).
- **Part E Phase 12–14:** Calendar infrastructure for quarter/cadences; Quarterly Meeting Log entry with full rollup + planning context fields + Team Activity Details; context updates (`values.md`, `current-priorities.md`, `capacity-rules.md`, `people/index.md`).

<a id="failure-modes-and-graceful-degradation"></a>
## Failure modes & graceful degradation

- **Withings / `health_get_summary` errors:** Skip silently for Withings preload; QoQ Phase 3 may require monthly log fallbacks only.
- **`sources.health_sync.ok` false:** Recent days may be slightly stale; older quarter flows from Notion; annotate archive partiality `(partial archive: <N> days)` where applicable (Pre-Flight and Phase 3).
- **`health_persist_recent` capped at 60 days:** Quarterly history cumulative via monthly + weekly cadence (already documented Pre-Flight).
- **Instrumentation gate:** KPIs without measurable sources drop or spawn instrumenting projects — no phantom targets (Phase 11).
- **Time pressure:** If short on time mid-session, defer Part B detail before shortening Part A or Part C (`Key Rules`).

<a id="see-also"></a>
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