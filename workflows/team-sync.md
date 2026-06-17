> **Source:** [`.cursor/rules/team-sync.mdc`](https://github.com/chromelot/life-command-center/blob/main/.cursor/rules/team-sync.mdc) in the private workspace repo. Do not edit this mirror directly.

# Team sync

## Table of contents

- [Execution + logging (mandatory)](#execution-logging-mandatory)
- [Philosophy](#philosophy)
- [Pre-flight (silent)](#pre-flight-silent)
- [Diff dimensions](#diff-dimensions)
- [Output contracts](#output-contracts)
  - [Table 1.1-A — Roster Summary (step `1.1`, first turn)](#table-11-a-roster-summary-step-11-first-turn)
  - [Table 1.1-B — Drift Flags (step `1.1`, second turn — list only)](#table-11-b-drift-flags-step-11-second-turn-list-only)
  - [Table 1.2-A — Item Decision (step `1.2`, one row per turn until exhausted)](#table-12-a-item-decision-step-12-one-row-per-turn-until-exhausted)
  - [Table 2.0 — Commit Checklist (FIELD CHECK)](#table-20-commit-checklist-field-check)
- [Walk through decisions](#walk-through-decisions)
- [Completion](#completion)
- [Key rules](#key-rules)

---


This rule activates when Aaron says "team sync", "roster diff", "people sync", or "audit roster". Target duration: 5 minutes.

## Execution + logging (mandatory)

Read `context/workflow-execution.md`, `context/systems/workflow-output-contracts.md`, and `context/systems/workflow-logs.md`.

1. `workflow-progress.mjs init --workflow team-sync`
2. Step `0` — diff pull (silent)
3. Step `1.0` — `workflow-notion-log.mjs create`
4. Step `1.1` — Tables 1.1-A, 1.1-B (one table per turn)
5. Step `1.2` — Table 1.2 per drift item (one decision per turn)
6. Step `2.0` — Table 2.0 commit + `workflow-notion-log complete`

**One drift item decision per turn. Do not batch.**

## Philosophy

Two stores hold roster information today:

1. **Airtable `Admin / Payable Employees`** (`appv05uQhbcO6LAnO` / `tblU2vrGDcFTPXey9`) — single source of truth for who is on the team and what platform IDs they own (Pipedrive, Hubstaff, Todoist, Knack AM, MS Teams). Drives every CL Bot permission category. Schema reference: [`context/systems/airtable-roster.md`](../../context/systems/airtable-roster.md).
2. **`context/people/index.md`** — editorial notes only: performance status, delegation skills, CS coverage, standing 1:1 topics, last-1:1 dates. Cursor-only context for 1:1 prep, weekly meeting, capacity intervention.

Drift between the two is normal — Aaron promotes someone in Airtable but the editorial notes lag, or vice versa. Team sync surfaces the drift in 5 minutes so neither store rots.

## Pre-flight (silent)

1. List Airtable employees (filterByFormula `{Current}=1`):
   ```
   airtable_list_records base_id=appv05uQhbcO6LAnO table_id_or_name=tblU2vrGDcFTPXey9
     fields=["Name","Email","Position","Sales & CS Manager","Photographer Manager",
             "Pipedrive User ID","Hubstaff User ID","Todoist ID","AAD ID","Knack AM Record ID"]
     filterByFormula="{Current}=1"
   ```
2. Read `context/people/index.md`.
3. Read `context/cast.md` (the "Chrome Lot team" subsection).

## Diff dimensions

For each Current employee in Airtable, compare against the editorial notes:

| Check | What to flag |
|---|---|
| Is the person listed in `people/index.md`? | If not → propose adding an entry (status TBD) |
| Is `cast.md` Chrome Lot team subsection up to date? | If a teammate is in Airtable but not in cast.md, propose adding the one-liner |
| Position field matches editorial role description? | E.g. Airtable Position="Account Manager" but `people/index.md` says "Photographer" |
| Sales & CS Manager / Photographer Manager checkboxes match the delegation matrix and 1:1 cadence? | E.g. Lexie has Sales & CS Manager but `people/index.md` lists her under Operations only |
| Inactive in editorial but `Current=1` in Airtable? | Propose flipping `Current` off + moving editorial note to Inactive section |
| Inactive in Airtable but still in `people/index.md` Active section? | Propose moving editorial note to Inactive section |

## Output contracts

### Table 1.1-A — Roster Summary (step `1.1`, first turn)

| Field | Value | Source |
|-------|-------|--------|
| Session date | | CT today |
| Airtable Current count | | Airtable pull |
| people/index.md Active count | | Markdown count |
| Drift items total | | Diff |

### Table 1.1-B — Drift Flags (step `1.1`, second turn — list only)

| Name | Drift type | Airtable | Editorial | Recommendation |
|------|------------|----------|-----------|----------------|
| | New / Role / Inactive / Missing ID | | | |

Drift types from § Diff dimensions above.

### Table 1.2-A — Item Decision (step `1.2`, one row per turn until exhausted)

| Field | Value |
|-------|-------|
| Name | |
| Drift type | |
| Action | Update Airtable / Update editorial / Both / Skip |
| Approved? | pending |

One AskQuestion per row. After all rows decided → `advance` to `2.0`.

### Table 2.0 — Commit Checklist (FIELD CHECK)

| Item | Pass |
|------|------|
| Approved Airtable writes executed | |
| Approved markdown edits done | |
| Summary in Workflow Session Log | |

## Walk through decisions

For each flagged item, ask "update Airtable, update editorial notes, both, or skip?". Default action by category:

- **New in Airtable** → propose editorial entry, ask for performance status guess (default: "Active — no Notion record yet"), then write to `people/index.md` with `personal_notion_update_page` (or just edit the markdown directly).
- **Role/category drift** → if Airtable Position is wrong, fix via `airtable_update_record`. If editorial description is stale, update markdown.
- **Inactive drift** → flip `Current` checkbox in Airtable (or move the markdown block to Inactive section).
- **Missing platform IDs** → use the bot: `admin set-pd <email> <id>` / `admin set-hubstaff <email> <id>` / `admin set-aad <email> <id>`. Those write Airtable directly + refresh the bot cache.

## Completion

1. Execute approved changes (Airtable writes via MCP, markdown edits via StrReplace).
2. If any Airtable writes happened, prompt Aaron: "Run `admin refresh-roster` in Teams so the bot picks up the changes."
3. Log: "[Date]: Team sync — added X, updated Y, flagged Z for follow-up." Append to `history/daily-log.md` if it exists.

## Key rules

- **Never** pull SSN / DL Number / DL Picture / Mailing Address / Pay Events from Airtable into Cursor context. The PII allowlist in `n8n/shared/airtable-roster.mjs` exists for a reason — respect it on the read side too.
- 5 minutes max. If a teammate's situation deserves a real conversation (performance plan, role change), defer to 1:1 prep or weekly meeting.
- Don't silently change `Sales & CS Manager` / `Photographer Manager` checkboxes — those gate bot permissions. Always ask first.
- The bot's Sunday 6 PM CT Roster Reconcile already diffs Airtable against PD active users and Hubstaff members. Don't re-do that; team sync focuses on Airtable ↔ editorial markdown drift only.