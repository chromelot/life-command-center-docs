> **Source:** [`context/systems/pipedrive.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/pipedrive.md) in the private workspace repo. Do not edit this mirror directly.

# Pipedrive — pipelines, stages, user IDs, quirks

Chrome Lot's CRM. Domain: `chromelot`.

## Pipelines

| ID | Name | Use |
|---|---|---|
| 1 | Sales | Prospects being sold to. Stages include Aware, Warm, Hot. |
| 6 | Customer Service (CS) | Existing photo customers. Deal names follow pattern `[Company] CS` (e.g., "Earth Motorcars CS", "One Legacy Motors CS"). No "Social" in title. |
| 13 | Social Media | Social product customers (`field_396` in Knack). Titles fuzzy-match customer name; not `[Company] Social CS` on Pipeline 6. |
| 12 | Not Actively Working | Dormant / archived deals. Also called the **cold sales pool** in weekly planning. |

### Cold sales pool (weekly plan convention)

During **weekly plan Phase 6**, stale Sales Pipeline 1 deals Aaron no longer actively pursues move to **Pipeline 12** after one-by-one review. Each move requires case-by-case approval before `pipedrive_update_deal`. Stale detection uses the **Stale** label (same signal as CL Bot `stale deals`); fallback: no activity in 14+ days.

## CS pipeline stages (Pipeline 6)

| Stage ID | Name |
|---|---|
| 28 | Needs Attention |
| 61 | Happy |
| 26 | AT RISK |

## User IDs

> Canonical roster lives in Airtable `Admin / Payable Employees` (`Pipedrive User ID` field) — see [airtable-roster.md](airtable-roster.md). The IDs below are a snapshot for quick API reference. If they drift, Airtable wins.

| User | ID | Notes |
|---|---|---|
| Aaron Hoegenauer | 18865844 | PD seat |
| Lexie Logan | 22704318 | PD seat |
| Tristen VanSteeter | 20938631 | **Removing PD seat** — Todoist-only delegate |
| Ran Peer | 19274648 | **Removing PD seat** — Todoist-only delegate |

**Todoist-only sub-delegates** (no PD seat; work via **deal** Sub AM Email → shared Todoist project): Tristen, Ran, Kadyn, Han, Lynn, Kevin. Org-level Sub AM is legacy. See `n8n/pd-delegate-fields.json` + `n8n/pd-delegate-resolution.mjs`.

## Deal-level Delegated label

When a deal has **Sub Account Manager Email** (1 or 2) set, automation adds the native deal **`Label`** option **Delegated** and sets custom **Sub-Delegate Name** from the primary Sub AM email (Sub AM 1, else Sub AM 2). When both email fields are cleared, **Delegated** is removed and **Sub-Delegate Name** is cleared. Other labels (e.g. **Stale**) are **not** replaced — writes merge comma-separated label option IDs, same pattern as the Stale rollup.

- Real-time: `PD Deal Delegate Label` workflow (`POST /webhook/pd-deal-event`, PD webhook `*.deal`)
- Nightly heal: `Team PD Reconciler` (open deals, cap 100 delegate-field changes/run)
- Setup: `node n8n/setup-pd-delegate-label.mjs` (after creating the label in Pipedrive); field keys in `n8n/pd-delegate-fields.json`

## MCP tools

`get_activities`, `get_prioritized_activities`, `add_activity`, `update_activity`, `add_note`, `get_deals`, `update_deal`, `search`, plus contact/org operations.

## Operating rules (always active)

- **Every activity must have a deal.** When creating a Pipedrive activity, search for and attach the appropriate deal. For customer service on existing customers, look for a deal ending in "CS". For new business, attach the relevant Pipeline 1 deal. **Never leave `deal_id` empty.**
- **Cap at 3 activities/day.** If planning more than 3, defer the rest with Aaron's input on timing.
- **Pipedrive is the system of record for customer tasks.** Any Todoist task involving customer/prospect contact should ALSO have a corresponding Pipedrive activity.

## Deferral tracking (evening defer)

Pipedrive activities cannot have custom fields. The n8n `Team Evening Defer` workflow (5pm CT Mon–Sat) defers overdue open activities to the next day, but **never to Fri/Sat/Sun** — those snap to Monday. It tracks how many times each activity has been auto-deferred:

| Where | Format |
|---|---|
| PD subject | Clean title (no deferral prefix) |
| PD `note` | `Deferrals: N` and `Manual deferrals: M` lines (machine-parseable) |
| Todoist mirror title | Same clean subject (via mirror sync) |
| Todoist mirror description | `Deferrals: N` line |
| Todoist labels | `deferred-1`, `deferred-2`, or `deferred-3` (3+ capped at `deferred-3`) |

Auto deferrals increment only in `Team Evening Defer`. Manual forward reschedules increment `Manual deferrals` (via `Todoist Events` + `PD Activity Mirror`). At **3+ total** deferrals, route through the delegation framework before rescheduling again (workspace non-negotiable). Filter stuck items: Todoist `@deferred-3`.

Implementation: `n8n/pd-deferral-tracking.mjs` (shared helpers inlined into deploy scripts).

## Deal-level stale label (`Stale`)

`Team Evening Defer` rolls up open-activity signals onto each deal in **Sales (1)** and **CS (6)** via the native Pipedrive deal **`Label`** option **Stale** (colored chip in pipeline kanban). **No stage moves.** Config: `n8n/pd-stale-labels.json`.

| Reason (computed at read time) | Rule |
|---|---|
| **Chronic deferral (7+)** | Any open activity has **auto + manual** deferrals ≥ 7 |
| **Deferral warning (3–6)** | Total deferrals 3–6 (no chronic flag) |
| **No touch 60+ days** | `last_activity_date` (last completed activity on deal) ≥ 60 days ago — both pipelines |

Any matching reason adds the **Stale** label to the deal's existing labels (multi-label set). Removing stale clears only **Stale** — other labels are preserved. On **first** application of the Stale label, an automated Pipedrive **note** is added to the deal with the reason(s). Reason detail also appears in Morning DM / Teams `stale deals` text (not on the chip).

Sales deals without `next_activity_date` are **not** stale — surfaced separately in `Team Morning DM` and Teams `deals` command.

**Deferral counting**

| Type | Increments when |
|---|---|
| Auto (`Deferrals: N`) | `Team Evening Defer` rolls due date forward at 5pm |
| Manual (`Manual deferrals: M`) | Due date pushed forward in Todoist or Pipedrive without an auto bump |
| Activity `note` | `Deferrals: N` + optional `Manual deferrals: M` |

| Where | Format |
|---|---|
| PD deal | Native **`Label` = Stale** — enable Label column in Sales + CS pipeline views |
| Morning DM / Teams `stale deals` | Lists deals with **Stale** label; reasons computed from activities |
| Activity vs deal | Activity: delegation at **3+** total deferrals. Deal chronic reason at **7+**. |

**Clearing:** On any activity completion (PD webhook or Todoist complete), `refreshDealStaleReasons` re-evaluates the deal and removes only the **Stale** label when no reasons apply (other labels unchanged). Also clears legacy **`Stale Reasons`** set field and **`Stale`** checkbox on write passes during migration.

**API note:** Pipedrive's deal `label` field is a multi-option set. Writes use comma-separated option IDs (e.g. `"89,133"`) so **Stale** merges with existing labels instead of replacing them.

Setup: `node n8n/setup-pd-stale-labels.mjs`. Rollup: `n8n/pd-deal-stale-rollup.mjs`. Dry-run: `node output/pd-stale-rollup-dryrun.mjs`.

## PD/Todoist sync (n8n, real-time)

Team-wide webhook stack — full architecture in [`n8n/sync/pd-todoist/pd-todoist-sync.md`](../../n8n/sync/pd-todoist/pd-todoist-sync.md).

- **Mirror**: `PD Activity Mirror` — every open PD activity (with due date) → shared Todoist project `Pipedrive` (`6gfPq74rf2fJwGJx`). Stop activities get the `stop` label.
- **Round-trip**: `Todoist Events` — complete/uncomplete/date/notes sync back to PD. Todoist completion on Sales/CS deals auto-creates next activity (with dedup).
- **Defer**: `Team Evening Defer` — 5pm CT Mon–Sat rolls unfinished activities forward + tracks deferrals + updates deal **Stale** label.
- **Safety net**: `Team PD Reconciler` — 4am drift heal; `Team Morning DM` — 9am read-only summary.

Legacy v1 `Morning Pipedrive Sync` / `Evening Pipedrive Close` (Aaron-only Office/Errands batch sync) were removed 2026-06-05.

## API quirks (matters for data pulls)

### User filtering

`pipedrive_get_activities` without `user_id` only returns the **authenticated user's** (Aaron's) activities. To get all users, call once per `user_id`:
- Aaron: 18865844
- Tristen: 20938631
- Lexie: 22704318
- Ran: 19274648

### Completed activities — use `updated_since`, not `due_date`

The `done: "1"` filter combined with `due_date_start/end` returns very old data via the v1 API. **Don't use this for recently completed activities.** Instead:

```javascript
pipedrive_get_activities({
  done: "1",
  updated_since: "2026-04-06T00:00:00Z"  // RFC3339, uses v2 API
})
```

Then **filter results client-side** by `marked_as_done_time` within the target date range.

### Open activities with date filter

`due_date_start` / `due_date_end` only return open (incomplete) activities. Completed activities in that date range are invisible to this filter. To get both, run two queries (open with `due_date_*`, completed with `updated_since` + client-side filter).

### Correct approach for weekly / monthly KPIs

1. Pull completed activities via `updated_since` set to the start of the target period
2. Filter results client-side by `marked_as_done_time` within target window
3. Group by `owner_id` and `type`

## Activity types in this workspace

| Type | Used for |
|---|---|
| `meeting` (Stop) | In-person customer / prospect visit. Mirrored to Todoist Errands. |
| `call` | Phone call to customer / prospect |
| `email` | Outbound email |
| `task` | Catch-all action (follow-up, prep, etc.) |

## Related Knack fields

- `field_1591` (object_2 / Customers) holds the **Pipedrive Deal ID** for cross-reference between Knack and Pipedrive.

## See also

- [mcp-servers.md](mcp-servers.md#pipedrive--chrome-lot-crm) — MCP entry
- [knack-fields.md](knack-fields.md) — Knack object/field references for the customer-side cross-references
- [../work/chrome-lot/customer-service.md](../work/chrome-lot/customer-service.md) — CS workflow that uses Pipeline 6
- [../work/chrome-lot/sales.md](../work/chrome-lot/sales.md) — sales workflow that uses Pipeline 1