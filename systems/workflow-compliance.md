> **Source:** [`context/systems/workflow-compliance.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/workflow-compliance.md) in the private workspace repo. Do not edit this mirror directly.

# Workflow Compliance — Gap Matrix & Remediation

Last audited: **2026-06-09**. Active Tier 2 workflows: **7**. Retired: micro-scrub, inbox-triage.

## Rubric (0–2 per dimension)

| # | Dimension | 2 = meets standard |
|---|-----------|------------------|
| R1 | Registry ↔ skill alignment | `workflow-registry.mjs` steps match skill ledger map; `phaseGates` defined |
| R2 | Execution protocol | Cites workflow-execution, workflow-output-contracts, workflow-logs; exact CLI |
| R3 | Notion logging | Dedicated log DB; create/sync/complete; **Session Complete** |
| R4 | Output contracts | Numbered tables, Present exactly, named sources |
| R5 | Section gates | FIELD CHECK + `gate` before major phase |
| R6 | Data pulls | Phase 0 → `output/*.md` summaries |
| R7 | Turn granularity | One table or one question per turn |
| R8 | Rule enforcement | `.mdc` execution + logging bootstrap |

**Reference:** weekly-plan (gold standard).

## Gap matrix

| Workflow | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 |
|----------|----|----|----|----|----|----|----|-----|
| weekly-plan | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| weekly-ops | 2 | 2 | 2 | 2 | 2 | 2 | 1 | 2 |
| monthly-plan | 1 | 1 | 1 | 1 | 1 | 0 | 1 | 1 |
| quarterly-plan | 1 | 1 | 1 | 1 | 1 | 0 | 1 | 1 |
| person-checkin | 2 | 2 | 1 | 1 | 1 | 0 | 2 | 2 |
| team-sync | 2 | 2 | 1 | 1 | 1 | 0 | 2 | 2 |
| pd-cleanup | 1 | 1 | 1 | 0 | 1 | 2 | 1 | 1 |

R3=1 until first live session writes Session Complete on new dedicated DBs.

## Per-workflow checklists

### weekly-plan
- [x] Dedicated Weekly Meeting Log
- [ ] Session Complete on all in-progress entries after migration
- [ ] `workflow-notion-log` in weekly-meeting.mdc

### weekly-ops
- [x] Dedicated Weekly Ops Meeting Log
- [ ] In-step sequence docs for 2.2, 3.1, 4.1 multi-item loops
- [ ] `workflow-notion-log` in weekly-ops.mdc

### monthly-plan
- [ ] Ledger map in SKILL matches registry (`1.1`–`12.0`)
- [ ] FIELD CHECK tables: `1.check`, `12.check`
- [ ] Full Notion log lifecycle in monthly-plan.mdc
- [ ] (Optional) `monthly-data-pull.mjs`

### quarterly-plan
- [ ] Ledger map matches registry (`A.1`–`E.2`)
- [ ] Present-exactly tables for Parts B–D
- [ ] Full Notion log lifecycle in strategy-review.mdc

### person-checkin
- [x] Dedicated 1:1 Prep Log DB
- [ ] Sub-steps `1.1a`–`1.1c` in rule
- [ ] (Optional) pull script

### team-sync
- [x] Dedicated Team Sync Log DB
- [ ] `team-sync-diff.mjs` briefing
- [ ] Steps `1.1` summary + `1.2` per-drift

### pd-cleanup
- [x] Dedicated PD Cleanup Log DB
- [ ] Numbered table contracts in SKILL
- [ ] Ledger `1.1`–`5.0` aligned with skill phases

## Retired workflows

| Workflow | Retired | Reason |
|----------|---------|--------|
| micro-scrub | 2026-06-09 | Legacy — replaced by monthly project audit + weekly plan |
| inbox-triage | 2026-06-09 | Legacy — idea routing handled ad hoc |

## Definition of done

Workflow is **up to code** when all dimensions ≥2 and one full run sets **Session Complete = Complete** on its log DB.

## See also

- [workflow-execution.md](../workflow-execution.md)
- [workflow-output-contracts.md](workflow-output-contracts.md)
- [workflow-logs.md](workflow-logs.md)