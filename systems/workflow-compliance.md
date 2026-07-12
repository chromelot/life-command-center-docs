> **Source:** [`context/systems/workflow-compliance.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/workflow-compliance.md) in the private workspace repo. Do not edit this mirror directly.

# Workflow Compliance — Gap Matrix & Remediation

Last audited: **2026-06-09**. Active workflows: **7**. Retired: micro-scrub, inbox-triage.

## Rubric (0–2 per dimension)

| # | Dimension | 2 = meets standard |
|---|-----------|------------------|
| R1 | Registry ↔ skill alignment | `steps[]` matches skill ledger map; `phaseGates` defined |
| R2 | Execution protocol | Cites workflow-execution, workflow-output-contracts, workflow-logs; exact CLI |
| R3 | Notion logging | Dedicated log DB; create/sync/complete; **Session Complete** |
| R4 | Output contracts | Numbered tables, Present exactly, named sources |
| R5 | Section gates | FIELD CHECK + `gate` before major phase |
| R6 | Data pulls | Phase 0 → `output/*.md` summaries |
| R7 | Turn granularity | One table or one question per turn |
| R8 | Rule enforcement | `.mdc` execution + logging bootstrap |

**Reference:** weekly-plan (gold standard).

## Gap matrix (post Waves 1–4)

| Workflow | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 |
|----------|----|----|----|----|----|----|----|-----|
| weekly-plan | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| weekly-ops | 2 | 2 | 2 | 2 | 2 | 2 | 1 | 2 |
| monthly-plan | 2 | 2 | 2 | 2 | 2 | 1 | 2 | 2 |
| quarterly-plan | 2 | 2 | 2 | 2 | 2 | 1 | 2 | 2 |
| person-checkin | 2 | 2 | 2 | 2 | 2 | 0 | 2 | 2 |
| team-sync | 2 | 2 | 2 | 2 | 2 | 0 | 2 | 2 |
| pd-cleanup | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |

R6=1 for monthly/quarterly: Phase 0 pulls inline (no single briefing script yet). R6=0 for person-checkin/team-sync: optional pull scripts not built.

## Per-workflow status

### weekly-plan — reference
- [x] All dimensions at standard

### weekly-ops
- [x] Logging wired in rule
- [x] In-step multi-item loops documented in skill for 4.1, 3.1, 5.1
- [x] Domain map aligned to Tasks (Phases 2–8)

### monthly-plan
- [x] Ledger map + FIELD CHECKs + Present map (Phases 3–11 tables) in SKILL
- [x] Registry `1.0`–`12.0` + phaseGates
- [x] Execution block in monthly-plan.mdc
- [ ] (Optional) `monthly-data-pull.mjs`

### quarterly-plan
- [x] Ledger map + Present maps (A/C/D/E) + Part B tables + FIELD CHECKs in SKILL
- [x] Registry `A.0`–`E.4` + phaseGates
- [x] Execution block in strategy-review.mdc (`A.0` create, `E.4` complete)
- [ ] (Optional) `quarterly-data-pull.mjs`

### person-checkin
- [x] Dedicated 1:1 Prep Log
- [x] Sub-steps `1.1a`–`1.1c` in registry + rule
- [ ] (Optional) `person-checkin-pull.mjs`

### team-sync
- [x] Dedicated Team Sync Log
- [x] Steps `1.1` + `1.2` per-drift in registry + rule
- [ ] (Optional) `team-sync-diff.mjs`

### pd-cleanup
- [x] Dedicated PD Cleanup Log
- [x] Ledger + tables in SKILL
- [x] Registry aligned with phases

## Retired

| Workflow | Date |
|----------|------|
| micro-scrub | 2026-06-09 |
| inbox-triage | 2026-06-09 |

## Definition of done

All dimensions ≥2 after one live run sets **Session Complete = Complete**.

## See also

- [workflow-execution.md](../workflow-execution.md)
- [workflow-output-contracts.md](workflow-output-contracts.md)
- [workflow-logs.md](workflow-logs.md)