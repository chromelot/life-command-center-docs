> **Source:** [`.cursor/rules/inbox-triage.mdc`](https://github.com/chromelot/life-command-center/blob/main/.cursor/rules/inbox-triage.mdc) in the private workspace repo. Do not edit this mirror directly.

# Inbox Triage

This rule activates when Aaron says "triage inbox", "process ideas", "inbox triage", or "clear inbox". Target duration: 10 minutes.

## Pre-Flight (silent)

Load via the router:

- `context/systems/routing-rules.md` — where new items belong (ideas → which inbox; promoted items → which project DB)
- `context/systems/capacity-rules.md` — weekly project-intake limit (max 2 new projects/week)
- `context/systems/notion-databases.md` — DB IDs for the inboxes and target project DBs

Determine which inbox to process:
- **Odd weeks**: App Idea Inbox
- **Even weeks**: AI Implementation Ideas
- Or ask Aaron which inbox to process

## Data Pull

Query the inbox database: `personal_notion_query_database` on the selected DB (ID from `context/systems/notion-databases.md`), limit 10, sorted by created time ascending (oldest first).

## Pre-Analysis

For each item, AI assesses:
1. **Overlap**: Does this duplicate or relate to an existing project in any database?
2. **Category suggestion**: What category fits? (for AI Ideas: Knack/Sales/Training/Admin/QA/Design/Automation/Hiring/Dispatching)
3. **Difficulty estimate**: XS/S/M/L/XL based on description
4. **Delegation potential**: Could someone else execute this?
5. **Values alignment**: Which value category does this serve? (Spirituality/Fitness/Work/Social/Admin/Parenting)
6. **Recommendation**: Route to project DB, keep in inbox for later, or archive

## Present to Aaron

Show top 5 items (oldest first):

```
INBOX TRIAGE: [Inbox Name] -- [Date]
[X total items in inbox]

1. "[Idea name]"
   AI Assessment: [Category], Difficulty: [S/M/L], Serves: [Archetype]
   Overlaps with: [existing project or "none"]
   Recommendation: [Route to Dev Projects (TG/CL/Personal) / Route to CL Internal / Delegate to [person] / Archive / Hold]

2. ...
```

## For Each Item, Get a Decision

- **Route to project DB**: Create entry in the target database with suggested properties, then archive the inbox item
- **Delegate**: Create a task in the appropriate system (Todoist or Work Notion Tasks) for the delegate, archive inbox item
- **Archive**: Mark as archived in Notion
- **Hold**: Leave in inbox for next triage cycle
- **Needs more thought**: Note for weekly meeting discussion

## Execution

After all 5 decisions:
1. Execute routing via Notion API (create entries, archive originals)
2. Execute delegations via Todoist or Notion
3. Log: "[Date]: Triaged [Inbox name]. Routed X, archived Y, held Z."
4. If >5 items remain in inbox, note: "[X items remaining. Next triage in 2 weeks.]"

## Key Rules

- Process exactly 5 items per session (or all if <5 remain)
- Oldest items first -- prevent ideas from rotting
- Max 2 items can be routed to active project databases per session (capacity limit)
- If Aaron wants to go deep on an idea, redirect: "Let's capture the structure now and plan it properly in the weekly meeting."
- 10 minutes max. Quick decisions.