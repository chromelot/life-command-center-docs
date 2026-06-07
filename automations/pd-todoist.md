> **Source:** [`context/systems/notion-guides/pd-todoist-automation.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/notion-guides/pd-todoist-automation.md) in the private workspace repo. Do not edit this mirror directly.

# PD ↔ Todoist Automations

> **Real-time sync** between Pipedrive (CRM) and the shared **Pipedrive** Todoist project. Account managers execute in Todoist; Pipedrive stays the system of record. CL Bot is the chat layer on top.

**Status:** Live, team-wide. Legacy Aaron-only morning/evening batch sync removed 2026-06-05.

**Canonical spec:** `n8n/sync/pd-todoist/pd-todoist-sync.md`

---

## Architecture at a glance

```
Pipedrive ──webhook──▶ PD Activity Mirror ──▶ Shared Todoist project
     ▲                                              │
     │                                              │ webhook
     └──────── Todoist Events ◀─────────────────────┘
              (complete, notes, due-date, auto-next)
```

**Shared project:** `6gfPq74rf2fJwGJx` (Chrome Lot Todoist workspace, name `Pipedrive`)

---

## Real-time workflows

| Workflow | Trigger | What happens |
|---|---|---|
| **PD Activity Mirror** | PD activity webhook | Create / update / complete / delete mirror task. Resolves assignee via Sub AM Email → Airtable → Todoist ID. |
| **Todoist Events** | Todoist webhook | Complete → PD done + **auto-next** (+3 days) + follow-up Adaptive Card to completer. Notes → deal Note. Due-date edits → PD reschedule. |
| **PD Deal Delegate Label** | PD deal webhook | Sub AM set → **Delegated** label + Sub-Delegate Name on deal. |

---

## Scheduled safety nets

| Workflow | Schedule | Teams DMs? | What it does |
|---|---|---|---|
| **Team Evening Defer** | 5pm Mon–Sat CT (closed Sun) | **None** | Defer open activities due ≤ today to next business day (never Fri/Sat/Sun → snaps to Monday). Roll up **Stale** deal label at 7+ deferrals. |
| **Team PD Reconciler** | 4am Mon–Fri CT | Optional | Drift heal: create missing mirrors, close/delete stale mirrors, heal delegate fields. `RECONCILER_DM_ENABLED=false` in deploy script = silent (default). |
| ~~Team Morning DM~~ | ~~9am Mon–Fri~~ | **Retired** | Deactivated 2026-06-06. Use CL Bot `today`, `deal gaps`, `stale deals`. |

---

## Policy defaults

| Topic | Rule |
|---|---|
| **Auto-next** | +3 days on every pipeline after Todoist completion; refine via follow-up card |
| **Deferrals** | 5pm job increments `Deferrals: N` in activity note; Todoist labels `@deferred-1` … `@deferred-3` |
| **Stale deals** | 7+ deferrals → Pipedrive **Stale** label on Sales + CS deals (no stage moves) |
| **CS proactive** | 60+ days without touch flagged in `today` / `cs health` (not auto-DM) |
| **Delegation** | Sub AM Email on deal routes mirror assignee; PD owner unchanged |

---

## Daily workflow for AMs (no scheduled DMs)

1. **`queue`** or **`today`** in CL Bot — see what's due
2. Work tasks in **Todoist** (complete + comment logs the touch)
3. **`deal gaps`** / **`stale deals`** when planning — schedule via draft card
4. **`org <name>`** / **`plan`** for ad-hoc follow-ups

Evening defer still runs silently — mirror due dates update when activities roll forward.

---

## Deploy

```sh
node n8n/sync/pd-todoist/check-pd-todoist-codestrings.mjs
node n8n/deploy-pd-activity-mirror.mjs
node n8n/deploy-todoist-events.mjs
node n8n/deploy-pd-todoist-schedules.mjs
node n8n/deploy-pd-deal-delegate-label.mjs
```

Manual test webhooks:
- `POST /webhook/team-evening-defer`
- `POST /webhook/team-pd-reconcile`

---

## See also

- **Chrome Lot Bot** — commands AMs use day-to-day
- **Account management roadmap** — `n8n/account-management/roadmap.md`
- **Airtable roster** — `context/systems/airtable-roster.md`
