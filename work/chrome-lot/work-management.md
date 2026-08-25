> **Source:** [`context/work/chrome-lot/work-management.md`](https://github.com/chromelot/life-command-center/blob/main/context/work/chrome-lot/work-management.md) in the private workspace repo. Do not edit this mirror directly.

# Chrome Lot — Work Management

Daily operational posture for field coverage and org priority. Reviewed in **Weekly Ops Phase 1.3** (review week, Mon–Sun) and summarized on the **Weekly Ops Meeting Log**.

## Daily operations records

Each **business day** in the review week gets a row in the ops briefing **DAILY OPERATIONS RECORDS** section (from `weekly-data-pull.mjs`):

| Signal | Source | Meaning |
|--------|--------|---------|
| Jobs | Knack `object_9` by job date (`field_1033`) | Throughput / business level |
| Call-ins | Knack `object_75` by date (`field_1264`) | Coverage stress |
| Backup photographer used | Jobs assigned to photographers whose rank (`field_1209`) includes **Backup** | Backup tier required that day |
| Unassigned jobs | Jobs with empty photographer (`field_52`) | Dispatch gap |

**Business level** (per day, from job count):

| Level | Jobs that day |
|-------|----------------|
| Slow | ≤ 4 |
| Normal | 5–7 |
| High | 8+ |

Kevin's **Daily Operations Check-In** / **Daily Closeout** (Process Street) are the human record — cross-check in the meeting when briefing flags call-ins, backups, or unassigned jobs.

## Backups required

Summarize the review week: which days needed **backup-tier photographers** (rank contains `Backup`) or had **call-ins** driving coverage changes. Name photographers and customers if action is needed.

## Org priority — Sales vs Hiring

Current posture for **where Aaron and ops attention goes this planning week**:

| Priority | When |
|----------|------|
| **Sales** | Account Management / new business debt is the binding constraint (stale pipeline, AM follow-ups, CS not the bottleneck) |
| **Hiring** | **Only when Knack hiring app shows HIRING ACTIVE: yes** (≥1 open position with `field_172` = Open) **and** personnel/delegation strain or unhealthy pipeline (zero review-week apps, Unhealthy job ad on open req) |
| **Service delivery** | Field coverage / backups / call-in pattern is the binding constraint *(ops priority label — not the Photographer Management department)* |
| **Balanced** | No single domain dominates |

**Hiring active gate:** `weekly-data-pull.mjs` → **HIRING — KNACK HIRING APP** section. If **HIRING ACTIVE: no**, Weekly Ops Phase 1.3 option **B (Hiring)** is N/A.

### Knack hiring app (read-only)

Separate app from main CL Knack (`KNACK_HIRING_*` in `n8n/.secrets.local`):

| Object | Use |
|--------|-----|
| `object_2` Open Positions | `field_172` status — **Open** = actively hiring |
| `object_23` Job Ads | `field_453` posting health; `field_454` → position |
| `object_4` Applications | `field_113` import date — weekly breakdown in briefing |

Healthy pipeline = regular application imports each week (briefing flags review week with zero apps).

## Hiring review (Weekly Ops `2.1`)

When **HIRING ACTIVE: yes**, Phase 2 opens with:

1. **Table 2.1-B** — each active job ad linked to an open position + **field_453** health
2. **Table 2.1-C** — applications imported by week (`field_113`)
3. **Table 2.1-D** — one pipeline issue per turn when unhealthy: repost ad, step in on applicant review, or redelegate review

When **HIRING ACTIVE: no**, step is N/A (ops priority option B also N/A in Phase 1.3).

Use Departments **Priority** + **Health** (briefing **DEPARTMENTS**) plus Phase 2.1 (hiring), Phase 8 (sales), and hiring briefing signals. Aaron confirms one letter in Phase 1.3; stored on Weekly Ops Log as **Ops Priority**.

## Department health (in-phase — not a separate phase)

**Departments** DB (`39bf40c2-487b-816d-97a3-f6f870b3b6e1`, `Domain = Chrome Lot`) — each department: **Health** (Healthy / Unhealthy / Critical / Stalled), **Priority** (Low / Medium / High).

Briefing **CL DEPARTMENTS — HEALTH & PRIORITY** is the baseline. **Re-rate in the domain phase** when that week's evidence warrants a change — not a separate cross-cutting pass. Updates require Aaron approval before Notion write (`personal_notion_update_page`).

| CL Department | Weekly Ops phase | Ledger step |
|---------------|------------------|-------------|
| Delegation & Hiring | Phase 2 | `2.1` — Table 2.1-H |
| Personnel Management | Phase 2 | `2.1` — Table 2.1-H |
| Photographer Management | Phase 3 | `2.4` — Table 2.4-H |
| Social Media | Phase 4 | `6.2` — Table 6.2-H |
| Customer Service | Phase 6 | `2.3` — Table 2.3-H |
| Admin | Phase 7 | `4.1` — Table 4-H |
| Account Management | Phase 8 | `3.2` — Table 3-H |

**Standard table** (per department):

| Health | Priority | Change? | Reason |
|--------|----------|---------|--------|

Full quarterly Picture-of-Success read remains **quarterly plan** only.

## See also

- [operations.md](operations.md) — Process Street daily workflows, Kevin reporting
- [../../skills/weekly-ops/SKILL.md](../../skills/weekly-ops/SKILL.md) — Phase 1.3 + Phase 2.1 hiring + in-phase dept health tables
- [../../systems/notion-databases.md](../../systems/notion-databases.md) — Departments + Weekly Ops Log fields