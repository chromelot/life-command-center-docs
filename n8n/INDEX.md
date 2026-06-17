> **Source:** [`n8n/INDEX.md`](https://github.com/chromelot/life-command-center/blob/main/n8n/INDEX.md) in the private workspace repo. Do not edit this mirror directly.

# n8n — domain map

> **Read first:** [`conventions.md`](conventions.md) (cross-bot patterns) → this map → domain spec.

## Read order by task

| Task | Read |
|---|---|
| Any bot / Teams / deploy work | [`conventions.md`](conventions.md) |
| CL Bot skills, debug, extend | [`bots/cl-bot/teams-bot.md`](bots/cl-bot/teams-bot.md) |
| PD ↔ Todoist sync | [`sync/pd-todoist/pd-todoist-sync.md`](sync/pd-todoist/pd-todoist-sync.md) |
| Inbox Guardian (email filter) | [`inbox-guardian/inbox-guardian.md`](inbox-guardian/inbox-guardian.md) |
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
