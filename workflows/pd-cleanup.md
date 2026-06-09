> **Source:** [`context/skills/pd-cleanup/SKILL.md`](https://github.com/chromelot/life-command-center/blob/main/context/skills/pd-cleanup/SKILL.md) in the private workspace repo. Do not edit this mirror directly.

# Pipedrive Data Cleanup — SKILL

## Trigger

Activates on **pd cleanup**, **pipedrive cleanup**, **deal cleanup**, or **crm cleanup**. Target duration: ~30–45 min first run; faster on repeat when counts are low.

## Execution Protocol (mandatory)

Read `context/workflow-execution.md`, `context/systems/workflow-output-contracts.md`, `context/systems/workflow-logs.md`.

1. `node scripts/workflow-progress.mjs init --workflow pd-cleanup`
2. Step `0` — `node scripts/pd-data-cleanup.mjs` (silent); present Table 0-A
3. Step `1.0` — `node scripts/workflow-notion-log.mjs create --ledger <path>`
4. Steps `1.1`–`4.1` — one approval per item per turn; sync after each `advance`
5. Before `5.0`: `gate --phase 5`
6. Step `5.0` — Table 5.check + `workflow-notion-log complete`

### Ledger step order

| Step | Skill phase |
|------|-------------|
| `0` | Phase 0 Analyze |
| `1.0` | Create PD Cleanup Log |
| `1.1` | Phase 1a Creates (photo CS + social) |
| `1.2` | Phase 1b Adopt photo deals |
| `1.3` | Phase 1c Delete ex-customer |
| `2.1` | Phase 2a Missing org |
| `2.2` | Phase 2b Missing POC |
| `3.1` | Phase 3 Duplicates |
| `4.1` | Phase 4 Wrong pipeline |
| `5.0` | Phase 5 Commit summary |

### Output contracts

**Table 0-A — Baseline counts** *(step `0`)*

| Category | Count | Source |
|----------|-------|--------|
| create_photo_cs | | `output/pd-cleanup-*.md` |
| create_social | | |
| adopt_photo | | |
| delete_ex_customer | | |
| missing_org | | |
| missing_poc | | |
| duplicates | | |
| wrong_pipeline | | |
| manual_review | | |

**Table N-A — Item review** *(steps `1.1`–`4.1`, one item per turn)*

| Field | Value |
|-------|-------|
| Category | |
| Deal / customer | |
| Proposed action | |
| Fields touched | |
| Recommendation | |
| Decision | Approve / Skip / Stop phase |

**Table 5.check — Commit** *(step `5.0`)*

| Item | Pass |
|------|------|
| Final counts vs baseline in PD Cleanup Log | |
| Remaining manual_review listed | |
| Approvals / executes counts logged | |
| Session Complete = Complete | |

## Inputs

Read via router before starting:

- `context/systems/pipedrive.md` — Pipeline 1 (Sales), 6 (CS), **13 (Social Media)**, stages, user IDs
- `context/systems/knack-fields.md` — `field_464` (current customer), `field_1591` (photo CS deal link), `field_396` (social product), `field_1438` (AM)
- `context/work/chrome-lot/customer-service.md` — ownership, naming conventions

## Interaction style

- **AI decides, Aaron approves — one write at a time.** For every proposed mutation, state "I recommend X because Y" and use AskQuestion on **that item only**: **Approve** / **Skip** / **Stop phase**.
- **Never batch-approve.** Do not ask "approve all 48 deletes?" Do not run `--execute --batch deletes`. Aaron must see and approve each write individually.
- **No writes without per-item approval.** Analyzer is read-only. Executor runs only items listed in `output/pd-cleanup-approved.json`.
- **Do not use Pipedrive MCP for cleanup writes** during this workflow — script + approved manifest only (audit trail in `output/pd-cleanup-execute-log.json`).
- **Re-analyze after each execute call** to confirm fixes and refresh counts.

## Approved manifest (required for any write)

Path: `output/pd-cleanup-approved.json` — JSON array, one object per approved item.

| Batch | Required keys |
|---|---|
| `create_photo_cs` | `batch`, `knack_record_id` |
| `create_social` | `batch`, `knack_record_id` |
| `adopt_photo` | `batch`, `knack_record_id`, `deal_id` |
| `delete_ex_customer` | `batch`, `deal_id` |
| `missing_org` | `batch`, `deal_id` |
| `missing_poc` | `batch`, `deal_id` |
| `duplicates` | `batch`, `keeper_deal_id`, `merge_deal_id` (one entry per merge) |
| `wrong_pipeline` | `batch`, `deal_id` |

Example — Aaron approved two items:

```json
[
  { "batch": "missing_org", "deal_id": 3949 },
  { "batch": "delete_ex_customer", "deal_id": 14 }
]
```

Execute **only** those items:

```
node scripts/pd-data-cleanup.mjs --execute --approved output/pd-cleanup-approved.json
```

The script **refuses `--execute` without `--approved`**. It will not run an entire category from the report.

After a successful run: trim executed entries from the manifest (or reset to `[]`), re-analyze, continue.

## Phase 0: Analyze (silent)

Run on lcc-hub:

```
node scripts/pd-data-cleanup.mjs
```

Read **`output/pd-cleanup-YYYY-MM-DD.md`** (today's date, America/Chicago). Do not load the JSON into context.

Initialize `output/pd-cleanup-approved.json` as `[]` if missing.

Open with summary counts. Group work by phase below — but **approval stays per item within each phase**.

## Phase 1: Deal completeness

Order: **creates → adopts → deletes** (deletes last — most destructive).

### Per-item loop (all of Phase 1)

For each candidate item in the current sub-step:

1. Show: customer/deal #, proposed action, fields touched (PD + Knack).
2. AskQuestion: **Approve this write** / **Skip** / **Stop phase**.
3. On **Approve**: append one entry to `output/pd-cleanup-approved.json`.
4. On **Skip**: do not append; move to next item.
5. On **Stop phase**: run execute for accumulated approvals (if any), then re-analyze or exit phase.

After Aaron finishes reviewing items (or stops), if the manifest is non-empty:

```
node scripts/pd-data-cleanup.mjs --execute --approved output/pd-cleanup-approved.json
```

Then clear executed entries and re-run Phase 0 analyze.

### 1a — Create photo CS + social

Categories: `1_create_photo_cs`, `2_create_social`. Photo creates also write Knack `field_1591`.

### 1b — Adopt unclaimed photo deals

Category: `3_adopt_photo`. Show score and stale `field_1591` replacement if applicable.

### 1c — Delete ex-customer open deals

Category: `4_delete_ex_customer`. **Warn:** sets `status=deleted`. One AskQuestion per `deal_id`.

## Phase 2: Missing orgs + POCs

### 2a — Missing org (`5_missing_org`)

Show deal, proposed org link or create. One approval per `deal_id`.

### 2b — Missing POC (`6_missing_poc`)

Only propose execute when `proposed_person_id` is set (single org contact). Ambiguous → `manual_review`, no append.

## Phase 3: Duplicates

Category: `7_duplicates`. One approval per **merge pair** (`keeper_deal_id` + `merge_deal_id`). Confirm keeper when unclear.

## Phase 4: Wrong pipeline

Category: `8_wrong_pipeline`. One approval per `deal_id`. Sales-style deals in CS pipeline stay in `manual_review` — no execute.

## Phase 5: Commit summary

1. Present final summary counts vs Phase 0 baseline.
2. List remaining `manual_review` items.
3. Pipedrive notes: **per-note approval** before any `add_note` MCP call.
4. Point to `output/pd-cleanup-execute-log.json` for audit trail.

## Out of scope (v1)

- Batch approve or execute without `--approved` manifest
- Auto-merge without keeper confirmation
- Knack writes beyond `field_1591`
- Sales prospect creation from Knack
- n8n automation (Tier 2 interactive only)