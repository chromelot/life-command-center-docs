> **Source:** [`context/systems/mcp-servers.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/mcp-servers.md) in the private workspace repo. Do not edit this mirror directly.

# MCP Servers

Inventory of every MCP server configured in this workspace, plus how to choose between them.

## Quick selection guide

| If you need to... | Call MCP |
|---|---|
| Create or move a Todoist task | `todoist` |
| Read or update a Notion personal-side DB (Tasks, Projects, Workouts, Values, etc.) | `notion-personal` |
| Read or update a Notion work-side DB (CL Tasks, CL Projects, TG Features, Team) | `notion` |
| Look up or create a Pipedrive deal/activity/contact | `pipedrive` |
| Read Knack customer/photographer/job records | `knack` |
| Get or send a Gmail message; read/write Calendar; read/write Drive | `google` |
| Send SMS or check Quo call transcripts | `quo` |
| Run a Process Street workflow check, manage runs/tasks | `processstreet` (API-key MCP) |
| Send a Teams message via the CL Bot, look up convIds | `teams` |
| Read Hubstaff hours / activity | `hubstaff` |
| Pull body comp + watch metrics in one call | `health-data` |
| Skin condition context (logs, meds snapshot, Drive index) | `skin-care` |
| List Make scenarios / pull blueprints for n8n migration | `make` |
| Read exported Zapier JSON for n8n migration | `zapier-export` |
| List/read Paperform forms, submissions, products, webhooks, Papersign | `paperform` |
| Read / write Turbo Gear MongoDB | `mongodb` |
| Look up GitHub PRs / commits | `github` |
| Pull Airtable data | `airtable` |

## Servers

### todoist — primary daily task driver

- **Token (personal workspace)**: `45703e887d680663d4fdd69689876c5f5fc0ddb9` — MCP + Cursor agent **default** for Aaron's personal tasks
- **Token (Chrome Lot admin)**: `TODOIST_ADMIN_TOKEN` in `n8n/.secrets.local` — team workspace; used by n8n PD↔Todoist sync and CL Bot. **Cursor agents must not use this token for personal tasks.**
- **Tools**: `get_task`, `get_tasks`, `get_completed_tasks`, `create_task`, `update_task`, `complete_task`, `reopen_task`, `delete_task`, `get_projects`, `create_project`, `add_comment`
- **Critical exclusion**: `Shopping List` (`6W36wRPXj8qC2RCc`, personal) is excluded from all planning workflows. Aaron uses it at the store only.

#### Two workspaces — do not mix

| Workspace | ID | Who uses it | Cursor agent default? |
|---|---|---|---|
| **Personal** | (none on project) | Aaron's life, habits, health, personal admin, dev mirrors | **Yes** — always default here |
| **Chrome Lot** | `172492` | Shared team execution — PD mirrors, CL recurring ops | **No** — double confirmation required (`context/rules.md`) |

**Anti-pattern:** Personal habits (skin log, gym, journal, etc.) must **never** use Chrome Lot projects such as `Recurring` or `Pipedrive`. Personal workspace has **no** `Recurring` project — if an agent picks `Recurring` (`6VrfxRV66H7GwQQg`), that is Chrome Lot workspace.

#### Personal workspace project IDs

| Project | ID | Notes |
|---|---|---|
| Inbox | `6VRWV4VxrMr285jv` | Quick capture, triage daily |
| Physical Warm Up & Spirit | `6fMXPxXq8X2XQwcM` | Morning routine |
| Morning Setup | `6Vq243QXJjHvJ2mM` | Morning routine |
| Gym Work | `6fMXQmM8PrwWFqfc` | Workout tracking |
| Office | `6VpxxqrxF655M5QQ` | Personal desk/admin work, health tracking mirrors |
| Errands | `6VmM476xRh7pPjHj` | Personal errands (not PD mirror destination) |
| Evening | `6VrPwCwfr5QmcwX2` | Evening routine |
| Pending Aaron | `6g3hXfFffjP8RxVH` | Waiting on Aaron |
| Shopping List | `6W36wRPXj8qC2RCc` | **EXCLUDED from planning workflows** |
| Lists | `6gxrCgXHWqr5M4JG` | Reference lists |
| Download | `6gXV7PWwWfqXHm8C` | Download queue |

#### Chrome Lot workspace project IDs (`172492`)

| Project | ID | Notes |
|---|---|---|
| Pipedrive | `6gfPq74rf2fJwGJx` | Shared — all team PD activity mirrors land here |
| Recurring | `6VrfxRV66H7GwQQg` | **CL team recurring ops only** — not for personal habits |
| Customer Projects | `6VpXPGr3F574xMwc` | CL customer work |
| Fake Demos | `6Vm7xh838xXRGVWm` | CL demo tasks |
| Sales Packets | `6cQpR3qQHrj5F49p` | CL sales prep |
| Same Day To Do | `6VmQrCh6h5X7FfFh` | Urgent same-day CL items |
| Project Tracker | `6Vm7x4mHwgqrqVJp` | CL project tracker |
| Random | `6Vm7cjxM2xVXGC36` | CL misc |

### pipedrive — Chrome Lot CRM

Domain: chromelot. Sales Pipeline 1, CS Pipeline 6, Dormant Pipeline 12. Full configuration → [pipedrive.md](pipedrive.md).

### notion-personal — Personal Notion

- **Token**: `secrets/notion-personal.json` (not in repo)
- **Role**: Projects, milestones, inboxes, meeting logs, journals, time-block databases, health data
- **Schemas**: → [notion-databases.md](notion-databases.md)

### notion — Business / Work Notion

- **Token**: local secrets only (not in repo)
- **Role**: CL tasks, CL projects, TG features, team performance records, recurring task assignments
- **Schemas**: → [notion-databases.md](notion-databases.md)

### processstreet — API-key MCP v2.0 (active in Cursor)

- **Path**: `C:/Users/Aaron/.cursor/mcp-servers/processstreet-mcp/dist/index.js` in [`.cursor/mcp.json`](../../.cursor/mcp.json)
- **Auth**: `PROCESSSTREET_API_KEY` (org Integrations → API key). Works in Cursor without OAuth.
- **Rebuild after edits**: `cd C:\Users\Aaron\.cursor\mcp-servers\processstreet-mcp; npm run build` then reload MCP in Cursor.
- **Tool count**: ~44 tools (Cursor may cap at ~40 per session — highest-value tools load first).

#### Run operations (16 tools)

List/get templates, create/list/get/stop runs, list/get/complete/update run tasks, comments, webhooks.

#### Authoring operations (28 tools)

| Area | Tools |
|------|-------|
| Folders | `list_folders`, `create_folder`, `update_folder`, `delete_folder` |
| Templates | `create_workflow`, `duplicate_workflow`, `delete_workflow` |
| Revisions | `list_revisions`, `get_draft_revision`, `create_draft_revision`, `publish_revision` |
| Tasks (draft) | `list_revision_tasks`, `get_revision_task`, `create_revision_task`, `delete_revision_task` |
| Widgets | `list_task_widgets`, `create_task_widget`, `update_task_widget`, `delete_task_widget` |
| Logic | `list_logic_rules`, `create_logic_rule`, `delete_logic_rule` |
| Other | `get_workflow_permissions`, `list_data_sets`, `list/create/delete_scheduled_workflows`, `raw_api_request` |

**Authoring pattern**: edits target a **Draft** revision — `get_draft_revision` → add tasks/widgets → `publish_revision` when ready.

**API gaps** (use UI or `raw_api_request` experiments): `PUT /workflows/{id}`, `PUT` revision tasks, checklist authoring, pages/data-set create, assignment/due-date rules.

- **Production writes**: require explicit Aaron approval before execution.
- **Destructive ops — case-by-case approval every time** (never batch, never "cleanup" without asking): `delete_workflow`, `delete_folder`, `delete_revision_task`, `delete_task_widget`, `delete_logic_rule`, `delete_scheduled_workflow`, `delete_webhook`, `publish_revision`. **Do not call these to probe or test the API.** Read-only GET/list tools are fine without approval.
- **Key templates**: New Account Manager Setup (`oPFeGVUGtOquv2GPwkNPQg`, **draft v1.0** — publish pending review), Daily Closeout (`ub2zoqkxITISOM4RNAZALw`), Daily Operations Check-In, Daily Customer Notifications, Daily Helpdesk Clearing, Daily Hiring Follow Up, Daily Operations Support Checklist, Afternoon Photographer Check In, Checking Customers with Missing Cars, Customers with High Job Days Report, Customer Standards Onboarding, Customer Exit Checklist, New Hire Onboarding (`g7V9_B1K7z7rqgLS8Q1GEg`)

#### Official MCP — `https://mcp.process.st/mcp` (not used in Cursor yet)

Process Street's remote MCP (~105 tools, workflow authoring) requires **OAuth login**, not the API key. URL-only config in `mcp.json` fails with **"Missing Authentication Token"** because:

1. The `/mcp` endpoint rejects `X-API-KEY` (verified — returns that exact error).
2. Cursor must complete an OAuth browser sign-in; Claude/ChatGPT document this as a connector flow, not a bare URL.
3. Known Cursor bug: servers with OAuth discovery may ignore `headers` even if you paste the API key there.

**To retry official later**: Cursor Settings → MCP → add `process-street` with `"url": "https://mcp.process.st/mcp"` and use any **Connect / Sign in** button Cursor offers. Not supported if org uses SAML SSO. Until OAuth works in Cursor, keep `processstreet` (stdio) above.

### google — Gmail, Calendar, Drive, Sheets

- **Capabilities**: List/create/update/delete events, Gmail send/read, Drive read/list/share, Sheets append/update
- **Three calendars**:
  - **Work**: default / `aaron@chromelot.com` — business meetings, client calls
  - **Personal**: `hoegenauera@gmail.com` — personal appointments, social, medical
  - **Time Blocks**: `10283d615faeb91862fc0ccd8f3ac216c7299a58f2196185e912be8f3e3cbe83@group.calendar.google.com` — recurring daily structure (Morning Routine, Business Development, Lift, Plano Work, Field Work, Free Time)
- **Always pull all three calendars** when checking schedule. Merge and label `[Work]` / `[Personal]` / `[Time Block]`.
- **Before creating any event**: search both calendars for existing events at the same time or with similar subject. If a match exists, confirm with Aaron whether to modify the existing event or skip. Never create duplicates.

### quo — phone / SMS

- **Role**: Mobile capture channel
- **Capabilities**: Send/receive SMS, call transcripts, AI call summaries

### knack — legacy Chrome Lot ops DB

- **App ID**: `5d1621582487c7000af3055e`
- **Role**: Customers, jobs, photographers, invoices. Being replaced by Turbo Gear.
- **Field reference**: → [knack-fields.md](knack-fields.md)

### teams — Microsoft Teams + CL Bot

- **CL Bot**: Azure Bot Service, App ID `68b86187-62c1-4e9d-9f1f-cfa24878e943`. Bidirectional. Aaron DMs it or `@mentions` it in group chats. Multi-user: every team member has a `Person` record with role/dept-based permissions.
- **Conventions**: `n8n/conventions.md` — read first for any bot/n8n work.
- **Operating manual**: `n8n/bots/cl-bot/teams-bot.md`. **CL Bot** workflow id `V2jeZQL8icXHpV3W`.
- **Deploy**: `node n8n/bots/cl-bot/check-bot-codestrings.mjs && node n8n/bots/cl-bot/deploy-teams-bot.mjs` (stubs at `n8n/deploy-teams-bot.mjs` still work).
- **Inbound**: `POST /webhook/teams-bot` (Azure routes here; JWT-validated)
- **Outbound (any caller can use)**: `POST /webhook/teams-bot-send` with `{ text|html, convId|aadId|email }`
- **Skills surface**: `help`, `today`, `deals`, `pipeline`, `stale`, `add`, `connect/disconnect todoist`, `team status`, `cs health`, `pull <person>`, `chats here`, `whoami`, plus `admin *` family
- **Captured group chats**: `archive/teams-chats.md` (convId registry)
- **Scheduled PD Teams DMs**: retired for 9am Morning DM and 5pm Evening Defer (2026-06-06). Reconciler optional DM via `RECONCILER_DM_ENABLED` in deploy script.
- **Weekly Hours Report**: Tue 8 AM CT, lives inside the `Teams Bot` workflow itself

### hubstaff — employee time tracking

- **Org**: 678423 (Chrome Lot)
- **API**: v2, base `https://api.hubstaff.com/v2`, rate limit 1000/hr
- **Auth**: rotating refresh token at `secrets/hubstaff-refresh-token.txt`. Access tokens expire in ~24-72 hours; refresh token rotates on each exchange.
- **Tools**: `hubstaff_get_organizations`, `hubstaff_get_members`, `hubstaff_get_weekly_hours`, `hubstaff_get_daily_activities`, `hubstaff_get_time_entries`, `hubstaff_get_projects`, `hubstaff_get_weekly_report`
- **Best for weekly review**: `hubstaff_get_weekly_report` with org ID — gives complete hours-per-member breakdown with names
- **Member + project IDs**: → [hubstaff.md](hubstaff.md)

### skin-care — skin condition context

- **Location**: `mcp/skin-care/` in workspace (local Node MCP)
- **Registered as**: `skin-care` in `.cursor/mcp.json`
- **Tools**: `skin_source_status`, `skin_get_context({ days })`
- **Static context**: `context/self/skin.md`
- **Architecture**: → [skin-care.md](skin-care.md)
- **Setup docs**: `mcp/skin-care/README.md`
- **Smoke test**: `cd mcp/skin-care && node test-skin.mjs`

### health-data — body comp + watch metrics

- **Location**: `mcp/health-data/` in workspace (local Node MCP)
- **Registered as**: `health-data` in `.cursor/mcp.json`
- **Tools**: `health_source_status`, `health_get_summary({ days })`, `health_get_metric({ metric, days })`, `health_get_daily({ date })`, `health_get_streaks({ metric, days })`, `health_persist_recent({ days })`
- **Architecture**: → [health-data.md](health-data.md)
- **Setup docs**: `mcp/health-data/README.md`
- **Smoke test**: `cd mcp/health-data && node test-drive.mjs`

### make — Make.com scenario introspection (n8n migration)

- **Location**: `mcp/make/` in workspace (local Node MCP, live Make API)
- **Registered as**: `make` in `.cursor/mcp.json`
- **Auth**: `secrets/make.env` via `MAKE_ENV_FILE` — token from Make Profile → API
- **Tools**: `make_status`, `make_list_organizations`, `make_list_scenarios`, `make_get_scenario`, `make_get_blueprint`, `make_summarize_scenario`, `make_list_scenario_logs`
- **Setup docs**: `mcp/make/README.md` — copy `mcp/make/make.env.example` → `secrets/make.env`
- **Smoke test**: `cd mcp/make && node test-make.mjs`
- **Read-only** — no create/run/delete in v1

### zapier-export — Zapier export file parser (n8n migration)

- **Location**: `mcp/zapier/` in workspace (file-based, no live Zapier API)
- **Registered as**: `zapier-export` in `.cursor/mcp.json`
- **Auth**: none — reads JSON from disk via `ZAPIER_EXPORT_PATH` in `secrets/zapier.env`
- **Export source**: Zapier Settings → Security and data → Download my Zap workflows (unzip if ZIP)
- **Tools**: `zapier_export_status`, `zapier_list_zaps`, `zapier_get_zap`, `zapier_summarize_zap`, `zapier_search_zaps`
- **Setup docs**: `mcp/zapier/README.md` — copy `mcp/zapier/zapier.env.example` → `secrets/zapier.env`
- **Smoke test**: `cd mcp/zapier && node test-zapier.mjs`

### paperform — forms, submissions, e-sign (Paperform + Papersign API)

- **Location**: `mcp/paperform/` in workspace (local Node MCP, live Paperform API)
- **Registered as**: `paperform` in `.cursor/mcp.json`
- **Auth**: `secrets/paperform.env` via `PAPERFORM_ENV_FILE` — API key from Paperform account settings
- **Tools**: 58 total — forms, fields, submissions, partial submissions, products, coupons, webhooks, spaces, translations, Papersign documents/folders/webhooks, plus `paperform_raw_request` escape hatch
- **Plan tiers**: Standard (read + coupons + field updates), Business (form/webhook/space/translation/product writes), Papersign (e-sign endpoints). **403 = plan doesn't include endpoint.**
- **Cursor cap**: ~40 tools may load; high-value reads registered first; use `paperform_raw_request` for anything missing
- **Setup docs**: `mcp/paperform/README.md` — copy `mcp/paperform/paperform.env.example` → `secrets/paperform.env`
- **Smoke test**: `cd mcp/paperform && node test-paperform.mjs`
- **Production writes** (delete submission, update form, webhooks, etc.): require explicit Aaron approval in Cursor sessions

### mongodb — Turbo Gear primary database

- **Role**: TG application data
- **Tools**: standard MongoDB MCP query/update tools

### github — code repositories

- **Repos active in workspace**: `turbo-gear`, `turbo-gear-api`, `turbo-gear-workspace` (under `torchcommercialmedia` org)
- **Use for**: PR review, commit history, contractor activity tracking

### airtable — secondary structured data

- (Listed in workspace MCP config; not currently a primary daily driver. Reach for it only when Aaron explicitly references Airtable.)

## Data pull scripts (deterministic, not MCPs)

Located in `scripts/` in the workspace. Use these BEFORE MCP calls when a workflow has a precomputed pull script — they handle date math in CST and parallelize API calls.

| Script | Used by |
|---|---|
| `scripts/weekly-data-pull.mjs` | Weekly meeting Phase 0 |
| `scripts/withings-sync.mjs` | Weekly / monthly / quarterly Phase 0 (writes body comp to Notion) |

**Key date formulas** (all use America/Chicago):
- `TODAY` = `new Date().toLocaleDateString('en-CA', { timeZone: 'America/Chicago' })` → YYYY-MM-DD
- `TOMORROW` = TODAY + 1 day
- `DAY_AFTER_TOMORROW` = TODAY + 2 days (used for Todoist completed-tasks `until` param, which is exclusive)
- `NINETY_DAYS_AGO` = TODAY - 90 days (Pipedrive overdue range)

**Retired scripts**: `morning-data-pull.mjs`, `evening-data-pull.mjs` were deleted when daily Cursor reviews were replaced by n8n workflows.

## MCP capabilities matrix

| Service | Read | Write | Known gaps |
|---|---|---|---|
| Knack | Records with filters, counts | Create/update/delete records | No aggregation; client-side processing required |
| Pipedrive | Deals, persons, orgs, activities | Create deals/persons/orgs, add activities/notes | Some pipeline metadata gaps |
| Todoist | Active tasks (filtered), projects, completed (by date) | Create/update/complete/reopen/delete | — |
| Google Calendar | Events by date range | Create/update/delete events | — |
| Gmail | List / read messages | Send messages, modify labels | — |
| Google Drive | List / read files | Create folders, share files | — |
| Google Sheets | Read/write cell ranges | Append rows, update values | — |
| Notion (both) | Query DB, get page, get block children | Create/update page, append blocks, archive | Block-level mutation is paginated |
| Hubstaff | Hours, activity, time entries, projects, members | None (read-only) | — |

## See also

- [../router.md](../router.md) — for any specific intent, route here first
- [notion-databases.md](notion-databases.md) — full Notion schema reference
- [pipedrive.md](pipedrive.md) — Pipedrive-specific config and quirks
- [knack-fields.md](knack-fields.md) — Knack field reference
- [hubstaff.md](hubstaff.md) — Hubstaff member + project IDs
- [health-data.md](health-data.md) — health-data MCP architecture