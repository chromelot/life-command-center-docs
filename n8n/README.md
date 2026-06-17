> **Source:** [`n8n/README.md`](https://github.com/chromelot/life-command-center/blob/main/n8n/README.md) in the private workspace repo. Do not edit this mirror directly.

# n8n Workflows

Workflows live on `https://n8n.turbogear.com` (DigitalOcean-hosted). Deploy scripts in this folder are idempotent.

## Read first

1. **[`conventions.md`](conventions.md)** — cross-bot patterns (Teams formatting, deploy, permissions, cards)
2. **[`INDEX.md`](INDEX.md)** — domain map and canonical deploy paths
3. **Domain spec** — e.g. [`bots/cl-bot/teams-bot.md`](bots/cl-bot/teams-bot.md) or [`sync/pd-todoist/pd-todoist-sync.md`](sync/pd-todoist/pd-todoist-sync.md)

## Workflows

| File / source | Trigger | Purpose |
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

## Deploy scripts

| Script | What it deploys |
|---|---|
| `bots/cl-bot/deploy-teams-bot.mjs` | Entire **Teams Bot** workflow |
| `bots/cl-bot/check-bot-codestrings.mjs` | Pre-deploy syntax check (CL Bot) |
| `sync/pd-todoist/deploy-pd-activity-mirror.mjs` | PD Activity Mirror |
| `sync/pd-todoist/deploy-todoist-events.mjs` | Todoist Events |
| `sync/pd-todoist/deploy-pd-todoist-schedules.mjs` | Evening Defer + Reconciler (Morning DM retired) |
| `sync/pd-todoist/check-pd-todoist-codestrings.mjs` | Pre-deploy syntax check (PD sync) |
| `inbox-guardian/deploy-inbox-guardian.mjs` | Inbox Guardian (`--dry-run` validates, `--enforce` arms) |
| `tools/explore-knack-schema.mjs` | Regenerate `knack-schema.json` |

## Quick deploy

```sh
# CL Bot
node n8n/bots/cl-bot/check-bot-codestrings.mjs
node n8n/bots/cl-bot/deploy-teams-bot.mjs

# PD ↔ Todoist
node n8n/sync/pd-todoist/check-pd-todoist-codestrings.mjs
node n8n/sync/pd-todoist/deploy-pd-activity-mirror.mjs
node n8n/sync/pd-todoist/deploy-todoist-events.mjs
node n8n/sync/pd-todoist/deploy-pd-todoist-schedules.mjs
```

Legacy stub paths at `n8n/deploy-*.mjs` still forward to canonical locations — see [`INDEX.md`](INDEX.md).

## Secrets

`n8n/.secrets.local` (gitignored). Deploy scripts read and inject as workflow constants.

## See also

- [`account-management/README.md`](account-management/README.md) — AM stack map
- `context/systems/operations-catalog.md` — control-plane hub
