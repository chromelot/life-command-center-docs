> **Source:** [`context/work/chrome-lot/social-media.md`](https://github.com/chromelot/life-command-center/blob/main/context/work/chrome-lot/social-media.md) in the private workspace repo. Do not edit this mirror directly.

# Chrome Lot — Social Media

Social product operations for dealership customers. Reviewed in **Weekly Ops Phase 4** (`6.2`) and tracked on **Departments** as **Social Media**.

## What it covers

- **Social Media Management** tiers (V1 / V2 / V3 weekly content volume)
- Service reels, account interaction, giveaways, post boosting
- Remote **Social Media Poster & Editor** + **Social Media Product Manager** roles
- **P&L** — billing vs delivery cost per active account
- **Posting coverage** — every active pipeline account must have posts scheduled/dated each review week

## Systems

| System | Use |
|--------|-----|
| **Pipedrive pipeline 13** | Active social customers — stages 57 / 59 / 60 |
| **Knack `object_65`** | Social posts — **`field_975`** schedule date; **`field_977`** customer |
| **Knack `object_25`** | Product subscriptions — **`field_281`** monthly price, **`field_501`** unit cost |
| **Knack `object_21`** | Line items — review-week **`field_266`** billing, cost fields for actual P&L |
| **Knack `object_2` `field_396`** | Social product link on customer (often empty; PD + product rows are source of truth) |

## Pipedrive pipeline 13 stages

| Stage ID | Name | Weekly ops treatment |
|----------|------|-------------------|
| 57 | Advanced Customers | **Active** — P&L + posting + gap/stale review |
| 59 | Active Sales | **Active** |
| 60 | Basic Customers | **Active** |
| 58 | Inactive Sales | Inactive pool — count only |
| *(other stage IDs)* | Often legacy / wrong pipeline | **Mis-staged** — flag for cleanup |

## P&L (Phase 4 — Table 6-B0)

From briefing **SOCIAL P&L — BILLING VS EXPENSES** + **SOCIAL LABOR DETAIL**:

### Billing (revenue)

| Row | Source |
|-----|--------|
| **Review-week billing** | Sum social **line items** (`object_21`) in review week — `field_266` |
| **Monthly run-rate billing** | Per active PD account: **confirmed override** → Knack social product **`field_281`** → customer **`field_459`**. Overrides (2026-06): Cars and Pickups Addison **$1,996/mo**, Central Autohaus **$900/mo**. |

When review-week line items are $0, use **monthly run-rate billing**.

### Billing by week (Table 6-B0b)

Briefing section **SOCIAL BILLING BY ACCOUNT** — current calendar month, Monday-start weeks:

| Column | Source |
|--------|--------|
| **Monthly** | Confirmed contract per account (see above) |
| **Wk MM-DD** | Sum of line items (`object_21` **`field_266`**) dated in that Mon–Sun week |
| **MTD actual** | Sum of line items in calendar month |
| **Source** | `confirmed override` / `Social product (object_25)` / `Monthly Value (field_459)` |

### Expenses (delivery cost)

| Row | Source |
|-----|--------|
| **Labor — Han, Kadyn, Naids** | Tracked hours in review week × **Airtable `Hourly Rate`**. Source per person: **Hubstaff** (`Hubstaff User ID`), **Time Doctor** (`Pay Tracking Methods` + email match), or **Airtable Time Punches** (`tblGUhK3zMWKDzsxq` — Han = `Name` **Han**). |
| **Buffer subscription** | Fixed monthly cost — set **`SOCIAL_BUFFER_MONTHLY_USD`** in `n8n/.secrets.local`. Review week = monthly ÷ (52/12). |
| **Monthly labor est** | Review-week labor × (52/12) |
| **Total expenses** | Labor + Buffer (Knack product `field_501` unit cost is **not** used for ops P&L) |

Briefing **SOCIAL LABOR DETAIL** table: hours, rate, cost, time source + ref per person. RED FLAG when no trackable source or zero hours.

| Person | Airtable rate (2026-06) | Time source | Ref |
|--------|-------------------------|-------------|-----|
| Hanbin Li (Han) | $18/hr | **Airtable Time Punches** | `Name` = Han (`Payment Method` = Zelle — payout only, not time tracking) |
| Kadyn Blair | $22/hr | Hubstaff | `3461841` |
| Naids Redona | $7.50/hr | Time Doctor | `naidseuredona@gmail.com` |

Flag: negative monthly margin; missing Buffer secret; missing/zero hours for rostered labor with a configured tracker.

## Active account posting (Phase 4 — Table 6-B1 / 6-B2)

**Active** = open PD deals on stages 57 / 59 / 60.

For each active account, count posts in the **review week** where Knack post **`field_977`** customer name matches the deal org name (fuzzy match).

| Status | Rule |
|--------|------|
| **OK** | ≥1 post with `field_975` in review week |
| **MISSING** | 0 posts — review one account per turn (Table 6-B2): schedule backlog, pause customer, data gap, or defer |

Briefing section **ACTIVE ACCOUNT POSTING** lists all rows; **MISSING POSTS** subsection feeds RED FLAGS.

## Pipeline behind rules

Active-stage deals with **no `next_activity_date`** or **60+ days** stale with no open activity — Table 6-B5, one account per turn (same pattern as CS behind visits).

## Briefing section

`weekly-data-pull.mjs` → **SOCIAL MEDIA — PIPELINE & POSTING**:

- **SOCIAL P&L** — billing vs expenses (review week + monthly run-rate)
- **ACTIVE ACCOUNT POSTING** — per-customer review-week post count + MISSING list
- Open deal counts by stage; deal gaps + stale list
- Posts by week aggregate (`field_975`)

## Department health (Phase 4)

**Departments** entry: **Social Media** (`39bf40c2-487b-816d-97a3-f6f870b3b6e1`, `Domain = Chrome Lot`). Re-rate in Phase 4 (Table 6.2-H) when P&L, posting gaps, or pipeline evidence changed.

## See also

- [overview.md](overview.md) — services list
- [../../systems/pipedrive.md](../../systems/pipedrive.md) — pipeline 13
- [../../systems/knack-fields.md](../../systems/knack-fields.md) — line items + posts
- [../../skills/weekly-ops/SKILL.md](../../skills/weekly-ops/SKILL.md) — Phase 4 tables