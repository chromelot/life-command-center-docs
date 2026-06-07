> **Source:** [`n8n/conventions.md`](https://github.com/chromelot/life-command-center/blob/main/n8n/conventions.md) in the private workspace repo. Do not edit this mirror directly.

# n8n automation conventions

> Cross-cutting patterns for CL Bot, PD↔Todoist sync, and any future Teams/n8n automations. **Read this before editing deploy scripts.** Domain specifics live in [`INDEX.md`](INDEX.md) → domain spec files.

---

## Architecture invariants

1. **Single `Teams Bot` workflow** (`V2jeZQL8icXHpV3W`) — all inbound dispatch, outbound send, OAuth, and scheduled reports share `$getWorkflowStaticData('global')`. Do not split Hubstaff token cache, roster cache, or conv refs across workflows.
2. **Only sanctioned Teams send path:** `POST https://n8n.turbogear.com/webhook/teams-bot-send` with `{ text|html, email|aadId|convId }`. No parallel Graph send nodes.
3. **Failures must be visible** — top-level `try/catch` in scheduled/webhook Code nodes should DM Aaron via `teams-bot-send` with `❌ <Workflow> failed.` Silent failures are unacceptable.
4. **New bot features** → new webhook or schedule entry point on the **same** `Teams Bot` workflow (see Weekly Hours Report as template).
5. **New integrations needing HS/TD/roster** → add a Code node inside `Teams Bot`, not a separate workflow, unless there is no shared state need.

---

## Teams message formatting (Aaron preferences)

Teams renders bot markdown inconsistently. Follow these rules in every handler return string and guide builder.

| Do | Don't |
|---|---|
| Markdown `-` bullet lists | `•` bullets, pipe `\|` tables, dense prose walls |
| `amGuideCommandRows` pattern: `- description — \`command\`` | Table layouts in bot text |
| `{ guideParts: [part1, part2, part3] }` for long guides | One giant message |
| 3-part guide: **(1)** workflow + common tasks, **(2)** automations/background, **(3)** filtered command list | Mixed cheat-sheet in one blob |
| Role-aware sections filtered by Airtable categories | One-size-fits-all when roles differ |
| `postActivityChunked` when text > ~10 KB | Single post → MessageSizeTooBig |

**Guide helpers** (in `deploy-teams-bot.mjs`): `amGuideH1`, `amGuideH2`, `amGuideKvRows`, `amGuideCommandRows`, `amGuidePart`, `amGuideFooter`. Return `{ guideParts: parts }` from handler; dispatcher posts each part as a separate activity.

**Live in-bot docs:** `am guide`, `manager guide` — re-runnable anytime; Process Street manuals get `output/*-paste.md` for manual paste only (never script-overwrite published PS content).

---

## Adaptive Cards

| Pattern | Rule |
|---|---|
| Disambiguation (org pick, etc.) | `Input.ChoiceSet` with `style: 'expanded'` (radio buttons) |
| User choice | Never `style: 'compact'` (dropdown) for deal/org disambiguation |
| CRM writes | **Draft card first** → user taps **Schedule** / **Submit** → then write |
| Scheduling | Reuse `buildPlanDraftReply` / `gapPlanOpen` — +3d default for gap rows |
| Attachments | Return `{ text, attachments: [adaptiveCardAttachment(card)] }` from handler |

---

## Permission + SKILLS pattern

- **Source of truth:** Airtable `Admin / Payable Employees` → `resolveCategories()` in [`shared/airtable-roster.mjs`](shared/airtable-roster.mjs).
- **Gates:** `SKILLS` registry entries use `requires: 'Self'`, `'Owner'`, or `{ categories: [...] }`. `categoryAllowed(person, requires)` at dispatch.
- **Separate checkboxes stay separate** — `Sales & CS Manager` and `Photographer Manager` are distinct gates. Unified *documentation* (`manager guide`) is fine; merging permission categories requires explicit Aaron approval.
- **Owner** short-circuits all category checks.
- **Help** (`handleHelp`) stays short — point to `manager guide` / `am guide` for depth; filter listing by caller categories.

---

## Automation policy defaults

- **Auto-next:** `+3 days` on every pipeline after Todoist task completion. User refines via follow-up Adaptive Card DM.
- **Manager org-wide commands:** read + schedule-only (`team gaps`, `team pipeline`, etc.) — no new write paths beyond existing `plan` draft flow.
- **Cursor-session writes:** Pipedrive / Todoist / Knack / Notion production writes need explicit Aaron approval. Tier 1 n8n syncs are exempt.

---

## Deploy checklist

```sh
# CL Bot (from repo root or via stub at n8n/deploy-teams-bot.mjs)
node n8n/bots/cl-bot/check-bot-codestrings.mjs
node n8n/deploy-teams-bot.mjs

# PD ↔ Todoist stack
node n8n/sync/pd-todoist/check-pd-todoist-codestrings.mjs
node n8n/deploy-pd-activity-mirror.mjs   # stubs at n8n/ root still work
```

1. Run codestrings check for the domain you touched.
2. Deploy from **lcc-hub** (agent runs commands; don't ask Aaron to paste).
3. Deploy script merge-updates live workflow (preserves node ids), slim PUT body.
4. On nginx **413**, deploy falls back to Dispatcher-only patch — enough for handler/SKILLS changes; new webhook nodes may need nginx `client_max_body_size` bump on DO.
5. Workflow deactivate/reactivate runs automatically to register webhooks.
6. Smoke: `ping`, one read command, one card flow (Schedule → Cancel).

### Embedded Code-node rules

- Leading **blank line** at top of `SHARED_CONSTS` and `DISPATCHER_CODE` (n8n V8 line-join quirk).
- No `/regex/` literals in shared helpers — use `new RegExp('…')` or `split`/`join`. See [n8n-io/n8n#26081](https://github.com/n8n-io/n8n/issues/26081).
- Re-run `explore-knack-schema.mjs` when Knack fields change, then redeploy bot.

---

## Adding a new bot command

In [`bots/cl-bot/deploy-teams-bot.mjs`](bots/cl-bot/deploy-teams-bot.mjs):

1. Add handler `async function handleFoo(ctx) { ... }`
2. Register in `SKILLS`: `{ match: 'foo', requires: ..., handler: handleFoo, description: '...' }`
3. Longer matches first if ambiguous (`am status` before `am` if ever added).
4. Update `handleHelp` only if args need a hint (registry drives most of help).
5. For interactive writes: pick card → draft card → confirm submit.
6. Optional: `teams-app/manifest.json` command list (hard cap **10 commands per personal scope**).
7. Document in [`bots/cl-bot/teams-bot.md`](bots/cl-bot/teams-bot.md) skills table.
8. Codestrings check + deploy + smoke.

---

## Adding a new bot domain (future)

When extending beyond Chrome Lot AM/ops (e.g. Turbo Gear, hiring):

| Question | Extend CL Bot | New workflow |
|---|---|---|
| Needs Hubstaff/roster/staticData? | Yes — add entry point to `Teams Bot` | Rarely |
| Separate Azure Bot app / Teams manifest? | Only if different app identity | Yes |
| Shared send path? | Still use `teams-bot-send` | Can POST to same webhook |
| Permissions | New Airtable categories or SKILLS gates | Document in conventions + roster |

Checklist for new domain:

1. Add domain folder under `n8n/bots/<name>/` or `n8n/sync/<name>/`
2. Add row to [`INDEX.md`](INDEX.md)
3. Add spec `.md` + `deploy-*.mjs` if n8n workflow
4. Update [`README.md`](README.md) and `scripts/generate-ops-catalog.mjs` if Tier 1
5. Add router row in `context/router.md`
6. Follow Teams formatting + deploy checklist above

---

## Documentation layering

| Layer | Location | Purpose |
|---|---|---|
| Conventions | `n8n/conventions.md` (this file) | Cross-bot patterns |
| Domain map | `n8n/INDEX.md` | Where to find what |
| Ops / debug | `bots/cl-bot/teams-bot.md`, `sync/pd-todoist/pd-todoist-sync.md` | Runbooks |
| Strategy | `account-management/roadmap.md` | Shipped vs pending |
| IDs / schemas | `context/systems/*.md` | Pipedrive, Knack, roster |
| Human paste | `output/*-paste.md` | PS manual sections |
| Catalog | `context/systems/operations-catalog.md` | Control-plane hub |

---

## See also

- [`INDEX.md`](INDEX.md) — folder layout and read order
- [`bots/cl-bot/teams-bot.md`](bots/cl-bot/teams-bot.md) — CL Bot operating manual
- [`sync/pd-todoist/pd-todoist-sync.md`](sync/pd-todoist/pd-todoist-sync.md) — real-time sync architecture
- [`account-management/README.md`](account-management/README.md) — AM stack map
