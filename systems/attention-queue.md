> **Source:** [`context/systems/attention-queue.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/attention-queue.md) in the private workspace repo. Do not edit this mirror directly.

# ZPT Attention Queue

Aaron's **single operations surface** in Zero Pit Stop: a ranked list of what needs him across managed entities — departments, people (v2: customers and deals from Turbo Gear).

Replaces the retired **weekly-ops** Cursor workflow (942 lines, ran once, stalled at step 2).

## The primitive

Every managed thing is an entity with:

- **health** (or status for people)
- **interval_days** — pure function of health (see tables below), unless overridden
- **last_visited_at** — for departments: `MAX(queue_visits.visited_at)`; for people: Airtable `Last 1:1`

Due-ness is **computed at read time**, never stored:

```
raw_ratio     = days_since(last_visited_at) / interval_days
overdue_ratio = min(raw_ratio, 3.0)   // starvation guard
sort          = overdue_ratio DESC, days_since DESC
```

**Rotation mode** sorts by `days_since` only — breadth over triage.

## Where health lives

| Entity | Health / status store | Aaron edits in ZPT |
|---|---|---|
| Department | D1 `departments` | `#/departments` |
| Person | Airtable `Management Status` | `#/person/<airtableRecId>` or queue review card |
| Customer / deal (v2) | Turbo Gear | TG app |

ZPT owns **attention** (visit log + decisions), not operational data copies.

## Interval tables

Configured in `dashboard/web/functions/lib/attention-intervals.js` (Workers + mirrored in `dashboard/web/src/attentionIntervals.js` for UI).

**Departments:** Critical 7 · Unhealthy 14 · Okay/Watch 28 · Healthy 56 · Not assessed/Stalled 14

**People:** Below expectations 7 · New/Onboarding 7 · Meeting 30 · Exceeding 42

Airtable optional override: `1:1 Interval Override` (days).

## UI routes

| Route | Purpose |
|---|---|
| `#/queue` | Ranked attention queue |
| `#/departments` | All departments |
| `#/department/<id>` | Department detail + visit history |
| `#/person/<airtableRecId>` | Person roster fields + visit history |

Nav: Work sidebar → **Attention queue** / **Departments** (after Chrome Lot — Today).

**Person visit loop (v1.1):** `Record visit` on a person PATCHes Airtable `Last 1:1` to the visit date (CT) before writing `queue_visits`. Ranking uses Airtable — without this step the person stays at the top.

## Adding a feed (v2+)

1. Add interval table (if new entity type) to `attention-intervals.js`.
2. Create `attention-feed-<type>.js` exporting `listEntities(env)` → common shape.
3. Register in `attention-queue.js` `FEEDS` array.
4. Extend review card if the entity needs special fields.

Do **not** add a second page or parallel queue.

## Visit log

Domain: `queue_visits` (append-only). Shape: `{ entity_type, entity_ref, entity_name, visited_at, health_at_visit, decisions_md, actions[], checklist_responses[] }`.

Recording a visit for a **person** also updates Airtable `Last 1:1` to the visit date.

## Roadmap

- **v2** — Turbo Gear customer + deal feeds (CRM plan Phase 7+)
- **v3** — Write-back (Todoist / Pipedrive actions from captured follow-ups)

## See also

- [`../work/systems/attention-queue-plan.md`](../work/systems/attention-queue-plan.md) — execution plan + locked decisions
- [`../work/chrome-lot/work-ops-roadmap.md`](../work/chrome-lot/work-ops-roadmap.md) — milestone trail
- [`airtable-roster.md`](airtable-roster.md) — people feed source fields
- [`operations-catalog.md`](operations-catalog.md) — workflow catalog