> **Source:** [`n8n/sync/pd-todoist/pd-todoist-sync.md`](https://github.com/chromelot/life-command-center/blob/main/n8n/sync/pd-todoist/pd-todoist-sync.md) in the private workspace repo. Do not edit this mirror directly.

# PD ↔ Todoist real-time sync

**Status:** Active (team-wide, real-time). Legacy v1 `Morning Pipedrive Sync` / `Evening Pipedrive Close` removed 2026-06-05.

Webhook-driven and team-wide: everyone with a Pipedrive seat (and Todoist-only sub-AMs delegated via the org's `Sub Account Manager Email` field) sees their PD activities mirrored into the shared **Pipedrive** Todoist project (`6gfPq74rf2fJwGJx`, Chrome Lot workspace) the moment they're created in PD.

## Architecture

```
              ┌─────────────────────────────┐
              │       Pipedrive             │
              │  (activities, deals, orgs)  │
              └──┬───────────────────────┬──┘
                 │ activity.{added,       │ PUT /activities/{id}
                 │   updated, deleted}    │ POST /activities (auto-next)
                 │     webhook            │ POST /notes
                 ▼                        ▲
        ┌────────────────────┐            │
        │ PD Activity Mirror │            │  ┌────────────────────┐
        │   (n8n workflow)   │            │  │  Todoist Events    │
        │                    │            └──┤   (n8n workflow)   │
        │ • create mirror    │               │                    │
        │ • update mirror    │               │ • complete → PD    │
        │ • complete mirror  │               │ • uncomplete → PD  │
        │ • delete mirror    │               │ • notes → PD Note  │
        └─────────┬──────────┘               │ • auto-next on     │
                  │                          │   completion       │
                  ▼                          └─────────▲──────────┘
       ┌─────────────────────────────────┐             │
       │  Shared "Pipedrive" Todoist     │             │
       │  project (6gfPq74rf2fJwGJx)     │── events ───┘
       │                                 │  webhook
       │  Each task description has:     │
       │    Pipedrive activity ID: <n>   │
       │    Owner: <email>               │
       │    Assignee: <email>            │
       └─────────────────────────────────┘
                  ▲
                  │ daily/weekly safety nets
                  │
       ┌──────────┴──────────────────────┐
       │                                 │
   ┌───────────────┐                         ┌──────────────┐
   │ Team Evening  │                         │ Team PD      │
   │ Defer (5pm)   │                         │ Reconciler   │
   │               │                         │ (4am)        │
   │ Defer not-done│                         │ Drift heal   │
   │ activities to │                         │ (missed      │
   │ tomorrow.     │                         │  webhooks)   │
   │ PD webhook    │                         │              │
   │ updates       │                         │              │
   │ mirrors.      │                         │              │
   │ (no Teams DM) │                         │              │
   └───────────────┘                         └──────────────┘
```

## Workflows

| Workflow | n8n ID | Trigger | What it does |
|---|---|---|---|
| `PD Activity Mirror` | `maHE1hghVsE1hGvy` | `POST /webhook/pd-activity-event` (real-time, from PD) | Idempotent reconciler: per event, looks up the live PD activity, computes the desired Todoist mirror state in the shared project, creates / updates / completes / deletes accordingly. Resolves assignee via Sub AM Email → Airtable → Todoist ID. |
| `Todoist Events` | `sRKv15LudXv7U2ts` | `POST /webhook/todoist-events` (real-time, from Todoist) | `item:completed` → marks PD done + creates auto-next activity on **any open deal** (`+3d` default) + DMs completer a Teams Adaptive Card (`pd-follow-up-card`) to capture outcome and refine the follow-up subject/due date. **PD-side completion** (activity already done when mirror closes Todoist): waits 60s and skips auto-next + card if the deal already has another open activity (covers completing in PD and immediately scheduling the next activity). Todoist-side completion is unchanged (immediate auto-next). `item:uncompleted` → reopen PD. `item:updated` → if the Todoist due date differs from PD's, push it to PD (`pdPut /activities/{id}`). Echo events from PD-driven Todoist updates no-op naturally because dates already match. Cleared due dates are ignored (PD's date stays). `note:added`/`note:updated` → posts comment as a PD Note on the linked deal. **Task mirrors** (description `Notion Dev Project: <uuid>`, any Todoist project): `item:completed` → Notion `Status=Done`; `item:uncompleted` → `In progress`; `item:updated` → sync `Due Date`. |
| `Team Evening Defer` | `Eo5bJynRwtGC3KnG` | Cron `0 17 * * 1-6` (Mon–Sat CT, **closed Sunday**) + `POST /webhook/team-evening-defer` | For every PD seat owner: defer not-done open activities (due ≤ today). Target is next calendar day, but **never Fri/Sat/Sun** — those snap to **Monday** (so Thu/Fri/Sat 5pm runs all land on Monday). PD `activity.update` webhooks refresh Todoist mirrors. Post-pass: rollup max deferral per deal → set/clear deal field **`Stale`** at **7+** (pipelines 1 + 6). **No Teams DMs** (retired 2026-06-06). |
| `Team Morning DM` | `TAg3bwtvjysAyIci` | _(deactivated)_ | **Retired 2026-06-06.** Use CL Bot `today`, `deal gaps`, `stale deals` on demand. |
| `Team PD Reconciler` | `7Rgm9BwSh9rjtNbl` | Cron `0 4 * * 1-5` + `POST /webhook/team-pd-reconcile` | Belt-and-suspenders: diffs every PD seat owner's open activities (within 30d) against the shared Todoist project. Creates missing mirrors, closes mirrors for activities now done, deletes mirrors for activities deleted in PD. Also syncs **Delegated** deal label on open deals (Sub AM set → add label; cleared → remove — merges with existing labels, never replaces). Bounded at 50 mutations per category per run. Optional Aaron DM (`RECONCILER_DM_ENABLED` in deploy script). |
| `PD Deal Delegate Label` | _(deploy to resolve id)_ | `POST /webhook/pd-deal-event` (real-time, from PD) | On `deal.added` / `deal.updated`: if deal has Sub AM Email → add **Delegated** label + set **Sub-Delegate Name** from primary email; if both emails cleared → remove **Delegated** + clear name. Label writes merge comma-separated IDs so **Stale** and other chips are preserved. Echo events no-op when already correct. |

## Constants

| Concept | Value |
|---|---|
| Shared Todoist project | `6gfPq74rf2fJwGJx` (`Pipedrive`, Chrome Lot workspace) |
| Sub AM field keys | `pd-delegate-fields.json` — **deal only** (Sub AM Email 1 + 2 + Sub-Delegate Name). Org/person fields are legacy/unused. |
| Delegated label option id | `pd-delegate-fields.json` → `delegatedLabelOptionId`. Setup: `node n8n/setup-pd-delegate-label.mjs` after creating **Delegated** in PD deal labels. |
| PD link parser | `/pipedrive activity id:\s*(\d+)/i` (in mirror task description) |
| CS stage IDs | `26` AT RISK / `28` Needs Attention / `61` Happy |
| Auto-next default | `+3d` on every pipeline; completer refines via follow-up Adaptive Card |
| Reconciler future window | 30 days |
| Reconciler heal cap | 50 per category per run |
| Failure DMs | All workflows DM Aaron via `teams-bot-send` on uncaught errors |

## Mirror task shape

Every mirror task in the shared project has:

- **Title**: `<subject> | <org-or-deal>` — clean, no prefixes.
- **Labels**: `['stop']` if PD activity type is `stop`, else `[]`. Stop activities are filterable in Todoist via `@stop`.
- **Description**:
  ```
  Deferrals: <n>
  [Manual deferrals: <m>]
  Pipedrive activity ID: <id>
  Owner: <pd_owner_email>
  Assignee: <effective_email> [(delegated via deal Sub AM)] [CC: secondary@...]
  ```
  `Deferrals:` is intentionally the **first line** so the count shows in Todoist's list-view description preview without opening the task.
- **Deferral tracking**: Pipedrive activities have no custom fields, so `Team Evening Defer` increments a counter stored in activity **`note`** (`Deferrals: N` / `Manual deferrals: M`). Subject/title stays clean. The mirror applies labels `deferred-1` / `deferred-2` / `deferred-3` (3+ uses `deferred-3`) and shows counts in the Todoist description. Only the 5pm defer job increments auto deferrals — manual date edits increment `Manual deferrals`. Completing an activity resets the count naturally (auto-next creates a fresh activity). Filter stuck items in Todoist with `@deferred-3`. Legacy `[×N]` subject prefixes are still read but stripped on next defer/mirror pass.
- **Project**: shared Pipedrive project
- **Due date**: matches PD `due_date` exactly (timezone in CT)
- **Assignee**: Todoist user ID of the resolved assignee (if known + collaborator); else unassigned.
- **Assigner** (delegated only): PD activity owner's Todoist ID → `assigned_by_uid` on create/update. This is the same mechanism as assigning someone in Todoist UI — the owner gets native completion/comment notifications, no extra tasks.

### Field-name gotcha

Todoist API v1 is asymmetric: the **write** field for assigning is `assignee_id`, but the same value comes back on **read** as `responsible_uid`. The assigner writes as `assigned_by_uid` and reads the same. Sending `responsible_uid` in a POST body is silently ignored. All sync code uses `assignee_id` / `assigned_by_uid` for writes and inspects `responsible_uid` / `assigned_by_uid` on reads.

## Project collaborators

`assignee_id` only sticks on workspace projects when the target user is on the project's collaborator list. "Team-visible" is necessary but **not sufficient** — each user must be explicitly shared in.

Bootstrap script:

```pwsh
node n8n/share-shared-project.mjs
```

This is idempotent. It shares with every `Current=true` Airtable row that has a **PD seat** or **Todoist ID**, plus the canonical Todoist-only delegate roster in `pd-delegate-resolution.mjs`.

Also runs nightly inside `Team PD Reconciler` (A6.1). Run manually when onboarding a new delegate before the next 4am pass.

## Assignee resolution

For each PD activity (`pd-delegate-resolution.mjs`):

1. Read Sub AM Email 1 + 2 from the activity's **deal** only (activity must be deal-attached)
2. Primary delegate email → mirror assignee; Sub AM 2 → `[CC: ...]` in description
3. If deal has no Sub AM → PD activity owner's email
4. Airtable `Todoist ID` lookup → `assignee_id` on mirror
5. **Delegated only:** PD activity owner's Todoist ID → `assigned_by_uid` (native Todoist notifications to owner on complete/comment)
6. If unmapped → mirror created unassigned; reconciler heals assignee/assigner drift nightly (A6.2)

## Failure modes + recovery

| Symptom | Cause | Recovery |
|---|---|---|
| New PD activity doesn't appear in Todoist | PD webhook not firing | Check PD Settings → Webhooks → "Last delivery". Re-trigger with reconciler `/webhook/team-pd-reconcile`. |
| Mirror exists but Todoist completion doesn't update PD | Todoist app webhook URL wrong, or user hasn't auth'd the bot's Todoist app | Reconciler at 4am will close the mirror; PD activity stays open. Check Todoist Developer console webhook config. |
| Wrong assignee on mirror | Sub AM Email wrong on org, or Airtable Todoist ID wrong | Fix in PD or Airtable; next webhook event will rewrite the mirror. Force fast: edit then re-save the activity in PD. |
| New hire's PD activities mirror but stay unassigned | They're not yet a collaborator on the shared Todoist project | `node n8n/share-shared-project.mjs` (idempotent). Then bump activity in PD to retrigger mirror. |
| Per-user DM didn't land | Recipient hasn't DM'd the bot once (no convRef captured) | Have them send any message to the bot in Teams once. Then `admin invite-todoist <email>` won't error either. |
| Mirror stuck after PD activity deleted | Webhook missed | Reconciler at 4am will delete it. Manual: `/webhook/team-pd-reconcile`. |

## Observability

- Every workflow's Code node returns a JSON summary; visible in n8n Execution log.
- Failures DM Aaron via teams-bot-send.
- Reconciler nightly DM is the single most important signal — if drift counts are growing day over day, webhooks are dropping events somewhere.

## Setup / maintenance runbook

### 1. Configure Pipedrive webhook subscription (Aaron, in PD UI)

- Settings → Tools and apps → Webhooks → Add new webhook
- **Endpoint URL**: `https://n8n.turbogear.com/webhook/pd-activity-event`
- **Event action**: `*` (all)
- **Event object**: `activity`
- **User**: All users
- **HTTP Auth**: none (the n8n endpoint is unauthenticated; URL kept private)
- **Version**: v2 (this is what PD's UI offers now; the handler is v2-native and also accepts v1 shapes for compatibility)

Save. PD will send a test request — the workflow is still disabled, so it'll get stored in n8n's webhook queue but not processed. That's fine.

### 2. Configure Todoist app webhook (Aaron, in Todoist Developer Console)

- https://developer.todoist.com/appconsole.html
- Open the existing app used for the bot's OAuth flow (the one whose client ID matches `TODOIST_OAUTH_CLIENT_ID` in `.secrets.local`)
- Webhooks tab:
  - **Webhook callback URL**: `https://n8n.turbogear.com/webhook/todoist-events`
  - **Watched events**: `item:completed`, `item:uncompleted`, `item:updated`, `note:added`, `note:updated`

Save.

### 3. Dry-run the backfill

```pwsh
cd "c:\Users\Aaron\Resilio Sync\Life Command Center\n8n"
node backfill-pd-mirrors.mjs
```

This prints what would happen without writing. Review:

- How many tasks will be ripped from Aaron's legacy Office + Errands projects
- How many candidate PD activities will be mirrored into the shared project
- Any "unmapped assignees" warnings (these are emails we have PD seats for but no Todoist ID in Airtable)

### 4. Run the backfill for real

```pwsh
node backfill-pd-mirrors.mjs --execute
```

This:
1. Deletes every PD-linked Todoist task from Aaron's old Office + Errands projects
2. Creates fresh mirrors in the shared Pipedrive project for every open PD activity due ≤ +30d, assignee resolved as above

### 5. Activate the new workflows

In n8n UI, toggle ON for each:

- `PD Activity Mirror` (so PD changes start mirroring)
- `Todoist Events` (so Todoist completions/comments round-trip)
- `Team Evening Defer` (5pm Mon–Sat CT; defer target never Fri/Sat/Sun — snaps to Monday; closed Sunday; no Teams DMs)
- CL Bot on demand (`today`, `deal gaps`, `stale deals`) — replaces retired 9am Morning DM
- `Team PD Reconciler` (4am M-F)

### 6. Smoke test

In Pipedrive: create a test activity (subject "Test sync 2026-05-14", due today, attached to any deal you own). Within seconds, it should appear in the shared Pipedrive Todoist project, assigned to whoever owns the org (or the Sub AM Email if set).

In Todoist: change the due date on that test task. Within seconds, the PD activity's due date should match.

Back in Pipedrive: change the due date back. Within seconds, the Todoist mirror's due date should match.

In Todoist: complete that test task. Within seconds, the PD activity should be marked done. An auto-next activity should appear on the deal (**+3 days** by default; adjust on the follow-up card if needed).

### 7. Monitor for 48h

- Daily reconciler DM at 4am — drift count should be ~0
- Failure DMs from any workflow — should be quiet

If drift accumulates or failures DM, check n8n execution logs and PD/Todoist webhook delivery logs.

## CL Bot deal ops (full AM + sub-delegate)

`my orgs`, `org`, `org history`, and `plan` work for **both** Pipedrive seat owners and Todoist-only sub-delegates. The bot unions deals you **own** (Sales / CS / Social Media) with deals where you are **Sub AM**.

| Need | Full AM | Sub-delegate (no PD seat) |
| --- | --- | --- |
| Work list | `today` or Todoist `Pipedrive` project | `queue` or `today` |
| Deal list | `my orgs` or `deals` | `my orgs` |
| Quick status | `org <name>` (picker if ambiguous) | `org <name>` |
| Call prep | `org history <name>` | `org history <name>` |
| Schedule follow-up | `plan <org> <title> [date]` — pick + draft cards; PD on **Schedule** | Same on delegated deals |
| Complete + log | Todoist complete + **comment** → PD Note | Same |
| Auto-next + card | CL Bot follow-up Adaptive Card after complete | Same (card goes to completer) |

Full playbook: `am guide` (auto-detects role).

## Future work

- **Auto-share new PD seat owners** in `Team PD Reconciler`: nightly idempotent share of the shared project to every Airtable PD seat owner, eliminating the need to run `share-shared-project.mjs` manually after a new hire.
- **Self-heal stale assignees/assigners** in `Team PD Reconciler`: on each pass, diff `assignee_id` and (when delegated) `assigned_by_uid` against desired values and update mismatches.
- **Fold `sync-todoist-roster.mjs --fix` into the nightly reconciler** so Airtable's `Todoist ID` field is always correct without a manual run.
- **Completion follow-up card** (C4, live): when auto-next is created, `Todoist Events` POSTs `teams-bot-send` with `mode: "pd-follow-up-card"`. Completer gets an Adaptive Card: what happened this time (completed activity `note`), next action title, notes for next time (follow-up activity `note`), and due date. Submit updates both activities + mirror picks up follow-up subject/date changes.
- **Sales coaching packet** + **live interrogation**: Aaron's manager-tier asks (Phase D).
- **Customer rollup** recurring DM (cadence TBD).
- **Hardening**: optional HTTP Basic Auth on PD webhook subscription (kept off in V1; URL is private).

## See also

- [`account-management/roadmap.md`](../../account-management/roadmap.md) — strategy + status view for the broader account-management arc this fits into
- [`context/systems/airtable-roster.md`](../../../context/systems/airtable-roster.md) — canonical roster + permission categories
- [`bots/cl-bot/teams-bot.md`](../../bots/cl-bot/teams-bot.md) — outbound DM mechanism (`teams-bot-send`) used by these workflows
- `deploy-pd-activity-mirror.mjs`, `deploy-todoist-events.mjs`, `deploy-pd-todoist-schedules.mjs`, `backfill-pd-mirrors.mjs` — source
- `check-pd-todoist-codestrings.mjs` — syntax validator for all four
- `share-shared-project.mjs` — idempotent bootstrap to add Airtable PD seat owners as Todoist collaborators
- `sync-todoist-roster.mjs` — diff Airtable's `Admin / Payable Employees` against Todoist's Chrome Lot workspace and report drift; pass `--fix` to auto-correct safe cases (wrong / missing Todoist ID, missing email). `stale_account` (account no longer in workspace) is always flagged for human review since it could mean either a removed teammate or an email change.
- `remove-legacy-pd-sync.mjs` — delete legacy v1 workflows from n8n (idempotent)
