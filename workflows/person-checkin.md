> **Source:** [`.cursor/rules/person-checkin.mdc`](https://github.com/chromelot/life-command-center/blob/main/.cursor/rules/person-checkin.mdc) in the private workspace repo. Do not edit this mirror directly.

# Person Check-In Prep

## Table of contents

- [Execution + logging (mandatory)](#execution-logging-mandatory)
- [Pre-Flight (silent)](#pre-flight-silent)
- [Identify the Person](#identify-the-person)
- [Data Pull (parallel)](#data-pull-parallel)
  - [If they have Todoist tasks](#if-they-have-todoist-tasks)
  - [If they have Work Notion CL Tasks](#if-they-have-work-notion-cl-tasks)
  - [If they have Pipedrive activity](#if-they-have-pipedrive-activity)
  - [If they have Process Street workflows](#if-they-have-process-street-workflows)
- [Output contracts (steps `1.1a`–`1.1c` — one table per turn)](#output-contracts-steps-11a11c-one-table-per-turn)
  - [Table 1.1-A — Session Header + Workload](#table-11-a-session-header-workload)
  - [Table 1.1-B — Talking Points](#table-11-b-talking-points)
  - [Table 1.1-C — Delegation + Follow-ups](#table-11-c-delegation-follow-ups)
  - [Table 2.0 — Commit (FIELD CHECK)](#table-20-commit-field-check)
- [During the Meeting](#during-the-meeting)
- [After the Meeting](#after-the-meeting)
- [Key Rules](#key-rules)

---


This rule activates when Aaron says "prep for [name]", "1:1 prep", "meeting with [name]", or "check in with [name]". Target duration: 5 minutes prep.

<a id="execution-logging-mandatory"></a>
## Execution + logging (mandatory)

Read `context/workflow-execution.md`, `context/systems/workflow-output-contracts.md`, and `context/systems/workflow-logs.md`.

1. `workflow-progress.mjs init --workflow person-checkin --person "<name>"`
2. Step `0` — identify person + parallel data pull (silent)
3. Step `1.0` — `workflow-notion-log.mjs create --person "<name>"`
4. Step `1.1a` — Table 1.1-A (workload); `1.1b` — Table 1.1-B (talking points); `1.1c` — Table 1.1-C (delegation)
5. Step `2.0` — Table 2.0 commit + `workflow-notion-log complete` (Session Complete = Complete)

<a id="pre-flight-silent"></a>
## Pre-Flight (silent)

Load via the router:

- `context/people/index.md` — master directory and delegation matrix. Find the person's entry; note their systems, last check-in date, and standing 1:1 topics.
- `context/systems/notion-databases.md` — for any Notion DB IDs needed (CL Tasks, etc.)
- `context/systems/pipedrive.md` — for Pipedrive user IDs if pulling activity by owner

<a id="identify-the-person"></a>
## Identify the Person

Extract the person's name from Aaron's message. Look them up in the people directory.

If not found: "I don't have [name] in the people directory yet. Want to add them? I need: role, check-in frequency, and what systems their work is tracked in."

<a id="data-pull-parallel"></a>
## Data Pull (parallel)

Based on the person's systems listed in the people directory:

<a id="if-they-have-todoist-tasks"></a>
### If they have Todoist tasks
- `todoist_get_tasks` filtered to their project or assigned to them
- Show: open tasks, overdue items, recently completed

<a id="if-they-have-work-notion-cl-tasks"></a>
### If they have Work Notion CL Tasks
- `notion_query_database` on CL Tasks (DB ID in `context/systems/notion-databases.md`) filtered by assignee
- Show: assigned tasks by status, overdue items, blocked items

<a id="if-they-have-pipedrive-activity"></a>
### If they have Pipedrive activity
- `pipedrive_search` for the person
- Show: recent activity, deal involvement, last interaction

<a id="if-they-have-process-street-workflows"></a>
### If they have Process Street workflows
- Check for workflow runs assigned to them
- Show: active workflows, completion rates

<a id="output-contracts-steps-11a11c-one-table-per-turn"></a>
## Output contracts (steps `1.1a`–`1.1c` — one table per turn)

<a id="table-11-a-session-header-workload"></a>
### Table 1.1-A — Session Header + Workload

| Field | Value | Source |
|-------|-------|--------|
| Person | | User message / people index |
| Session date | | CT today |
| Last check-in | | `people/index.md` |
| Open tasks | | Todoist / CL Tasks pull |
| Overdue | | Pull |
| Completed recently | | Pull |
| Highest priority | | Pull or `—` |
| Blocked on | | Pull or `—` |

<a id="table-11-b-talking-points"></a>
### Table 1.1-B — Talking Points

| # | Topic | Prompt |
|---|-------|--------|
| 1 | Overdue | |
| 2 | Workload | |
| 3 | Standing topic | people index |
| 4 | Recent win | |

Minimum 3 rows; derive from pull data. No generic filler.

<a id="table-11-c-delegation-follow-ups"></a>
### Table 1.1-C — Delegation + Follow-ups

| Type | Item | Notes |
|------|------|-------|
| Delegation opportunity | | |
| Action from last 1:1 | | or `—` |

<a id="table-20-commit-field-check"></a>
### Table 2.0 — Commit (FIELD CHECK)

| Item | Pass |
|------|------|
| Summary in Workflow Session Log | |
| Person field set | |
| Key talking points captured | |

<a id="during-the-meeting"></a>
## During the Meeting

If Aaron asks for help during the actual meeting:
- Take notes in real-time
- Create tasks in the appropriate system as they're discussed
- Update the person's last check-in date in `people-directory.md`

<a id="after-the-meeting"></a>
## After the Meeting

1. Update `context/people/index.md` with today's date as last check-in
2. Create any agreed-upon tasks via Todoist or Notion
3. Note any follow-up items for the next check-in

<a id="key-rules"></a>
## Key Rules

- Prep should take 5 minutes max. Present the summary, don't deep-dive.
- Always include delegation opportunities -- this is a key moment to hand off work.
- Update the last check-in date so the n8n reminder system stays accurate.
- If the person isn't in `context/people/index.md`, add them. This is how the system grows.