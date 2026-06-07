> **Source:** [`n8n/bots/cl-bot/teams-bot.md`](https://github.com/chromelot/life-command-center/blob/main/n8n/bots/cl-bot/teams-bot.md) in the private workspace repo. Do not edit this mirror directly.

# CL Bot — Operating Manual

> Authoritative reference for the Chrome Lot Teams bot ("CL Bot"). When a future agent (or Aaron) needs to debug, extend, or operate the bot, read this first.
>
> Last refactored: 2026-04-29. Live in production.

---

## TL;DR — first thing to know

The CL Bot is **one n8n workflow** named **`Teams Bot`** (n8n workflow id `V2jeZQL8icXHpV3W`) that does everything: inbound dispatch, outbound sends, OAuth callbacks, scheduled reports. **All Code nodes share the same `$getWorkflowStaticData('global')`** — that's why everything is in one workflow instead of five small ones.

**Where the code lives:** `n8n/bots/cl-bot/deploy-teams-bot.mjs` (stub: `n8n/deploy-teams-bot.mjs`). This single file generates the entire workflow.

**Deploy a change:**

```sh
node n8n/bots/cl-bot/check-bot-codestrings.mjs   # syntax-check embedded Code-node strings
node n8n/bots/cl-bot/deploy-teams-bot.mjs         # PUT the workflow into n8n
```

**Manual fire of any workflow path:**

```sh
# Inbound dispatch (simulating a Teams message — rarely useful, JWT validation will reject)
# Outbound send to Aaron's DM
curl -X POST https://n8n.turbogear.com/webhook/teams-bot-send \
  -H 'Content-Type: application/json' \
  -d '{"text":"hello","email":"aaron@chromelot.com"}'
# Weekly Hours Report
curl -X POST https://n8n.turbogear.com/webhook/weekly-hours-report -d '{}'
```

**Tail recent executions:**

```sh
$h = @{ 'X-N8N-API-KEY' = $env:N8N_API_KEY }   # see .secrets.local
Invoke-RestMethod -Uri 'https://n8n.turbogear.com/api/v1/executions?workflowId=V2jeZQL8icXHpV3W&limit=10' -Headers $h
# Then drill into a specific execution:
Invoke-RestMethod -Uri 'https://n8n.turbogear.com/api/v1/executions/<id>?includeData=true' -Headers $h
```

---

## Architecture

```
┌─────────────────┐                                ┌──────────────────────────────────────────┐
│  Teams client   │   inbound JWT-signed HTTPS    │          n8n workflow "Teams Bot"        │
│  (Aaron + team) │ ──────────────────────────▶  │                                          │
└────────┬────────┘                                │  ┌──────────────────────────────────┐    │
         │                                         │  │  /webhook/teams-bot              │    │
         │  proactive replies                      │  │     ↓                            │    │
         │ ◀────────────────────────────────────── │  │  Dispatcher Code (route + run)   │    │
         │                                         │  └──────────────────────────────────┘    │
         │                                         │                                          │
┌────────▼────────┐                                │  ┌──────────────────────────────────┐    │
│  Azure Bot      │ ←── outbound app-only JWT ────│  │  /webhook/teams-bot-send         │    │
│  Service        │                                │  │     ↓                            │    │
│  (router)       │                                │  │  Send Code (email→convId→post)   │    │
└─────────────────┘                                │  └──────────────────────────────────┘    │
                                                   │                                          │
External callers ──────────────────────────────── │  ┌──────────────────────────────────┐    │
(daily syncs, Todoist                              │  │  /webhook/todoist-callback       │    │
 OAuth, manual fires,                              │  │  /webhook/weekly-hours-report    │    │
 future scheduled                                  │  │  Schedule: Tue 8am CT            │    │
 sends)                                            │  └──────────────────────────────────┘    │
                                                   │                                          │
                                                   │  staticData (shared across all paths):   │
                                                   │   • outbound bot token (Azure)           │
                                                   │   • JWKS cache                           │
                                                   │   • conversations.byConvId / byAadId     │
                                                   │   • people.byAadId / byEmail / byPdUserId│
                                                   │   • hubstaff.{accessToken, refreshToken} │
                                                   │   • timedoctor.{membersCache, ...}       │
                                                   │   • todoist OAuth pending states         │
                                                   │   • pd.stagesByPipeline cache            │
                                                   └──────────────────────────────────────────┘
```

**Why one workflow:** every entry point needs to read or write the same `staticData` (conversation references, people, refresh tokens). n8n's `staticData` is workflow-scoped, so splitting into multiple workflows would duplicate caches and — worse — Hubstaff's rotating refresh token would race between workflows.

---

## File layout

```
n8n/
├── conventions.md                ← Cross-bot patterns. Read before editing deploy scripts.
├── INDEX.md                      ← Domain map.
├── bots/cl-bot/
│   ├── teams-bot.md              ← THIS DOC.
│   ├── deploy-teams-bot.mjs      ← Generates + deploys the entire bot workflow.
│   ├── check-bot-codestrings.mjs ← Pre-deploy syntax check.
│   └── teams-app/
├── shared/                       ← airtable-roster.mjs, pd-delegate-*.mjs, etc.
├── .secrets.local                ← All credentials. gitignored.
└── teams-app/ (under bots/cl-bot/)
    ├── manifest.json             ← Teams App Manifest (commands, scopes, name).
    ├── package-app.mjs           ← Builds build/cl-bot.zip from manifest + icons.
    ├── color.png                 ← 192×192 color icon.
    ├── outline.png               ← 32×32 outline icon.
    └── build/
        └── cl-bot.zip            ← Sideload this into Teams to install/update the app.
```

The deploy script is **one big template-literal**. Every Code node gets:

```js
const SOMETHING_CODE = `
${SHARED_CONSTS}    // common helpers + secret constants
// ... node-specific logic ...
`;
```

`SHARED_CONSTS` includes every helper any node might want: HTTP retry wrapper, JWT parse, formatting (`ctDate`, `ctTime`, `fmtDuration`, `fmtMoney`), Pipedrive helpers, Hubstaff token rotation, Time Doctor request helpers, etc. **If you write a helper one node needs, put it in SHARED_CONSTS** — that way every other node gets it for free.

---

## Code-node inventory (inside `Teams Bot` workflow)

| Webhook / trigger | Code node | What it does |
|---|---|---|
| `POST /webhook/teams-bot` | **Dispatcher** | Validates inbound Bot Framework JWT, captures conversation reference, parses commands, routes to skill handler, posts reply. |
| `POST /webhook/teams-bot-send` | **Send** | Used by daily syncs + scheduled sends. Body: `{ text\|html, convId\|aadId\|email }` or `{ mode: "pd-follow-up-card", email, nextActivityId, ... }` for Adaptive Card follow-up capture. Optional `attachments` array for raw Bot Framework attachments. Resolves a convRef and posts via `postActivity`. |
| `GET /webhook/todoist-callback` | **Todoist OAuth Exchange** + **Todoist OAuth Respond** | Handles the redirect from Todoist OAuth, exchanges `code` for token, stores it on the user's `Person` record, replies with HTML success page. |
| Schedule: `0 8 * * 2` (Tue 8am CT) | **Weekly Hours Report** | Computes last full Mon–Sun. Pulls Hubstaff daily totals + Time Doctor worklog. Builds Markdown with risk-flag section. DMs Aaron. |
| `POST /webhook/weekly-hours-report` | **Weekly Hours Report** (same node, manual fire) | Same as above, on demand. Used for testing or ad-hoc pulls. |

Every Code node has a top-level `try/catch` so a transient failure surfaces in Teams (or in the response body for OAuth callback) rather than silently dying.

---

## Identity layer

> **Source of truth: Airtable `Admin / Payable Employees`** (`appv05uQhbcO6LAnO` / `tblU2vrGDcFTPXey9`). Schema reference: [`context/systems/airtable-roster.md`](../../../context/systems/airtable-roster.md). Single-source helpers live in [`n8n/shared/airtable-roster.mjs`](../../shared/airtable-roster.mjs) and get inlined into every Code node via `SHARED_CONSTS`.

Two stores in `staticData`:

```js
// 1. Airtable projection (refreshed every 6h, on-demand via admin refresh-roster).
//    PII-filtered: only fields in AIRTABLE_PEOPLE_ALLOWLIST land here.
staticData.peopleCache = {
  byRecordId:   { 'rec...': { recordId, fields } },
  byEmail:      { 'aaron@chromelot.com': 'rec...' },
  byAadId:      { '<azureAdObjectId>':  'rec...' },
  byPdUserId:   { '18865844':           'rec...' },
  byTodoistId:  { '49523524':           'rec...' },
  byHubstaffId: { '3332476':            'rec...' },
  refreshedAt:  <epoch ms>,
  size:         <n>,
};

// 2. Bot-only mutable state (NOT in Airtable). convRefs, OAuth tokens, etc.
staticData.peopleState = {
  byAadId: {
    '<aadId>': {
      convRef, todoistToken, todoistUserId, lastSeenAt, askUsage, ...
    }
  }
};

// 3. Legacy `staticData.people` still exists for back-compat with a few helpers
//    that haven't migrated yet (todoistToken read paths). Permissions no longer
//    read from it.
```

On every inbound DM, the dispatcher calls `refreshRosterCache(http, staticData, { force: false })` (TTL'd) and then `decoratePersonWithCategories(callerPerson)` which derives the categories list from `resolveCategories(rec.fields)` and attaches it to the in-memory person object as `person.categories`.

### Permission categories

Eight category strings, derived from Airtable on every request:

| Category | Source field |
|---|---|
| `Self` | always (any identified person) |
| `Owner` | `Position === "Owner"` (short-circuits all other categories) |
| `Sales & CS Manager` | `Sales & CS Manager` checkbox |
| `Photographer Manager` | `Photographer Manager` checkbox |
| `Account Manager` | `Position` contains "Account Manager" |
| `Has PD Seat` | `Pipedrive User ID` non-empty |
| `Authorized Delegate` | `Todoist ID` non-empty (i.e., on the shared CL Pipedrive Tasks project) |

A skill's `requires` field accepts any of:

```js
{ requires: 'any' }                                      // public
{ requires: 'Self' }                                     // identified caller
{ requires: 'Owner' }                                    // single category
{ requires: { categories: ['Owner', 'Photographer Manager'] } }   // any-of
{ requires: { categories: ['Owner', 'Sales & CS Manager'], match: 'all' } } // all-of
```

`categoryAllowed(person, requirement)` is the single gate function (used by `dispatchOne` + `handleHelp`). `Owner` short-circuits to all categories so Aaron always passes.

### Skill ↔ category map (current)

| Skill | Required categories |
|---|---|
| `help`, `ping` | any |
| `whoami`, `today`, `deals`, `pipeline`, `stale deals`, `stale activities`, `connect/disconnect todoist`, `add`, `chats here`, `permissions` | `Self` |
| `queue` | `Authorized Delegate` — delegated Todoist tasks in the shared Pipedrive project |
| `my orgs`, `plan <org> <title> [date]` | `Account Manager` OR `Authorized Delegate` — **owned** + **delegated** deals. `plan` uses pick + draft Adaptive Cards; PD write on **Schedule** only. |
| `org <name>`, `org history <name>` | `Account Manager` OR `Authorized Delegate` (portfolio-scoped) · **also** `Owner` OR `Sales & CS Manager` (org-wide search across open Sales / CS / Social Media deals via Pipedrive search) |
| `am guide` | `Self` — full AM playbook (PD↔Todoist, auto-next card, commands); auto-detects role |
| `manager guide` (`mgr guide`) | `Self` — unified manager playbook; sections filtered by `Sales & CS Manager` / `Photographer Manager` categories |
| `team status`, `weekly hours`, `schedule`, `photog roster/flags/overdue`, `mine photogs` | `Owner` OR `Photographer Manager` |
| `cs health`, `sales health`, `team pipeline`, `team gaps`, `team stale`, `delegation map`, `am status <name>`, `ar status` | `Owner` OR `Sales & CS Manager` |
| `pull`, `ask`, `team roster` | `Owner` OR `Sales & CS Manager` OR `Photographer Manager` OR `Account Manager` |
| `hiring status`, `highjobdays`, `admin *` | `Owner` |

The current mapping is also visible live via `admin skills`.

### Editing the roster

| Action | Command |
|---|---|
| Add a teammate | Airtable UI (set `Current=1`, `Name`, `Email`, `Position`) |
| Mark inactive | Airtable UI (uncheck `Current`) |
| Toggle Sales & CS Manager / Photographer Manager | `admin set-category <email> <sales-cs-manager\|photographer-manager> <on\|off>` |
| Set Pipedrive seat | `admin set-pd <email> <pdUserId>` _(writes Airtable + refreshes cache)_ |
| Set Hubstaff id | `admin set-hubstaff <email> <hubstaffUserId>` |
| Set Azure AD Object ID | `admin set-aad <email> <aadObjectId>` |
| Force a cache refresh | `admin refresh-roster` |
| Find drift between live systems | `/webhook/roster-reconcile` (or wait for Sun 6 PM CT auto-run, see "Daily and scheduled workflows") |
| Bind the shared Todoist project | `admin bootstrap-shared-project [name]` _(default name: `CL Pipedrive Tasks`)_ |

**Legacy `admin set-role` / `admin set-dept`:** still present but **no longer affect gating**. They edit the legacy `staticData.people` which nothing reads anymore. Use `admin set-category` instead.

**Impersonation:** `admin as <email> <command>` runs a command as another teammate (Owner-only). Useful for verifying their `help` view or their `permissions`.

### When a person is unknown

- Caller's AAD ID doesn't match Airtable AAD field AND email doesn't match Airtable Email → `categories = ['Self']`. They can run public + `Self` skills (`help`, `whoami`, `today`, `add`, `connect todoist`, etc.) but no manager skills.
- Sunday roster reconcile flags AAD-ID gaps so Aaron can backfill via `admin set-aad <email> <aadId>`.

---

## Skills (current registry)

> Permission column lists Airtable categories (see "Identity layer" section above for category derivation). `Owner` always passes — Aaron is the only Owner today.

| Command | Permission | What it does |
|---|---|---|
| `help`, `ping` | any | Lists commands available to caller (filtered by their categories) / health check. |
| `whoami` | Self | Shows what the bot knows about you (aadId, email, categories, Airtable record id, todoist linked?). |
| `permissions [<email>]` | Self _(Owner-only for cross-user lookup)_ | Your categories + platform IDs from Airtable. |
| `today` | Self | Tasks due today. Prefers Todoist if connected, else Pipedrive activities. |
| `deals` | Self | Owned + delegated open deals in Sales, CS, and Social Media — **grouped by pipeline** with stage and next-activity (or last-touch for CS on owned deals). |
| `deal gaps` (`gaps`) | Self | Open Sales / CS / Social Media deals you own or delegate with **no next activity** — Adaptive Card list; **Schedule** per row opens the same draft card as `plan` (+3 days default). |
| `pipeline` | Self | Your open deals grouped by stage with values. |
| `stale deals` | Self | Your deals with the **Stale** label (usually from repeated deferrals), with reasons. |
| `stale activities` | Self | Your open activities overdue >7 days. (`stale` is an alias.) |
| `add <text>` | Self | Quick-capture a task into your Todoist inbox. |
| `connect todoist` / `disconnect todoist` | Self | Manage your stored Todoist OAuth token. |
| `am guide` [`full`\|`sub`\|`promote`\|`demote`\|`exit`] | Self | Account-management playbook: PD↔Todoist sync, **auto-next + follow-up card**, CL Bot commands, recurring automations. Auto-detects path from PS run or roster. |
| `manager guide` (`mgr guide`) | Self | Manager playbook (3 messages): weekly loops, domain commands, full filtered command list. Shows Sales & CS block, Photographer ops block, or both based on roster categories. |
| `queue` | Authorized Delegate | Open tasks in the shared Pipedrive Todoist project (due now / upcoming / undated). |
| `my orgs` | Account Manager OR Authorized Delegate | Owned + delegated deals **grouped by pipeline** (Sales / CS / Social Media), tagged owned/delegated. |
| `org <name>` | Account Manager OR Authorized Delegate | Quick deal status: pipeline, stage, next/last activity (owned or delegated). |
| `org history <name>` | AM/Delegate (portfolio) OR Owner/Sales & CS Manager (org-wide) | Call-prep: open + recent completed activities, recent deal notes. Multiple matches → **Adaptive Card** picker. |
| `plan <org> <activity title> [date]` | Owner / Sales & CS Manager (org-wide) OR AM / Delegate (portfolio) | Scored match → **pick card** if ambiguous → **draft card** (deal owner + sub-delegate when delegated; title/notes/date/type). LLM parse when configured. **PD write only on Schedule submit.** Documented in `am guide`. |
| `chats here` | any | Echoes the current conversation's `convId` (for adding to scheduled-send registry). |
| `team status` | Owner OR Photographer Manager | Live "who's tracking time right now" across Hubstaff + Time Doctor with merged dedup. |
| `weekly hours` | Owner OR Photographer Manager | Rolling 7-day Hubstaff + Time Doctor hours (on-demand; scheduled Tue report uses last full week). |
| `schedule` | Owner OR Photographer Manager | Today's Knack jobs: count, estimated cars (`field_613`), assigned vs unassigned photographers. |
| `photog roster` / `photog flags` / `photog overdue` / `mine photogs` | Owner OR Photographer Manager | Photographer performance views (Adaptive Cards). |
| `cs health` | Owner OR Sales & CS Manager | CS pipeline snapshot. Flags deals overdue for the 60-day proactive policy + at-risk stages. |
| `sales health` | Owner OR Sales & CS Manager | Sales pipeline snapshot: by stage, by owner, deals missing `next_activity_date`. |
| `team pipeline` | Owner OR Sales & CS Manager | Org-wide open deals in Sales / CS / Social Media — by stage and by owner per pipeline. |
| `team gaps` | Owner OR Sales & CS Manager | Org-wide deals missing next activity — Adaptive Card; **Schedule** per row (+3d default, same draft flow as `plan`). |
| `team stale` | Owner OR Sales & CS Manager | Org-wide stale-labeled Sales + CS deals and overdue activities (>7d) by PD seat holder. |
| `delegation map` | Owner OR Sales & CS Manager | Sub AM email assignments on open deals (from `delegationIndex`). |
| `am status <name\|email>` | Owner OR Sales & CS Manager | Quick portfolio snapshot for an AM (PD seat) or sub-delegate (delegated deal count + gaps). |
| `ar status` | Owner OR Sales & CS Manager | A/R report: outstanding, aging buckets, top customers, escalations (Adaptive Card). |
| `pull <name\|email>` | Owner OR any manager | 1:1 prep dump for one person (Hubstaff hours + PD deals/activities). Fuzzy first-name match. |
| `ask <question>` | Owner OR any manager | LLM-powered Knack agent. GPT-4o-mini with 5 tools. Posts a `Working on it...` ack first, final answer 5-30s later. Daily quota 30/user. |
| `team roster` | Owner OR any manager | Live Airtable roster grouped by category. |
| `hiring status` | Owner | Separate **Hiring** Knack app: open positions, job ads + posting health, applications/comments in last ~4 days. |
| `highjobdays` | Owner | Current customers with High Job Days > 7 (`object_2.field_1035` display). |
| `admin refresh-roster` | Owner | Force-refresh `staticData.peopleCache` from Airtable. |
| `admin set-pd <email> <pdUserId>` | Owner | Set Pipedrive User ID (writes Airtable + cache). |
| `admin set-hubstaff <email> <hubstaffUserId>` | Owner | Set Hubstaff User ID (writes Airtable + cache). |
| `admin set-aad <email> <aadObjectId>` | Owner | Set Azure AD Object ID (writes Airtable + cache). |
| `admin set-category <email> <category> <on\|off>` | Owner | Toggle `sales-cs-manager` / `photographer-manager` / `account-manager` / `owner`. |
| `admin invite-todoist <email>` | Owner | DM a teammate the Todoist OAuth link (requires their convRef captured first). |
| `admin bootstrap-shared-project [name]` | Owner | Bind the `CL Pipedrive Tasks` shared Todoist project to `staticData.sharedProject`. |
| `admin shared-project` | Owner | Show currently-bound shared project. |
| `admin list` / `admin perms` | Owner | Legacy `staticData.people` views. |
| `admin skills` | Owner | Every skill grouped by category requirement. |
| `admin chats list` | Owner | Inventory of every captured conversation reference (DM, group chat, channel). |
| `admin ask-stats [days=7]` | Owner | Per-user `ask` usage and estimated $ cost over the last N days. |
| `admin add-person <email> [name]` / `remove-person` | Owner | Legacy stub-people management — superseded by Airtable but still works. |
| `admin set-role` / `set-dept` / `set-email` | Owner | **Legacy no-ops for gating** — kept for back-compat; permissions are Airtable-only. |
| `admin as <email> <command>` | Owner | Execute any command as another user (testing / coverage). |

The dispatcher splits multi-line input on newlines and runs each line as a separate command, then concatenates replies. Useful for batch admin operations.

In group chats / channels, the bot **only responds when @mentioned**. The dispatcher detects `conversationType === 'groupChat' | 'channel'` and bails unless there's an `<at>CL Bot</at>` in the text.

---

## Adding a new skill — minimum diff

In `n8n/bots/cl-bot/deploy-teams-bot.mjs`:

1. **Add a route** in the `ROUTES` array near the dispatcher:
   ```js
   { match: 'metrics', requires: 'self', handler: handleMetrics },
   ```
2. **Write the handler** somewhere in the dispatcher's Code body:
   ```js
   async function handleMetrics(ctx) {
     // ctx = { http, staticData, person, args, activity }
     // return a markdown string
     return '**Metrics**\\n- something';
   }
   ```
3. **Add to the manifest** (`n8n/teams-app/manifest.json` `commandLists`) if you want it to autocomplete in Teams. **Hard cap: 10 commands per scope.** Bump the `version` (e.g. `1.5.2 → 1.5.3`), then `node teams-app/package-app.mjs` to repackage `cl-bot.zip`, then re-upload in Teams Admin Center.
4. **Update help** (`handleHelp`) — it generates the listing automatically based on the registry, but if your skill takes args, add a hint there.
5. `node check-bot-codestrings.mjs && node deploy-teams-bot.mjs`.

If your skill is slow (>8s), don't worry — Teams' "10s ack" budget is enforced at the Bot Framework layer. The dispatcher posts the reply via `postActivity` after the work is done. As long as the work fits in n8n's webhook timeout (~120s), it's fine.

---

## Ask agent (LLM-powered Knack queries)

The `ask` skill is a tool-using LLM agent. Architecture:

- **Model**: GPT-4o-mini (configurable via `OPENAI_MODEL` in `.secrets.local`).
- **Loop**: pure JS in the dispatcher Code node — no external libs. Max 8 tool calls per question. 60s hard timeout. `max_tokens=800` per OpenAI call.
- **Pacing**: handler posts an immediate `_Working on it..._` ack, then the final reply 5-30s later via a second `postActivity`.
- **Permissions**: `Owner` OR any manager category (Sales & CS Manager / Photographer Manager / Account Manager). Daily quota 30 calls/user (raise via the constant `ASK_DAILY_QUOTA`).
- **Cost tracking**: each call's prompt/completion tokens go into `staticData.askUsage[aadId][YYYY-MM-DD]`. Aaron sees totals via `admin ask-stats [days=7]`.

### Schema awareness

The agent's system prompt embeds the **narrative reference** from `context/systems/knack-fields.md` (single source of truth — Aaron edits there, not in the bot). The deploy script reads that file and injects it.

The **machine-readable schema** lives in `n8n/knack-schema.json`, generated by `node n8n/explore-knack-schema.mjs` against the live Knack API. It's loaded into the dispatcher Code node as `KNACK_SCHEMA` and exposed to the agent via the `knack_lookup_schema` tool. Re-run the explorer when Knack adds/renames fields, then redeploy.

**Hiring Knack (separate app):** Chrome Lot operations data uses `KNACK_APP_ID` / `KNACK_API_KEY`. The hiring pipeline app uses **`KNACK_HIRING_APP_ID`** and **`KNACK_HIRING_API_KEY`** (same Builder → API & Code screen). Those power the read-only `hiring status` skill via `knackHiringRequest` — they do not change the `ask` agent schema, which stays on the main Knack app.

### Tools

| Tool | Args | Purpose |
|---|---|---|
| `knack_lookup_schema` | `object_key?` | Returns object index (no arg) or full field list for one object. |
| `knack_search` | `object_key, filters?, search_text?, fields?, sort_field?, sort_order?, limit?` | Generic record search. Limit capped at 50. |
| `knack_jobs_for_customer` | `customer, since?, until?, limit?` | Photography jobs for a dealership in a date range. |
| `knack_late_invoices` | `customer?, days_overdue?, limit?` | Invoices with status=Late, optionally per-customer. |
| `knack_customer_standards` | `customer` | Full customer record dump. Surfaces image-typed fields as Teams attachments. |

### Adding a new Knack tool

1. Add a `{ type: 'function', function: { name, description, parameters }}` entry to `ASK_TOOL_DEFS` in the dispatcher Code body. Keep the description tight — it's part of the prompt every call.
2. Implement `tool_<name>(ctx, args)` returning a JS object. Use `knackRequest`, `knackFiltersToQs`, `knackResolveCustomer`, `stripKnackHtml` helpers.
3. Register in `TOOLS_BY_NAME`.
4. If the tool returns image URLs, push to `ctx._collectedAttachments` — `handleAsk` attaches them to the final reply.
5. `node check-bot-codestrings.mjs && node deploy-teams-bot.mjs`.

### Debugging

- The agent posts errors inline (`⚠️ OpenAI call failed: ...`). Quota and timeout messages are user-friendly.
- Per-execution: `Invoke-RestMethod -Uri 'https://n8n.turbogear.com/api/v1/executions/<id>?includeData=true'` and grep for `"text":  "ask`.
- Token cost going up unexpectedly? Run `admin ask-stats 30`.
- Unexpected `Cannot resolve customer "X"`? Knack's customer table uses `field_6` for the name; if the customer lives under a different field on this app, edit `knackResolveCustomer` to widen the search.

### Cost guard

Pricing constants live in `estimateCostCents`. As of 2026-04: `gpt-4o-mini` $0.15/$0.60 per Mtok. Update the function if OpenAI changes pricing.

---

## Sending a message from outside the bot

Any n8n workflow, script, or external service can DM Aaron (or any captured user) by hitting `/webhook/teams-bot-send`:

```js
POST https://n8n.turbogear.com/webhook/teams-bot-send
Content-Type: application/json

{
  "text": "**Daily summary**\n- ...",
  "email": "aaron@chromelot.com"          // resolves to that person's DM
  // OR
  // "convId": "19:abc...@thread.v2"       // for group chats
  // OR
  // "aadId":  "<azureAdObjectId>"         // direct AAD lookup
}

// PD follow-up capture (Adaptive Card) — used by Todoist Events after auto-next:
{
  "mode": "pd-follow-up-card",
  "email": "ran@chromelot.com",
  "nextActivityId": 12345,
  "completedActivityId": 12340,
  "dealId": 218,
  "dealTitle": "Earth Motorcars CS",
  "completedSubject": "Quarterly check-in",
  "defaultNextSubject": "Follow-up: Quarterly check-in",
  "defaultDueDate": "2026-06-08"
}
```

Card submit is handled by the inbound dispatcher (`activity.value.action === "pdFollowUpSubmit"`): **what happened** → completed activity `note`; **next action** → follow-up subject; **notes for next time** → follow-up activity `note`; **when** → follow-up due date. Mirror follows follow-up subject/date via PD webhook.

The bot resolves the convRef from `staticData.conversations`. If the recipient has never DM'd the bot, the lookup fails — they need to DM the bot once first to capture their conversation reference.

For group chat / channel sends, capture the convId by `@mention`ing the bot in that chat with `chats here`. The bot replies with the convId; add it to `context/archive/teams-chats.md` for future reference.

---

## External integrations

| System | Auth | Where it's stored | Notes |
|---|---|---|---|
| **Bot Framework** (outbound) | App-only OAuth (Client Credentials) | `TEAMS_BOT_APP_ID` / `TEAMS_BOT_APP_SECRET` / `TEAMS_BOT_TENANT_ID` in `.secrets.local`. Access token cached in `staticData.outboundToken` for 1h. | App ID `68b86187-62c1-4e9d-9f1f-cfa24878e943`. |
| **Bot Framework** (inbound JWT) | Validate JWT against `https://login.botframework.com/v1/.well-known/keys` JWKS | JWKS cached in `staticData.jwks` for 24h. **Claims-only validation** (iss/aud/exp) — n8n sandbox doesn't allow `crypto`. We accept that; webhook path obscurity is the additional defense. | Don't ever use `eval` or external JWT libs in the Code node. |
| **Pipedrive** | API token | `PIPEDRIVE_ADMIN_TOKEN` in `.secrets.local`. Single token used for everything (no per-user PD OAuth). | Pipelines: 1=Sales, 6=CS, 12=Not Actively Working. CS deals follow 60-day proactive contact rule. |
| **Todoist (admin)** | API token | `TODOIST_ADMIN_TOKEN`. Used by daily morning/evening syncs. | |
| **Todoist (per-user)** | OAuth (auth code flow) | Token stored on each `Person` record (`person.todoistToken`). Triggered by `connect todoist`. | Redirect URI `https://n8n.turbogear.com/webhook/todoist-callback` must be whitelisted in the Todoist app config. |
| **Hubstaff** | OAuth refresh token (rotates on every refresh) | Seed in `HUBSTAFF_REFRESH_TOKEN`. Live token cached in `staticData.hubstaff.{accessToken, refreshToken, expiresAt}`. | **Refresh tokens rotate** — only one workflow can refresh at a time. That's a major reason all bot logic stays in one workflow. |
| **Time Doctor 2** | Long-lived JWT (~6 month lifetime) | `TIMEDOCTOR_TOKEN` + `TIMEDOCTOR_COMPANY_ID`. Re-mint manually every ~6 months. | Re-mint: `POST https://api2.timedoctor.com/api/1.0/login {email, password, permissions:'read'}`. Company ID stays the same. |

---

## Daily and scheduled workflows that USE the bot

These are separate n8n workflows that POST to `teams-bot-send` for delivery:

### Team-wide PD ↔ Todoist sync (active)

Detailed architecture → [`pd-todoist-sync.md`](pd-todoist-sync.md). Legacy v1 morning/evening batch sync removed 2026-06-05.

| Workflow | Schedule | Manual webhook | What it does |
|---|---|---|---|
| `PD Activity Mirror` | _real-time, PD webhook_ | `/webhook/pd-activity-event` | Mirror PD activity create/update/complete/delete to shared `Pipedrive` Todoist project. Idempotent reconciler. |
| `Todoist Events` | _real-time, Todoist webhook_ | `/webhook/todoist-events` | `item:completed` → mark PD done + auto-next activity on any open deal (`+3d` default) + follow-up Adaptive Card DM. `note:added` → post as PD Note on linked deal. |
| `Team Evening Defer` | `0 17 * * 1-6` (5pm CT, Mon–Sat; closed Sun) | `/webhook/team-evening-defer` | Defer open activities due ≤ today. Tomorrow, but never Fri/Sat/Sun (snaps to Monday). No Teams DMs. |
| `Team Morning DM` | _(retired 2026-06-06)_ | — | Deactivated. Use `today`, `deal gaps`, `stale deals` in CL Bot. |
| `Team PD Reconciler` | `0 4 * * 1-5` (4am CT, Mon-Fri) | `/webhook/team-pd-reconcile` | Drift safety net: heal anything webhooks missed. Aaron-only DM with drift counts. |

All PD ↔ Todoist workflows have top-level `try/catch` that emits a `❌ ... failed` Teams message on crash. All use the patched `makeRetryingHttp` that handles 503s correctly across n8n's IPC boundary.

Internal scheduled sends inside the `Teams Bot` workflow itself (so they share `staticData`):

| Trigger | What it does |
|---|---|
| Schedule `0 8 * * 2` (Tue 8am CT) → `Weekly Hours Report` Code node | Aggregate last full Mon–Sun across Hubstaff + Time Doctor. Flag low-activity people. DM Aaron. |

When adding a new scheduled report that needs HS or TD data, **add it inside the `Teams Bot` workflow** (don't create a new workflow) — see `Weekly Hours Report` as a template.

---

## Common errors and how to debug

### "ctDate is not defined" (or `fmtDuration`, `ctTime`, etc.)

You wrote a new Code node that tries to use a helper that's defined in the **dispatcher** body but not in `SHARED_CONSTS`. Fix: move the helper into `SHARED_CONSTS`. All shared formatting helpers belong there.

### Workflow runs for ~3 seconds and errors with "Request failed with status code 503"

This is the **IPC error-stripping** bug. n8n's Code-node task runner serializes errors across an IPC boundary, which strips `e.response`. Our retry helper used to read `e.response?.status` and got `undefined`, so it bailed instead of retrying.

**Fixed in both deploy scripts as of 2026-04-29.** The fix uses a `statusOf(e)` helper that also parses the message regex (`/status code (\d{3})/`) and checks `e.statusCode` / `e.httpCode`. If you see this again on a new HTTP path, make sure you're calling through `http()` (the wrapped helper), not raw `this.helpers.httpRequest`.

### Webhook fires but no execution shows up in n8n

n8n's webhook returns `{ message: "Workflow was started" }` when the trigger isn't connected to anything, OR when the workflow is paused. Sanity checks:

```sh
# Confirm workflow is active
Invoke-RestMethod -Uri 'https://n8n.turbogear.com/api/v1/workflows/V2jeZQL8icXHpV3W' -Headers $h | Select-Object active

# List nodes — make sure the webhook node is connected to a downstream node
... | Select-Object -ExpandProperty nodes | Format-Table name, type
```

If a node is in the workflow but not connected, `connections:` in the deploy script's `teamsBotWorkflow()` is missing the wire.

### `deploy-teams-bot.mjs` fails with `413 Request Entity Too Large`

nginx on `n8n.turbogear.com` caps request bodies around **1 MB**. The Teams Bot workflow JSON is larger when every Code node is replaced in one PUT. The deploy script now **merges into the live workflow** (preserves node ids) and sends a slim PUT body; if that still 413s, it **falls back to a Dispatcher-only patch** (enough for command/handler changes).

If you added a **new webhook node** or changed **connections**, bump `client_max_body_size` on the nginx vhost (recommend `4m`) and re-run the full deploy. Until then, wire new nodes manually in the n8n UI.

### Dispatcher fails with `missing /` or bogus `SyntaxError` on valid JS

n8n’s JS task-runner can concatenate its VM **preamble and your Code on one line**, which confuses V8 when parsing some **`/regex/` literals** (especially character classes like `[a-z]` or `[^>]`). The error text is often unhelpful (`missing /`, `Unexpected token`).

**Mitigations in `deploy-teams-bot.mjs`:** leading **blank line** at the top of `SHARED_CONSTS` and `DISPATCHER_CODE`; **no `/[...]/` regex literals** in shared helpers (`escapeHtml`, `stripKnackHtml`, `ctOffset` GMT parse, `photogEscCell`, `photogAmKey`, `photogNumField`, `splitInboundLines`, Bearer strip, `admin as` match, `findPeople` / `handleAdmin` tokenizing); use **`new RegExp('…')`** only when the pattern is entirely inside a string, or plain loops/`split`/`join`. Details: [n8n-io/n8n#26081](https://github.com/n8n-io/n8n/issues/26081).

### Bot replies "Unknown command: `CL Bot foo`"

The dispatcher's HTML stripping isn't removing the `@mention` properly. Check: the dispatcher should call the strip-mentions logic before parsing the command. If a Teams `<at>CL Bot</at>` ends up in the parsed command text, route matching fails.

### Manifest upload fails "command list exceeded 10 entries"

Teams' personal scope has a hard 10-command limit. Trim `manifest.json` `commandLists` for `personal` scope to ≤ 10. Bump version (e.g., `1.5.2 → 1.5.3`), repackage with `node teams-app/package-app.mjs`, re-upload.

### Hubstaff calls fail with 401 days/weeks after a working state

The rotating refresh token expired or was invalidated by another consumer. Mint a fresh one (Hubstaff developer console → your app → regenerate token), update `HUBSTAFF_REFRESH_TOKEN` in `.secrets.local`, redeploy. The bot will auto-pick up the new seed.

### `today` / `add` returns "status code 401"

The user's stored Todoist OAuth token has been revoked (Todoist app permissions panel, account password reset, etc.). As of 2026-04-29 the bot detects this automatically: `callTodoist` re-throws a tagged `TODOIST_AUTH_EXPIRED` error, the affected handler wipes the dead token, and the user gets `⚠️ Your Todoist connection expired. Run \`connect todoist\` to re-link.` `today` additionally falls back to Pipedrive in the same reply.

If a user reports the raw `status code 401` with this code path, the deploy is older than the fix — redeploy the bot. Manually verify a stored token is dead with:

```pwsh
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$h = @{ Authorization = 'Bearer <storedToken>' }
Invoke-WebRequest -Uri 'https://api.todoist.com/api/v1/tasks?limit=1' -Headers $h -UseBasicParsing
```

### Time Doctor calls fail with 401

The 6-month JWT expired. Re-mint:

```pwsh
$body = @{ email='aaron@chromelot.com'; password='<password>'; permissions='read' } | ConvertTo-Json
Invoke-RestMethod -Uri 'https://api2.timedoctor.com/api/1.0/login' -Method POST -Body $body -ContentType 'application/json'
# Copy the .data.token field into TIMEDOCTOR_TOKEN in .secrets.local
```

Then `node deploy-teams-bot.mjs`.

### "No conversation reference for ..." when sending to a user

That user has never DM'd the bot. Tell them to send the bot any message (or `whoami`) once. The dispatcher captures their convRef on first contact.

For group chats / channels: someone needs to `@CL Bot chats here` once in that chat. The convId then shows up in `admin chats list` and can be referenced for scheduled sends.

---

## How to read execution logs

```pwsh
# Recent executions (with duration)
$h = @{ 'X-N8N-API-KEY' = '<key from .secrets.local>'; Accept = 'application/json' }
$execs = Invoke-RestMethod -Uri 'https://n8n.turbogear.com/api/v1/executions?workflowId=V2jeZQL8icXHpV3W&limit=15' -Headers $h
$execs.data | Select-Object id, status, startedAt, @{n='dur_sec';e={[Math]::Round(((Get-Date $_.stoppedAt) - (Get-Date $_.startedAt)).TotalSeconds, 1)}} | Format-Table

# Drill into one execution (saves to file because it's huge)
Invoke-RestMethod -Uri 'https://n8n.turbogear.com/api/v1/executions/<id>?includeData=true' -Headers $h | ConvertTo-Json -Depth 12 | Out-File output/exec-<id>.json
# Then grep:
rg '"error":|"message":|"text":' output/exec-<id>.json
```

A successful Weekly Hours run takes ~2-4 seconds. A successful Dispatcher reply takes <1 second. Anything that looks too short (<0.5s) for a multi-step workflow probably hit the catch block early.

---

## Glossary

- **`aadId`** — Azure Active Directory Object ID. Globally unique per user in the Chrome Lot tenant. The primary key for `Person` records.
- **`convId`** — Bot Framework conversation ID. For DMs it looks like `a:1u8...`; for group chats it ends in `@thread.v2`. The key into `staticData.conversations.byConvId`.
- **`convRef`** — Conversation Reference object: `{ serviceUrl, channelId, conversation, user, bot, tenantId }`. Required to send a proactive (bot-initiated) message.
- **Skill** — One row in the `ROUTES` registry + one handler function. Skills receive a `ctx` object with `{ http, staticData, person, args, activity }`.
- **Stub Person** — A `Person` record with `aadId = 'stub:<email>'`. Created by `admin add-person` before the user has ever DM'd the bot. Auto-merged into the real AAD ID record on first DM.
- **`SHARED_CONSTS`** — Big template literal at the top of `deploy-teams-bot.mjs` that's prepended to every Code-node body. Helpers placed here are available to every node.
- **`staticData`** — n8n's per-workflow persistent key-value store. `$getWorkflowStaticData('global')`. Survives across executions of the same workflow.
- **Inline reply** — Dispatcher posts the response within the same handler invocation. Default for all current skills.
- **Proactive send** — A bot-initiated message via `/webhook/teams-bot-send`, not in response to an inbound message.
- **CL Bot vs LCC Bot** — Renamed `LCC Bot → CL Bot` on 2026-04-29. Some old strings may still reference LCC Bot in stale screenshots; ignore them.

---

## Capacity / hygiene rules baked into the bot

- **Pipedrive 60-day proactive policy on CS deals.** `cs health`, `deals`, and the morning sync all enforce: a CS deal whose `last_activity_date` (or `add_time` if never touched) is 60+ days old gets flagged. Sales deals still need a `next_activity_date` scheduled.
- **Hubstaff/Time Doctor risk thresholds.** Weekly Hours Report flags anyone <25h in the prior week. `0h` is always escalated as 🛑. Tunable in `WEEKLY_HOURS_CODE` constants.
- **Group chat opt-in.** In group chats / channels, the bot only responds when `@mentioned`. Auto-replying to every group message would be spammy and confuses non-bot conversations.

---

## Status

- **Multi-user identity layer:** live.
- **All current skills:** live.
- **Daily syncs (morning/evening):** live, on resilient retry helper.
- **Weekly Hours Report:** live, schedules Tue 8am CT.
- **Todoist OAuth:** live (Aaron has connected; others can self-connect via `connect todoist`).
- **Hubstaff + Time Doctor:** live, both contributing to `team status` and `Weekly Hours Report`.
- **Group chats + scheduled sends:** topology in place. Two convIds captured (Account Management + Scheduling) — see `context/archive/teams-chats.md`.
- **Pending:** Zapier → n8n migration. (Started planning, paused per Aaron's request 2026-04-29.)

---

## Pointers

- **Spec deltas / decisions log:** Cursor agent transcripts under `agent-transcripts/`. Search for "CL Bot", "team status", "weekly hours" to find rationale for prior changes.
- **Adjacent context:** start at `context/router.md` and `context/AGENTS.md` (workspace root). Most-used pointers for bot-related work:
  - `context/systems/mcp-servers.md` — full MCP inventory and the `teams` MCP details.
  - `context/systems/notion-databases.md` — every DB ID across Personal + Business Notion.
  - `context/people/index.md` — team roster, Hubstaff / Pipedrive IDs, performance notes.
  - `context/archive/teams-chats.md` — captured group chat convIds (was `context/teams-chats.md`).
  - `context/rules.md`, `context/systems/capacity-rules.md`, `context/systems/cadences.md` — broader operating principles.
- **n8n workflow IDs** (for `Invoke-RestMethod` against the n8n API):
  - `V2jeZQL8icXHpV3W` — Teams Bot
  - `maHE1hghVsE1hGvy` — PD Activity Mirror
  - `sRKv15LudXv7U2ts` — Todoist Events
  - `Eo5bJynRwtGC3KnG` — Team Evening Defer
  - `TAg3bwtvjysAyIci` — Team Morning DM
  - `7Rgm9BwSh9rjtNbl` — Team PD Reconciler
