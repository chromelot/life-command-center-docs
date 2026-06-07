> **Source:** [`context/systems/notion-guides/cl-bot.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/notion-guides/cl-bot.md) in the private workspace repo. Do not edit this mirror directly.

# Chrome Lot Bot (CL Bot)

> **Teams chat interface** for Chrome Lot account managers, managers, and ops. One n8n workflow (`Teams Bot`, id `V2jeZQL8icXHpV3W`) handles inbound commands, outbound DMs, OAuth, and scheduled reports.

---

## What it does

- **Deal & pipeline reads** — open deals, gaps, stale flags, org history, CS/sales health
- **Scheduling** — draft Adaptive Card first, then write to Pipedrive on **Schedule** / **Submit**
- **AM onboarding** — `am guide` (3-part playbook by role), `manager guide` for ops managers
- **Follow-up capture** — after you complete a Todoist task, bot DMs an Adaptive Card to refine the auto-created next activity
- **Ops pulls** — Hubstaff hours, Knack schedule, photographer flags, A/R status, roster admin

**Canonical technical manual:** `n8n/bots/cl-bot/teams-bot.md`  
**Cross-bot patterns:** `n8n/conventions.md`

---

## How to talk to it

1. Open a **1:1 chat** with **CL Bot** in Teams (not only the group channel — DMs need a personal conv ref).
2. Type a command or natural-language skill match.
3. `help` — short list filtered to your Airtable permission categories.
4. `am guide` or `manager guide` — full role-aware playbook (3 messages).

---

## Key commands by role

### Account Managers (Pipedrive seat)

| Command | What it does |
|---|---|
| `today` | Today's activities + deal gaps + stale flags (replaces retired 9am Morning DM) |
| `deals` / `pipeline` | Open deals by stage |
| `deal gaps` | Deals missing next activity — pick card → schedule |
| `stale deals` / `stale activities` | Stale-labeled deals and chronic deferrals |
| `org <name>` / `org history <name>` | Deal status and touch history |
| `plan <org> <title> [date]` | Draft follow-up card (+3d default) |
| `my orgs` | Portfolio you own or delegate |

### Sub-delegates (Todoist only)

| Command | What it does |
|---|---|
| `queue` | Your delegated Todoist tasks |
| `my orgs` / `org <name>` | Lookup + `plan` via CL Bot |
| `am guide` | Todoist-path playbook |

### Managers (Sales & CS / Photographer)

| Command | What it does |
|---|---|
| `team gaps` / `team pipeline` | Org-wide deal health |
| `team stale` / `sales health` / `cs health` | Pipeline snapshots |
| `am status <name>` / `delegation map` | Per-AM snapshot + Sub AM coverage |
| `schedule` / `team status` / `weekly hours` | Field ops (photographer path) |
| `manager guide` | Unified manager playbook |

---

## Scheduled messages (still active)

| When | What | Who gets it |
|---|---|---|
| **Tue 8:00 AM CT** | Weekly Hours Report | Aaron — Hubstaff + Time Doctor rollup |
| **On task completion** | Follow-up Adaptive Card | Completer — refine auto-next activity |
| **On workflow failure** | `❌ <Workflow> failed` | Aaron — via `teams-bot-send` |

**Retired 2026-06-06:** 9am Morning DM and 5pm Evening Defer summary DMs. Use `today` / `deal gaps` on demand instead.

---

## Permissions

Source: **Airtable** `Admin / Payable Employees` → cached in bot `staticData.peopleCache`.

| Category | Typical access |
|---|---|
| `Account Manager` + PD seat | Full AM command set |
| `Authorized Delegate` | `queue`, `org`, `plan` (Todoist path) |
| `Sales & CS Manager` | Org-wide reads + `plan` draft flow |
| `Photographer Manager` | Schedule, hours, photog flags |
| `Owner` | Everything + admin commands |

Separate manager checkboxes stay separate — unified `manager guide` docs only.

---

## Deploy & debug

```sh
node n8n/bots/cl-bot/check-bot-codestrings.mjs
node n8n/deploy-teams-bot.mjs
```

- **Code:** `n8n/bots/cl-bot/deploy-teams-bot.mjs` (single template generates whole workflow)
- **Outbound send:** `POST /webhook/teams-bot-send` with `{ text, email }`
- **Smoke:** `ping`, one read command, one card flow (Schedule → Cancel)

---

## See also

- **PD ↔ Todoist Automations** (Notion sub-page) — mirrors and auto-next behind the bot
- **n8n Tier 1 Automations** — full registry of webhook + cron workflows
