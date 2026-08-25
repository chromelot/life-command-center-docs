> **Source:** [`context/work/chrome-lot/customer-service.md`](https://github.com/chromelot/life-command-center/blob/main/context/work/chrome-lot/customer-service.md) in the private workspace repo. Do not edit this mirror directly.

# Chrome Lot — Customer Service

CS for existing customers. Pipeline 6 in Pipedrive. Used by weekly meeting Phase 4.

## Pipeline structure

- **Pipedrive Pipeline 6**: CS deals
- **Naming convention**: `[Company] CS` (e.g., "Earth Motorcars CS", "iDrive1 CS")
- **Stages**: 28 = Needs Attention, 61 = Happy, 26 = AT RISK

## Account ownership (CS)

| Person | Accounts |
|---|---|
| Aaron | Earth Motorcars, plus strategic accounts not delegated |
| Lexie | Karma, iDrive1, A-to-B, Addison Autoplex, Whips, Ally Auto, Go Luxury, all Addison stores **except Earth** |
| Tristen | Streetside Classics, Excellence Auto Direct, Novak Motors, DFW Motorcars, Central AutoHaus, Always A-Way collections |
| Ran | Maverick Motors, Texas Car One |

Geographic clustering helps with stops planning — see [`sales.md`](sales.md) for routing approach.

## Customer health signals (Knack)

A customer is flagged for CS attention if **any** of the following — see [../../systems/knack-fields.md](../../systems/knack-fields.md) for exact fields:

- `field_464 = Yes` AND `field_1035 > 10` (HJD)
- `field_1601` ∈ {`Needs Attention`, `At Risk`}
- `field_1410` ∈ {`Account Manager Follow Up`, `Max Escalation`}
- `field_1428 > 3` (Adjusted Late Invoices)
- `field_1491 > 1` (Aged Invoices)

## Weekly CS workflow (Phase 4 of weekly meeting)

### Part A — Check-in cadence

1. **Cross-reference Knack & Pipedrive**: Current customers in Knack (`field_464 = Yes`) vs. open CS deals in Pipeline 6. Flag mismatches.
2. **Staleness check**: Calculate days since last activity per CS deal. Flag any 60+ days overdue.
3. **HJD flag**: Any customer with `field_1035 > 10`.
4. **Health status review**: `field_1601` = `Needs Attention` or `At Risk`.
5. **Propose weekly check-in batch** (~6-9 stops), prioritized by:
   - AT RISK / Needs Attention stage first
   - Days overdue (most stale first)
   - Account value (`field_459`)
   - Grouped by geography where possible
6. **Delegation**: Assign stops to Tristen, Lexie, Ran, or Aaron based on the ownership table above. **Spread across the week — don't pile on Aaron.**

### Part B — Late invoice review

1. Pull customers where `field_1410` ∈ {`Account Manager Follow Up`, `Max Escalation`}
2. Pull customers where `field_1428 > 3` OR `field_1491 > 1`
3. Combine both lists (deduplicate). Review each one by one.
4. Create Pipedrive activities (escalation follow-up) assigned to either Aaron or the account manager (`field_1438`).

### Invoice escalation rules

| Trigger | Action |
|---|---|
| `field_1410 = "Account Manager Follow Up"` or `"Max Escalation"` | Always flag |
| `field_1428 > 3` | Flag |
| `field_1491 > 1` | Flag |
| Any one trigger | Create Pipedrive activity |

## Pipedrive operating rules (always)

- **Every CS activity must have a deal attached.** Search for `[Company] CS` in Pipeline 6 first.
- **Cap 3 activities/day.** If planning more, defer.
- For full Pipedrive details (user IDs, API quirks, completion-time filtering) → [../../systems/pipedrive.md](../../systems/pipedrive.md).

## See also

- [../../systems/pipedrive.md](../../systems/pipedrive.md) — full Pipedrive config
- [../../systems/knack-fields.md](../../systems/knack-fields.md) — Knack field reference
- [../../people/index.md](../../people/index.md) — delegation matrix
- [sales.md](sales.md) — Pipeline 1 sales workflow
- [operations.md](operations.md) — photographer performance review