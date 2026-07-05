> **Source:** [`n8n/INDEX.md`](https://github.com/chromelot/life-command-center/blob/main/n8n/INDEX.md) in the private workspace repo. Do not edit this mirror directly.

# n8n — domain map

> **Read first:** [`conventions.md`](conventions.md) (cross-bot patterns) → this map → domain spec.

## Read order by task

| Task | Read |
|---|---|
| Any bot / Teams / deploy work | [`conventions.md`](conventions.md) |
| CL Bot skills, debug, extend | [`bots/cl-bot/teams-bot.md`](bots/cl-bot/teams-bot.md) |
| LCC Bot (personal habits, health) | [`bots/lcc-bot/lcc-bot.md`](bots/lcc-bot/lcc-bot.md) |
| PD ↔ Todoist sync | [`sync/pd-todoist/pd-todoist-sync.md`](sync/pd-todoist/pd-todoist-sync.md) |
| Inbox Guardian (email filter) | [`inbox-guardian/inbox-guardian.md`](inbox-guardian/inbox-guardian.md) |
| Toggl 2 → Notion time sync | [`sync/toggl-notion/toggl-notion-sync.md`](sync/toggl-notion/toggl-notion-sync.md) |
| Notion Start Timer → Toggl 2 | [`webhooks/toggl-start/toggl-start-code.mjs`](webhooks/toggl-start/toggl-start-code.mjs) |
| Tracker Matcher (period links) | [`sync/tracker-matcher/tracker-matcher-sync.md`](sync/tracker-matcher/tracker-matcher-sync.md) |
| Week Tracker Sunday create | [`sync/week-tracker/week-tracker-create.md`](sync/week-tracker/week-tracker-create.md) |
| Month/Quarter/Year rollover | [`sync/period-tracker/period-tracker-create.md`](sync/period-tracker/period-tracker-create.md) |
| AM strategy / roadmap | [`account-management/roadmap.md`](account-management/roadmap.md) |
| AM Process Street setup | [`sync/am-setup/ps-am-setup-verification.md`](sync/am-setup/ps-am-setup-verification.md) |
| Roster / permissions | [`shared/airtable-roster.mjs`](shared/airtable-roster.mjs) + `context/systems/airtable-roster.md` |

## Folder layout

```
n8n/
├── conventions.md          ← cross-cutting patterns (START HERE for agents)
├── INDEX.md                ← this file
├── README.md               ← quick pointer
├── .secrets.local          ← credentials (gitignored)
├── knack-schema.json       ← generated; run tools/explore-knack-schema.mjs
│
├── bots/cl-bot/            ← Teams Bot workflow + manifest
│   ├── teams-bot.md
│   ├── deploy-teams-bot.mjs
│   ├── check-bot-codestrings.mjs
│   └── teams-app/
├── bots/lcc-bot/           ← Personal LCC Bot (Aaron only)
│   ├── lcc-bot.md
│   ├── deploy-lcc-bot.mjs
│   └── teams-app/
│
├── sync/pd-todoist/        ← real-time PD ↔ Todoist stack
│   ├── pd-todoist-sync.md
│   ├── deploy-pd-activity-mirror.mjs
│   ├── deploy-todoist-events.mjs
│   ├── deploy-pd-todoist-schedules.mjs
│   ├── deploy-pd-deal-delegate-label.mjs
│   └── check-pd-todoist-codestrings.mjs
│
├── sync/am-setup/          ← Process Street AM setup verifier
│   ├── ps-am-setup-verification.md
│   ├── deploy-am-setup-verifier.mjs
│   └── ...
│
├── inbox-guardian/         ← LLM email filter for aaron@chromelot.com
│   ├── inbox-guardian.md
│   └── deploy-inbox-guardian.mjs
│
├── sync/toggl-notion/      ← Toggl 2 → Time Punches (2-min poll)
│   ├── toggl-notion-sync.md
│   ├── deploy-toggl-notion-sync.mjs
│   ├── bootstrap-toggl2-secrets.mjs
│   └── toggl-notion-code.mjs
├── webhooks/toggl-start/   ← Notion ▶ Start Timer → Toggl 2
│   ├── toggl-start-code.mjs
│   └── deploy-toggl-start.mjs
├── sync/tracker-matcher/   ← Notion page.created → Day/Week/Month/Quarter/Year links
│   ├── tracker-matcher-sync.md
│   └── deploy-tracker-matcher.mjs
│
├── shared/                 ← modules imported by deploy scripts
│   ├── airtable-roster.mjs
│   ├── pd-delegate-resolution.mjs
│   └── *.json field maps
│
├── account-management/     ← AM stack strategy
│   ├── README.md
│   └── roadmap.md
│
├── specs/                  ← cron-only workflow specs (no deploy script)
│   └── weekly-metrics.md, ...
│
└── tools/                  ← one-shot CLIs, schema explorer
    └── explore-knack-schema.mjs, backfill-*.mjs, ...
```

## Deploy commands (canonical paths)

```sh
# CL Bot
node n8n/bots/cl-bot/check-bot-codestrings.mjs
node n8n/bots/cl-bot/deploy-teams-bot.mjs

# PD ↔ Todoist (order matters on first setup — see pd-todoist-sync.md)
node n8n/sync/pd-todoist/check-pd-todoist-codestrings.mjs
node n8n/sync/pd-todoist/deploy-pd-activity-mirror.mjs
node n8n/sync/pd-todoist/deploy-todoist-events.mjs
node n8n/sync/pd-todoist/deploy-pd-todoist-schedules.mjs
node n8n/sync/pd-todoist/deploy-pd-deal-delegate-label.mjs

# Inbox Guardian (email filter) — see inbox-guardian.md for one-time Gmail OAuth
node n8n/inbox-guardian/deploy-inbox-guardian.mjs --dry-run   # validate
node n8n/inbox-guardian/deploy-inbox-guardian.mjs             # shadow (default)
node n8n/inbox-guardian/deploy-inbox-guardian.mjs --enforce   # armed

# Toggl 2 → Notion (Time Punches poll + start timer) — see sync/toggl-notion/toggl-notion-sync.md
node n8n/sync/toggl-notion/bootstrap-toggl2-secrets.mjs --token=toggl_sk_...
node n8n/sync/toggl-notion/deploy-toggl-notion-sync.mjs --dry-run
node n8n/sync/toggl-notion/deploy-toggl-notion-sync.mjs
node n8n/webhooks/toggl-start/deploy-toggl-start.mjs

# Tracker Matcher — see sync/tracker-matcher/tracker-matcher-sync.md
node n8n/sync/tracker-matcher/deploy-tracker-matcher.mjs

# Morning Journal Notion webhook — see sync/morning-journal-notion/morning-journal-notion-sync.md
node n8n/sync/morning-journal-notion/deploy-morning-journal-notion-sync.mjs
node n8n/sync/morning-journal-notion/register-notion-morning-journal-webhook.mjs
```

## Deprecated stub paths (still work)

Muscle-memory aliases at `n8n/` root forward to canonical locations:

| Stub | Forwards to |
|---|---|
| `deploy-teams-bot.mjs` | `bots/cl-bot/deploy-teams-bot.mjs` |
| `check-bot-codestrings.mjs` | `bots/cl-bot/check-bot-codestrings.mjs` |
| `deploy-todoist-events.mjs` | `sync/pd-todoist/deploy-todoist-events.mjs` |
| `deploy-pd-activity-mirror.mjs` | `sync/pd-todoist/deploy-pd-activity-mirror.mjs` |
| `deploy-pd-todoist-schedules.mjs` | `sync/pd-todoist/deploy-pd-todoist-schedules.mjs` |
| `deploy-pd-deal-delegate-label.mjs` | `sync/pd-todoist/deploy-pd-deal-delegate-label.mjs` |
| `check-pd-todoist-codestrings.mjs` | `sync/pd-todoist/check-pd-todoist-codestrings.mjs` |
| `deploy-am-setup-verifier.mjs` | `sync/am-setup/deploy-am-setup-verifier.mjs` |
| `account-management-roadmap.md` | `account-management/roadmap.md` |

## Domain tags (for ops catalog)

| Tag | Folder |
|---|---|
| **CL Bot** | `bots/cl-bot/` |
| **PD↔Todoist** | `sync/pd-todoist/` |
| **AM Setup** | `sync/am-setup/` |
| **AM Strategy** | `account-management/` |
| **Inbox Guardian** | `inbox-guardian/` |
| **Toggl→Notion** | `sync/toggl-notion/` + `webhooks/toggl-start/` |
| **Tracker Matcher** | `sync/tracker-matcher/` |
