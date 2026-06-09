> **Source:** [`context/systems/workflow-logs.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/workflow-logs.md) in the private workspace repo. Do not edit this mirror directly.

# Workflow Logs — Notion + Ledger Standard

Every **Tier 2 Cursor workflow** produces a **Notion log entry** and maintains a **local JSON ledger** in sync.

## Session Complete (all log DBs)

| Property | Values | When |
|----------|--------|------|
| **Session Complete** | Incomplete / Complete | Incomplete on create + sync; **Complete** only via `workflow-notion-log complete` |

Replaces legacy `Workflow Status` (In Progress / Complete / Paused). Paused sessions stay **Incomplete** with `Current Step` preserved.

## Resume properties

| Property | Purpose |
|----------|---------|
| Current Step | Ledger `current_step` |
| Session Complete | Finished or in-progress |
| Completed Steps | Audit trail |
| Ledger Suffix | `week_of` or `session_date` |

## Log database map (7 active workflows)

| Workflow | Log database | ID / config |
|----------|--------------|-------------|
| Weekly Plan | Weekly Meeting Log | `322f40c2-487b-81bd-8953-ffc40ac6432d` |
| Weekly Ops | Weekly Ops Meeting Log | `379f40c2-487b-8130-916d-eba9ce85134c` |
| Monthly Plan | Monthly Meeting Log | `344f40c2-487b-806d-98b2-ef710856bd07` |
| Quarterly Plan | Quarterly Meeting Log | `344f40c2-487b-80ed-b183-ebed46d80130` |
| 1:1 Prep | 1:1 Prep Log | `config/workflow-logs.json` → `person_checkin_log_db_id` |
| Team Sync | Team Sync Log | `config/workflow-logs.json` → `team_sync_log_db_id` |
| PD Cleanup | PD Cleanup Log | `config/workflow-logs.json` → `pd_cleanup_log_db_id` |

Provisioner: `node scripts/create-workflow-log-dbs.mjs`

**Retired:** Workflow Session Log (`37af40c2-487b-8139-9ae1-ec32aae63e0e`) — deprecated 2026-06-09.

## Scripts

| Script | Role |
|--------|------|
| `scripts/lib/workflow-registry.mjs` | Steps, log DB IDs, phase gates |
| `scripts/workflow-progress.mjs` | Ledger |
| `scripts/workflow-notion-log.mjs` | create · sync · commit-phase · complete |
| `scripts/create-workflow-log-dbs.mjs` | Provision session log DBs + Session Complete |
| `scripts/provision-workflow-log-properties.mjs` | Idempotent property patch |

## Agent checklist

```
1. Read workflow-execution.md + workflow-output-contracts.md + workflow-logs.md
2. workflow-progress.mjs init
3. workflow-notion-log.mjs create (Session Complete = Incomplete)
4. status → present only current_step
5. advance → sync after each step
6. complete → Session Complete = Complete
```

### Monthly plan — phase writes

| Step | Notion fields |
|------|---------------|
| `1.0` | Create log (review month title, Session Complete = Incomplete) |
| `1.1`–`1.4` | Incremental — life health selects as rated |
| `11.2` | `Priority Stack`, `Domains Parked`, `Active CL Sprint` (session state → log at 12.0) |
| `12.0` | Full Monthly Meeting Log rollup + Team Activity Details + Session Complete = Complete |

### Quarterly plan — phase writes

| Step | Notion fields |
|------|---------------|
| `A.0` | Create Quarterly Meeting Log shell |
| `A.1`–`B.3` | Lessons / narrative fields incrementally |
| `E.3` | `Priority Stack`, `Domains Parked` (session state) |
| `E.4` | Full quarterly rollup + Team Activity Details + Session Complete = Complete |

### Session workflows — phase writes

| Workflow | Step | Action |
|----------|------|--------|
| person-checkin | `1.0` | Create 1:1 Prep Log |
| person-checkin | `1.1a`–`1.1c` | Workload, talking points, delegation fields |
| person-checkin | `2.0` | Session Complete = Complete |
| team-sync | `1.0` | Create Team Sync Log |
| team-sync | `1.1` | Drift summary counts |
| team-sync | `1.2` | Per-drift resolution (one item/turn) |
| team-sync | `2.0` | Session Complete = Complete |
| pd-cleanup | `1.0` | Create PD Cleanup Log |
| pd-cleanup | `1.1`–`4.1` | Approvals / executes logged incrementally |
| pd-cleanup | `5.0` | Final counts + Session Complete = Complete |

## See also

- `context/systems/workflow-compliance.md` — gap matrix
- `context/systems/workflow-output-contracts.md` — table specs
- `context/systems/notion-databases.md` — full property schemas