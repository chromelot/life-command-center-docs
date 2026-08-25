> **Source:** [`context/work/chrome-lot/operations.md`](https://github.com/chromelot/life-command-center/blob/main/context/work/chrome-lot/operations.md) in the private workspace repo. Do not edit this mirror directly.

# Chrome Lot — Operations

Day-to-day photographer performance and recurring workflows. **Weekly Ops Phase 3 — Photographer Management** (`2.4`).

## Photographer Management (Weekly Ops `2.4`)

### Briefing pull

`weekly-data-pull.mjs` → **PHOTOGRAPHER MANAGEMENT**:

| Section | Content |
|---------|---------|
| **Roster summary** | All active photographers: rank, grade, review-week jobs, call-ins, issues/car, days since 1:1 (`field_985`) |
| **Perf meetings behind** | `field_985` ≥ **30 days** since last logged perf 1:1 |
| **Active monitoring** | Threshold flags: call-ins >3, issues/car >0.5, time-off >3, below-grade, missing grade |

Skips photographers with **Exclude from Photographer Table** (`field_1382`).

### Meeting flow

1. Present summary tables 2.4-C0 → 2.4-C1 from briefing
2. Clear **perf behind** list (Table 2.4-C2) — schedule 1:1s
3. Review **one monitored photographer per turn** (Table 2.4-C3) — pull latest `object_57` comment where `field_1006` = Photographer
4. Decide action: coaching, written warning, schedule meeting, set missing grade (Knack write with approval), Todoist follow-ups

### Flag thresholds (object_7)

| Field | Trigger |
|-------|---------|
| field_1267 | Call-ins last month > 3 |
| field_1362 | Average issues per car > 0.5 |
| field_1446 | Time off requests last month > 3 |
| field_1338 | Below Expectations or blank |
| field_985 | ≥ 30 days since logged perf 1:1 |

## Process Street — recurring workflows

Active templates Aaron's team runs day-to-day:

- Daily Closeout
- Daily Operations Check-In
- Daily Customer Notifications
- Daily Helpdesk Clearing
- Daily Hiring Follow Up
- Daily Operations Support Checklist
- Afternoon Photographer Check In
- Checking Customers with Missing Cars
- Customers with High Job Days Report
- Customer Standards Onboarding
- Customer Exit Checklist

## Daily ops reporting (Kevin)

Kevin Romero owns daily ops reporting. Aaron's recurring Todoist tasks for Kevin:

- Send agenda to Kevin
- Brief list/summary for check-in report

These are Aaron's **briefing** tasks; Kevin executes the actual reports. See [../../systems/task-glossary.md](../../systems/task-glossary.md) for the full glossary.

## Vmax / vDubs photographer management

Lynn Lomibao owns weekly photographer reminders for the Vmax and vDubs accounts. Standing recurring Todoist tasks include:

- Send Vmax photogs new standards
- Remind vDubs photographers
- Saturday coverage check
- Check vDubs listings

## Sale stops vs. CS stops

- **CS stops** — see [customer-service.md](customer-service.md). Existing customers, weekly batch of 6-9 stops distributed across team.
- **Sale stops** — see [sales.md](sales.md). Prospects, slower (often turn into discovery), Aaron does 2-3/week max.

## See also

- [../../systems/knack-fields.md](../../systems/knack-fields.md) — full field reference for `object_7` (Photographers) and `object_57` (Comments)
- [../../people/index.md](../../people/index.md) — team delegation matrix
- [../../systems/task-glossary.md](../../systems/task-glossary.md) — opaque task names
- [customer-service.md](customer-service.md) — CS workflow
- [sales.md](sales.md) — sales workflow
- [work-management.md](work-management.md) — daily ops records, ops priority