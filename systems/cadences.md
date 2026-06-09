> **Source:** [`context/systems/cadences.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/cadences.md) in the private workspace repo. Do not edit this mirror directly.

# Cadences — schedule and tier ownership

Every review and maintenance activity has a defined cadence and tier assignment. **If it can run without Aaron's attention, it does.**

## Two-tier model

- **Tier 1 — n8n** (scheduled, runs on DigitalOcean)
- **Tier 2 — Cursor** (interactive, this layer)

Daily morning/evening Cursor reviews were retired. The PD ↔ Todoist loop is real-time (webhooks) plus scheduled safety nets — see [`n8n/sync/pd-todoist/pd-todoist-sync.md`](../../n8n/sync/pd-todoist/pd-todoist-sync.md).

---

## Daily

### Tier 1 (n8n) — PD ↔ Todoist

| Time | Workflow | What |
|---|---|---|
| Real-time | PD Activity Mirror | PD activity events → shared `Pipedrive` Todoist project |
| Real-time | Todoist Events | Todoist completions/dates/notes → Pipedrive |
| 4:00 AM CT (Mon-Fri) | Team PD Reconciler | Drift heal between PD and shared Todoist project |
| 5:00 PM CT (Mon–Sat) | Team Evening Defer | Defer open activities due ≤ today (Sat → Mon; closed Sun; no Teams DMs) |
| On demand | CL Bot (`today`, `deal gaps`) | AM day summary (replaces retired 9am Morning DM) |

> Daily morning/evening Cursor sessions remain retired.

---

## Weekly

### Tier 1 (n8n)

| Day / Time | Workflow | What |
|---|---|---|
| Sunday 8 PM | Weekly Metrics | Aggregate Todoist completions, Pipedrive velocity, calendar analysis, rescheduling rate to Google Sheets |
| Daily | 1:1 Reminder Check | Compare people directory last-checkin dates against frequency, create Todoist task if overdue (max 1/day) |
| Tue 8 AM CT | Weekly Hours Report | Posts to team channel via CL Bot (lives inside the `Teams Bot` n8n workflow) |

### Tier 2 (Cursor)

| Workflow | Duration | Trigger |
|---|---|---|
| Weekly Plan | ~40 min | "weekly plan", "weekly meeting", "Monday review", "sprint planning" |
| Weekly Ops | ~40 min | "weekly ops", "ops review", "CL ops meeting", "operations meeting" |

#### Weekly Plan — Pre-Phase 0 gate + 9-phase workflow

**Pre-Phase 0 — Monthly plan gate:** Query Monthly Plan Log for `Monthly Plan for [Planning Month YYYY]` (current calendar month). If no entry exists, pause and run the full monthly plan (~83 min) before continuing. Hard prerequisite — no weekly-only bypass.

0. Data Pull (silent — Todoist, Pipedrive, Calendar, Notion source DBs incl. Monthly Plan Log, Knack, Hubstaff, health-data MCP)
1. Center (wellness screening, values pulse, capacity adjustment)
1a. Monthly Plan Review (align week with committed monthly Action Items + KPIs)
2. Last Week Review (habit scorecard, project review)
3. Project Selection (One Thing Framework — pick 1-3 Dev Projects, set This Week; schedule Personal-project tasks + Todoist mirrors, incl. custody)
4. CS Management (check-in cadence + delegation; late invoice review)
5. Sales Management (Aaron's pipeline; team sales oversight)
6. People Management (photographer review; staff 1:1s + Hubstaff)
7. Personal Life & Social (Bus, social, dating, compulsion scan)
8. Commit (summary, capacity check, execute, log to Weekly Meeting Log)

> **Actual cadence + catch-up retune (2026-06-02):** Aaron runs this consistently, usually **Fridays** (sometimes later), not Monday. The gap is that it re-plans but doesn't *clear* the Chrome Lot backlog. Retune: add a standing **currency check** (Pipedrive overdue count, oldest 1:1, stale CS) and a hard rule that each meeting must **retire a defined slice** of backlog, not just reschedule it — calibrated to the capacity caps.

---

## Monthly

### Tier 2 (Cursor)

| Workflow | Duration | Trigger |
|---|---|---|
| Monthly Plan | 83 min | "monthly plan", "monthly review" |
| CL Projects Review | 15 min | "review CL projects" |
| Deep Pipedrive Pipeline Review | 15 min | Once/month, included in weekly |

#### Monthly Plan — 14-phase workflow

Forward-looking: **plan current month**, **review prior month** (early June → plan June, review May).

0. Data Pull (silent — Withings, health persist, health_source_status, review-month sources)
1. Wellness Trends (PHQ-2/GAD-2 trend, energy avgs, values category check — **review month**)
1b. Identity Check (six values On track/Slipping/Stalled)
1c. Quarterly Plan Checkpoint (**hard gate** — stop monthly plan if Q plan not committed; run quarterly plan first)
2. Fitness Review (consistency trends, body comp + recovery MoM for **review month**)
3. Dev Work Review & Goals (quarter progress; set 2–3 goals for **planning month**)
3b. Idea Roadmap Scrub (TG → CL → Personal; promote ≤3/type)
4. Sales Progress & Goals (**review month** retro; **planning month** targets)
5. Personal Project Tracker Audit (Dev Projects Type=Personal, stalled, archive dead)
6. Financial Review (personal budget, CL revenue/AR for **review month**)
7. CS Health Monthly Review (churn trends, invoice aging, workload balance)
8. People & Operations Trends (photographer trends, Hubstaff trends, 1:1 consistency, hiring)
9. Quarterly KPI Update (current-quarter Quarterly Outcomes, update KPI actuals for **review month**)
10. Trip Planning with Bus (**planning month** — calendar + Todoist)
11. Personal Time Off (2 days off in **planning month**, coverage coordination)
12. Commit (Monthly Log for **review month**; Action Items for **planning month**; Pipedrive append)

### Tier 1 (n8n)

| Workflow | What |
|---|---|
| Monthly Metrics Summary | Compile month-over-month trends from Google Sheets data |

---

## Quarterly

### Tier 2 (Cursor)

| Workflow | Duration | Trigger |
|---|---|---|
| Quarterly Plan | 90-120 min | "quarterly plan", "strategy review" |
| Mid-Quarter Checkpoint | 15 min | "mid-quarter check" (week 6 of quarter) |

#### Quarterly Plan — 5-part, 14-phase workflow

- **Part A — Look Back (~25 min)**: 1) Wide narrative retrospective, 2) Values re-examination with Picture of Success edits, 3) Numeric retrospective (dual-level trend tables)
- **Part B — Look Around (~15 min)**: 4) People & relationships, 5) External environment scan, 6) Capacity & time reality (Hubstaff vs. intent)
- **Part C — Look Forward Strategically (~35 min)**: pre-step creates future Quarter Tracker records, then per-domain blocks: 7) Personal (values recap → theme → roadmap → no-list), 8) Chrome Lot (department read → theme → roadmap → no-list), 9) Turbo Gear (department read → theme → roadmap → no-list)
- **Part D — Plan Tactically (~15 min)**: 10) Commit projects to this quarter with cross-domain capacity check + Due Dates, 11) Per-domain KPI targets + instrumentation gate (no phantom targets)
- **Part E — System & Commit (~15 min)**: 12) System & cadence calibration (name the intervention for any collapsed cadence), 13) Calendar stake-in-ground (weekly meeting + monthly plans + mid-quarter checkpoint), 14) Commit & log (Quarterly Meeting Log entry, team Pipedrive detail append, context file updates)

#### Mid-Quarter Checkpoint (week 6, ~15 min)

Lightweight "are we on track with the three domain themes?" review. Pulls current-quarter Dev Projects with Due Date, checks what slipped or stalled, confirms Q[next+1] plan date is still on calendar. Not a replacement for monthly plan — just a theme check.

---

## Missed cadence detection

The weekly meeting rule automatically checks:
- Whether the monthly plan has been committed (Pre-Phase 0 gate — escalates to full monthly plan if review-month Monthly Log entry is missing)
- How long since last strategy review
- Which 1:1 meetings are overdue

If a cadence is falling behind, it surfaces during the weekly meeting with a specific recommendation (do it now, schedule it, or deprioritize it).

## See also

- [capacity-rules.md](capacity-rules.md) — daily/weekly limits enforced by these cadences
- [routing-rules.md](routing-rules.md) — where new items go (intake)
- [../router.md](../router.md) — workflow triggers