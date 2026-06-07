> **Source:** [`.cursor/rules/person-checkin.mdc`](https://github.com/chromelot/life-command-center/blob/main/.cursor/rules/person-checkin.mdc) in the private workspace repo. Do not edit this mirror directly.

# Person Check-In Prep

This rule activates when Aaron says "prep for [name]", "1:1 prep", "meeting with [name]", or "check in with [name]". Target duration: 5 minutes prep.

## Pre-Flight (silent)

Load via the router:

- `context/people/index.md` — master directory and delegation matrix. Find the person's entry; note their systems, last check-in date, and standing 1:1 topics.
- `context/systems/notion-databases.md` — for any Notion DB IDs needed (CL Tasks, etc.)
- `context/systems/pipedrive.md` — for Pipedrive user IDs if pulling activity by owner

## Identify the Person

Extract the person's name from Aaron's message. Look them up in the people directory.

If not found: "I don't have [name] in the people directory yet. Want to add them? I need: role, check-in frequency, and what systems their work is tracked in."

## Data Pull (parallel)

Based on the person's systems listed in the people directory:

### If they have Todoist tasks
- `todoist_get_tasks` filtered to their project or assigned to them
- Show: open tasks, overdue items, recently completed

### If they have Work Notion CL Tasks
- `notion_query_database` on CL Tasks (DB ID in `context/systems/notion-databases.md`) filtered by assignee
- Show: assigned tasks by status, overdue items, blocked items

### If they have Pipedrive activity
- `pipedrive_search` for the person
- Show: recent activity, deal involvement, last interaction

### If they have Process Street workflows
- Check for workflow runs assigned to them
- Show: active workflows, completion rates

## Generate Talking Points

Based on the data pulled:

```
1:1 PREP: [Name] -- [Date]
Last check-in: [date from people directory]

THEIR CURRENT WORKLOAD:
- [X] open tasks, [Y] overdue, [Z] completed recently
- Highest priority: [task name]
- Blocked on: [task name, if any]

SUGGESTED TALKING POINTS:
1. [Based on overdue items: "The [task] is X days overdue -- what's blocking it?"]
2. [Based on workload: "You have X items assigned. How's the load feeling?"]
3. [Based on standing topics from people directory]
4. [Based on recent completions: "Nice work on [task]. How did that go?"]

DELEGATION OPPORTUNITIES:
- [Tasks from Aaron's plate that match this person's skills]

ACTION ITEMS TO DISCUSS:
- [Items from last meeting notes, if available]
```

## During the Meeting

If Aaron asks for help during the actual meeting:
- Take notes in real-time
- Create tasks in the appropriate system as they're discussed
- Update the person's last check-in date in `people-directory.md`

## After the Meeting

1. Update `context/people/index.md` with today's date as last check-in
2. Create any agreed-upon tasks via Todoist or Notion
3. Note any follow-up items for the next check-in

## Key Rules

- Prep should take 5 minutes max. Present the summary, don't deep-dive.
- Always include delegation opportunities -- this is a key moment to hand off work.
- Update the last check-in date so the n8n reminder system stays accurate.
- If the person isn't in `context/people/index.md`, add them. This is how the system grows.