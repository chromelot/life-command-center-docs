> **Source:** [`output/ops-catalog.md`](https://github.com/chromelot/life-command-center/blob/main/output/ops-catalog.md) in the private workspace repo. Do not edit this mirror directly.

# Operations Catalog — Life Command Center

> **Control plane, not procedure.** This page is the human-facing map of every cadence, automation, and integration in the Life Command Center. Phase-by-phase workflow detail lives in `context/skills/*/SKILL.md`; deploy specs live in `n8n/`; IDs and schemas live in `context/systems/`.

## Two-tier model

- **Tier 1 — n8n** (DigitalOcean): scheduled and webhook automations that run without Aaron's attention. Registry: [`n8n/README.md`](../../n8n/README.md).
- **Tier 2 — Cursor** (interactive): planning sessions, scrubs, triage, and 1:1 prep triggered by phrase. Rules: [`.cursor/rules/`](../../.cursor/rules/); procedures: [`context/skills/`](../skills/).

Full schedule and tier ownership: [`cadences.md`](cadences.md). Agent intent routing (separate concern): [`router.md`](../router.md).

## How to review

| When | Action |
|---|---|
| After changing a workflow rule, skill, or n8n spec | Push to GitHub (auto-sync) — docs rebuild via Actions; `node scripts/publish-notion-index.mjs` refreshes Notion index |
| Quarterly plan Part E Phase 12 | Regenerate catalog, review **Health / staleness** on [GitHub Pages docs](https://chromelot.github.io/life-command-center-docs/ops-catalog) |
| Weekly (optional) | `OpsCatalogSync` on lcc-hub (Sunday 8:15 PM CT) — regenerate + Notion index |
| Human-readable docs | [life-command-center-docs](https://chromelot.github.io/life-command-center-docs/) (GitHub Pages) |
| Notion index | [Systems Hub](https://app.notion.com/p/Life-Command-Center-Systems-Hub-377f40c2487b8142b5fdfd6707572d29) — links only |
| Private workspace | [life-command-center](https://github.com/chromelot/life-command-center) on GitHub |

## Drill-down map

| Need | Read |
|---|---|
| Full workflow procedure (phases, data pulls, writes) | [GitHub Pages workflows](https://chromelot.github.io/life-command-center-docs/) or `context/skills/<name>/SKILL.md` |
| Cursor trigger constraints | `.cursor/rules/<name>.mdc` |
| n8n deploy + debug | `n8n/conventions.md`, `n8n/INDEX.md`, `n8n/bots/cl-bot/teams-bot.md`, `n8n/sync/pd-todoist/pd-todoist-sync.md` |
| Human-readable automation guides (Notion) | `context/systems/notion-guides/` — sub-pages under Systems Hub |
| Automation conventions (Teams UX, cards, deploy) | `n8n/conventions.md` |
| Notion DB IDs and schemas | `context/systems/notion-databases.md` |
| MCP inventory | `context/systems/mcp-servers.md` |
| Capacity limits | `context/systems/capacity-rules.md` |
| Meeting log outputs | Notion Weekly / Monthly / Quarterly Meeting Log DBs |

---

## Cadences at a glance

<!-- AUTO:CADENCE_MATRIX -->
| Workflow | Tier | Cadence | Duration | Phases | Gates | Triggers |
|---|---|---|---|---|---|---|
| weekly plan | Tier 2 (Cursor) | Weekly (~Fridays) | ~40 min (life + dev) | 1 life + 2 dev + commit | Monthly plan (review month) must exist | "weekly plan", "weekly meeting", "Monday review", "sprint planning" |
| weekly ops | Tier 2 (Cursor) | Weekly (separate session) | ~40 min | ops + CS + sales + people + commit | Same planning week as weekly plan | "weekly ops", "ops review", "CL ops meeting", "operations meeting" |
| monthly plan | Tier 2 (Cursor) | Monthly (first week) | 83 min | 14 | Quarterly plan must be committed | "monthly plan", "monthly review" |
| quarterly plan | Tier 2 (Cursor) | Quarterly (week 1) + mid-quarter week 6 | 90–120 min / 15 min checkpoint | 5-part, 14 phases | None (top-level strategic gate) | "quarterly plan", "strategy review" |
| annual plan | Tier 2 (Cursor) | On demand | — | — | — | "annual plan", "year kickoff", "plan this year" |
| 1:1 prep | Tier 2 (Cursor) | On demand (before check-ins) | ~5 min | inline in rule | None | "1:1 prep", "person check-in", "<name> check-in" |
| team sync | Tier 2 (Cursor) | On demand | ~10 min | inline in rule | None | "team sync", "roster diff", "people sync", "audit roster" |
| pd cleanup | Tier 2 (Cursor) | On demand | 30–45 min | 6 | Explicit approval per batch | "pd cleanup", "pipedrive cleanup", "deal cleanup", "crm cleanup" |
| Custody-related file edits | Tier 2 (Cursor) | On file edit | varies | inline in rule | FROZEN flag on summary-for-hope.md | Custody-related file edits |
| PD ↔ Todoist sync | Tier 1 (n8n) | Real-time + 4am/5pm CT (no scheduled DMs) | — | — | — | Webhooks + cron |
| Weekly Metrics | Tier 1 (n8n) | Sunday 8 PM CT | — | — | — | Cron |
| 1:1 Reminder Check | Tier 1 (n8n) | Daily 8 AM CT | — | — | — | Cron |
| Weekly Hours Report | Tier 1 (n8n) | Tuesday 8 AM CT | — | — | — | Cron via Teams Bot |
| Monthly Metrics Summary | Tier 1 (n8n) | Monthly | — | — | — | Cron |
<!-- /AUTO:CADENCE_MATRIX -->

---

## Tier 1 — n8n automations

<!-- AUTO:TIER1_AUTOMATIONS -->
| Source / workflow | Trigger | Purpose |
|---|---|---|
| `bots/cl-bot/deploy-teams-bot.mjs` → **CL Bot** | Multiple webhooks + Tue 8am CT | Bidirectional CL Bot. Dispatcher, `teams-bot-send`, OAuth, Weekly Hours. See `bots/cl-bot/teams-bot.md`. |
| `sync/pd-todoist/deploy-pd-activity-mirror.mjs` → **PD Activity Mirror** | Webhook `pd-activity-event` | Real-time PD → Todoist mirror. See `sync/pd-todoist/pd-todoist-sync.md`. |
| `sync/pd-todoist/deploy-todoist-events.mjs` → **Todoist Events** | Webhook `todoist-events` | Todoist → PD: complete, notes, auto-next (+3d). |
| `sync/pd-todoist/deploy-pd-todoist-schedules.mjs` → **Team Evening Defer / Reconciler** | Cron + manual webhooks | 5pm defer (silent), 4am reconciler. |
| `sync/pd-todoist/deploy-pd-deal-delegate-label.mjs` → **PD Deal Delegate Label** | Webhook `pd-deal-event` | Sub AM Delegated label sync. |
| `sync/am-setup/deploy-am-setup-verifier.mjs` → **AM Setup Verifier** | Webhook | PS AM setup ↔ Airtable verification. |
| `inbox-guardian/deploy-inbox-guardian.mjs` → **Inbox Guardian** | Gmail trigger 1min + cron 4pm CT | LLM cold-pitch filter for aaron@chromelot.com (whitelist + OpenAI; spam/archive). See `inbox-guardian/inbox-guardian.md`. |
| `specs/weekly-metrics.md` | Cron Sun 8pm ET | Weekly metrics to Google Sheets. |
| `specs/one-on-one-reminders.md` | Cron 8am ET daily | Overdue 1:1 reminders. |
| `specs/health-data-sync.md` | Webhook | Health data → Notion. |

Spec files: `n8n/conventions.md`, `n8n/bots/cl-bot/teams-bot.md`, `n8n/sync/pd-todoist/pd-todoist-sync.md`, plus `n8n/specs/` and domain READMEs.
<!-- /AUTO:TIER1_AUTOMATIONS -->

---

## Tier 2 — Cursor workflows

<!-- AUTO:TIER2_WORKFLOWS -->
| Triggers | Cursor rule | Skill / procedure | Core context |
|---|---|---|---|
| "weekly plan", "weekly meeting", "Monday review", "sprint planning" | `.cursor/rules/weekly-meeting.mdc` | `context/skills/weekly-planning/SKILL.md` | systems/cadences.md, systems/capacity-rules.md, systems/notion-databases.md, self/values.md, people/index.md |
| "weekly ops", "ops review", "CL ops meeting", "operations meeting" | `.cursor/rules/weekly-ops.mdc` | `context/skills/weekly-ops/SKILL.md` | systems/cadences.md, systems/pipedrive.md, systems/knack-fields.md, work/chrome-lot/, people/index.md |
| "monthly plan", "monthly review" | `.cursor/rules/monthly-plan.mdc` | `context/skills/monthly-plan/SKILL.md` | systems/cadences.md, systems/notion-databases.md, self/values.md, people/index.md, work/chrome-lot/, work/turbo-gear/ |
| "quarterly plan", "strategy review" | `.cursor/rules/strategy-review.mdc` | `context/skills/quarterly-plan/SKILL.md` | self/values.md, work/chrome-lot/, work/turbo-gear/, self/, people/index.md |
| "annual plan", "year kickoff", "plan this year" | `.cursor/rules/annual-plan.mdc` | `context/skills/annual-plan/SKILL.md` | self/values.md, systems/notion-databases.md (Goals/Projects/Year Tracker), work/chrome-lot/, work/turbo-gear/ |
| "1:1 prep", "person check-in", "<name> check-in" | `.cursor/rules/person-checkin.mdc` | `(inline in rule)` | people/index.md, [domain folder for that person] |
| "team sync", "roster diff", "people sync", "audit roster" | `.cursor/rules/team-sync.mdc` | `(inline in rule)` | systems/airtable-roster.md, people/index.md, cast.md |
| "pd cleanup", "pipedrive cleanup", "deal cleanup", "crm cleanup" | `.cursor/rules/pd-cleanup.mdc` | `context/skills/pd-cleanup/SKILL.md` | systems/pipedrive.md, systems/knack-fields.md, work/chrome-lot/customer-service.md |
| Custody-related file edits | `.cursor/rules/custody-modification.mdc` | `(inline in rule)` | family/custody/ |
<!-- /AUTO:TIER2_WORKFLOWS -->

---

## Workflow guides (full breakdowns)

Full phase-by-phase procedures live in Notion **sub-pages** (sidebar under this hub). Canonical source: `context/skills/*/SKILL.md` and `.cursor/rules/*.mdc`.

<!-- AUTO:WORKFLOW_GUIDES -->
| Workflow | Summary | Duration | Triggers | Session log | Workspace source |
|---|---|---|---|---|---|
| Weekly Plan | Personal life review, then Dev Review (CL/TG) and Personal Project Review — selects This Week from monthly Dev Projects tracker and syncs Notion. | ~40 min (life + dev) | "weekly plan", "weekly meeting", "Monday review", "sprint planning" | Weekly Meeting Log | `context/skills/weekly-planning/SKILL.md` |
| Weekly Ops | Chrome Lot operations session — eight domain phases (CS, sales, finance, service delivery, post production, workload, 1:1s) aligned to Dev Projects. Separate from weekly plan; writes to Weekly Ops Meeting Log. | ~40 min | "weekly ops", "ops review", "CL ops meeting", "operations meeting" | Weekly Ops Meeting Log | `context/skills/weekly-ops/SKILL.md` |
| Monthly Plan | Forward-looking planning for the current calendar month backed by a structured review of the prior month across life, business, and people domains. | 83 min | "monthly plan", "monthly review", "plan this month" | Monthly Plan Log | `context/skills/monthly-plan/SKILL.md` |
| Quarterly Plan | Quarterly strategy review — retrospective, environment scan, per-domain themes and no-lists, tactical quarter commits, and Quarterly Meeting Log. | 90–120 min | "quarterly plan", "strategy review", "quarterly review", "big picture" | Quarterly Meeting Log | `context/skills/quarterly-plan/SKILL.md` |
| 1:1 Prep | Five-minute pre-meeting pull of a team member's workload from Todoist, Notion, and Pipedrive plus structured talking points for the check-in. | ~5 min | "1:1 prep", "prep for [name]", "meeting with [name]", "check in with [name]" | 1:1 Prep Log | `.cursor/rules/person-checkin.mdc` |
| Team Sync | Diff Airtable Admin/Payable Employees against people/index.md to catch roster, role, and platform-ID drift before it breaks bot permissions or 1:1 prep. | ~10 min | "team sync", "roster diff", "people sync", "audit roster" | Team Sync Log | `.cursor/rules/team-sync.mdc` |
| Pipedrive Cleanup | Knack↔Pipedrive deal hygiene audit with per-item Aaron approval only — creates, adopts, deletes, org/POC fixes, duplicates, and wrong-pipeline moves. | 30–45 min | "pd cleanup", "pipedrive cleanup", "deal cleanup", "crm cleanup" | PD Cleanup Log | `context/skills/pd-cleanup/SKILL.md` |

_Notion sub-page links are added when you run `publish-ops-catalog-to-notion.mjs`._
<!-- /AUTO:WORKFLOW_GUIDES -->

---

## Automation guides (Notion sub-pages)

Human-readable deep dives for Tier 1 automations. Canonical workspace sources: `context/systems/notion-guides/` + `n8n/`.

<!-- AUTO:AUTOMATION_GUIDES -->
| Guide | Summary | Workspace source |
|---|---|---|
| Chrome Lot Bot (CL Bot) | Teams interface for AMs, managers, and ops — deal lookup, scheduling, guides, Weekly Hours. | `n8n/bots/cl-bot/teams-bot.md` + `context/systems/notion-guides/cl-bot.md` |
| PD ↔ Todoist Automations | Real-time Pipedrive ↔ Todoist sync, evening defer, 4am reconciler, auto-next. | `n8n/sync/pd-todoist/pd-todoist-sync.md` + `context/systems/notion-guides/pd-todoist-automation.md` |
| n8n Tier 1 Automations | All scheduled and webhook automations on DigitalOcean — registry, deploy, conventions. | `n8n/README.md` + `context/systems/notion-guides/n8n-automations.md` |

_Notion sub-page links are added when you run `publish-ops-catalog-to-notion.mjs`._
<!-- /AUTO:AUTOMATION_GUIDES -->

---

## Infrastructure

<!-- AUTO:INFRASTRUCTURE -->
| File | Covers |
|---|---|
| mcp-servers.md | Inventory of every MCP server, role, key tools, when to call which |
| notion-databases.md | All Notion DB IDs and property schemas (Personal + Business) |
| pipedrive.md | Pipelines, stages, user IDs, API quirks, deal-naming conventions |
| knack-fields.md | All Knack object IDs and field references for ops review |
| hubstaff.md | Hubstaff API specifics, member IDs, project IDs |
| health-data.md | Health Data MCP architecture, Withings + Health Sync data flow |
| skin-care.md | Skin Care MCP architecture, Notion logs + Drive clinical files |
| operations-catalog.md | Control-plane hub — all cadences, automations, infrastructure (Notion mirror) |
| notion-guides/ | Human-readable Notion sub-page sources (CL Bot, PD↔Todoist, n8n Tier 1) |
| cadences.md | Daily / weekly / monthly / quarterly cadences and tier ownership |
| capacity-rules.md | Daily/weekly limits, overcommitment triggers, intervention protocol |
| routing-rules.md | Where new items go (intake routing) |
| task-glossary.md | Opaque task names (Bench, Pay and Invoice, Treat Your Feet, Workout Sync) |
| n8n/conventions.md | Cross-bot automation patterns (Teams formatting, deploy, permissions) — read before editing deploy scripts |
| synology-nas.md | Interval NAS — model, DSM version, SSH access, share layout, hardware constraints |

Host runbooks: `context/systems/lcc-hub.md`, `context/systems/media-pc.md`, `context/systems/synology-nas.md`.

Project runbooks: `context/projects/plex-media-stack.md`, `n8n/README.md`.
<!-- /AUTO:INFRASTRUCTURE -->

---

## Health / staleness

<!-- AUTO:HEALTH_REPORT -->
⚠ **14 issue(s)** across 97 context files.

### stale (2)
- `context/self/current-priorities.md` — 36d old (threshold 14d)
- `context/systems/operations-catalog.md` — 37d old (threshold 30d)

### missing-frontmatter (11)
- `context/skills/annual-plan/SKILL.md` — no-frontmatter
- `context/skills/planning-sprint/SKILL.md` — no-frontmatter
- `context/skills/weekly-planning/SKILL.md` — no-frontmatter
- `context/systems/cursor-model-strategy.md` — no-frontmatter
- `context/systems/notion-databases.md` — no-frontmatter
- `context/systems/notion-guides/cl-bot-first-time-setup.md` — no-frontmatter
- `context/systems/notion-guides/cl-bot.md` — no-frontmatter
- `context/systems/notion-guides/n8n-automations.md` — no-frontmatter
- `context/systems/notion-guides/pd-todoist-automation.md` — no-frontmatter
- `context/systems/readings-curation.md` — no-frontmatter
- `context/workflow-execution.md` — no-frontmatter

### broken-see-also (1)
- `context/systems/audiobook-pipeline.md` — ../../output/audiobook-handoff-2026-05-05.md

Full audit: `node scripts/audit-context.mjs`
<!-- /AUTO:HEALTH_REPORT -->

---

## Last generated

<!-- AUTO:LAST_GENERATED -->
**2026-07-13 08:22:54** (America/Chicago) — source: `scripts/generate-ops-catalog.mjs`
<!-- /AUTO:LAST_GENERATED -->

## See also

- [cadences.md](cadences.md) — schedule and phase summaries
- [../router.md](../router.md) — agent intent-to-path map
- [../skills/index.md](../skills/index.md) — skill folder index
- [../../n8n/README.md](../../n8n/README.md) — Tier 1 automation registry
- [index.md](index.md) — systems reference files