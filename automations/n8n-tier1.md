> **Source:** [`context/systems/notion-guides/n8n-automations.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/notion-guides/n8n-automations.md) in the private workspace repo. Do not edit this mirror directly.

# n8n Tier 1 Automations

> **Scheduled and webhook automations** on `https://n8n.turbogear.com` (DigitalOcean). These run without Aaron in the loop. Interactive planning sessions are **Tier 2 — Cursor** (see Workflow guides sub-pages).

---

## Two-tier model

| Tier | Where | Examples |
|---|---|---|
| **Tier 1 — n8n** | DigitalOcean, cron + webhooks | PD↔Todoist sync, CL Bot, weekly metrics, 1:1 reminders |
| **Tier 2 — Cursor** | lcc-hub, phrase-triggered | Weekly plan, monthly plan, micro-scrub, 1:1 prep |

**Agent read order for bot work:** `n8n/conventions.md` → `n8n/INDEX.md` → domain spec.

---

## Workflow registry

| Source | Trigger | Purpose |
|---|---|---|
| `deploy-teams-bot.mjs` → **CL Bot** | Webhooks + Tue 8am CT | Teams bot — dispatch, send, OAuth, Weekly Hours |
| `deploy-pd-activity-mirror.mjs` | Webhook `pd-activity-event` | Real-time PD → Todoist mirror |
| `deploy-todoist-events.mjs` | Webhook `todoist-events` | Todoist → PD: complete, notes, auto-next |
| `deploy-pd-todoist-schedules.mjs` | Cron + webhooks | 5pm defer (silent), 4am reconciler |
| `deploy-pd-deal-delegate-label.mjs` | Webhook `pd-deal-event` | Delegated label on deals |
| `deploy-am-setup-verifier.mjs` | Webhook | Process Street AM setup verification |
| `specs/weekly-metrics.md` | Sun 8pm ET | Weekly metrics → Google Sheets |
| `specs/one-on-one-reminders.md` | Daily 8am ET | Overdue 1:1 reminders |
| `specs/health-data-sync.md` | Webhook | Health data → Notion |

Stub paths at `n8n/deploy-*.mjs` still forward to canonical subfolders.

---

## Scheduled Teams messages (what still DMs)

| Automation | Schedule | Recipient | Notes |
|---|---|---|---|
| Weekly Hours Report | Tue 8am CT | Aaron | Inside CL Bot workflow |
| Follow-up Adaptive Card | On Todoist complete | Completer | Interactive card, not a summary DM |
| Reconciler summary | 4am Mon–Fri | Aaron | **Off by default** (`RECONCILER_DM_ENABLED=false`) |
| Workflow failure alerts | On uncaught error | Aaron | `❌ <Workflow> failed` |

**Removed 2026-06-06:** 9am Team Morning DM, 5pm Evening Defer per-user and aggregate DMs.

---

## Deploy checklist

1. Edit deploy script in `n8n/bots/cl-bot/` or `n8n/sync/pd-todoist/`
2. Run domain codestrings check
3. Deploy from **lcc-hub** (agent runs commands)
4. Workflow auto-reactivates; smoke one webhook or command

```sh
# CL Bot
node n8n/bots/cl-bot/check-bot-codestrings.mjs
node n8n/deploy-teams-bot.mjs

# PD ↔ Todoist
node n8n/sync/pd-todoist/check-pd-todoist-codestrings.mjs
node n8n/deploy-pd-todoist-schedules.mjs
```

**Secrets:** `n8n/.secrets.local` (gitignored) — injected as workflow constants on deploy.

---

## Conventions (don't skip)

- **Single CL Bot workflow** — all paths share `staticData` (conv refs, roster, Hubstaff token)
- **Only sanctioned Teams send:** `POST /webhook/teams-bot-send`
- **Teams formatting:** `-` bullets only; no pipe tables; disambiguation cards use radio buttons (`expanded`)
- **CRM writes from chat:** draft Adaptive Card → user confirms → then write
- **Code-node rules:** leading blank line; no `/regex/` literals in embedded strings

Full reference: `n8n/conventions.md`

---

## Domain map (`n8n/INDEX.md`)

| Folder | Contents |
|---|---|
| `bots/cl-bot/` | Teams bot deploy + manual |
| `sync/pd-todoist/` | PD↔Todoist deploy scripts + spec |
| `sync/am-setup/` | AM onboarding verifier |
| `shared/` | airtable-roster, pd-delegate-*, field JSON |
| `account-management/` | AM strategy roadmap |
| `specs/` | Cron-only markdown specs |
| `tools/` | One-shot maintenance CLIs |

---

## Keeping Notion in sync

After changing workflows, skills, or n8n specs:

```sh
node scripts/generate-ops-catalog.mjs
node scripts/publish-ops-catalog-to-notion.mjs
```

`OpsCatalogSync` scheduled task on lcc-hub (Sun 8:15 PM CT) can run the same pair.
