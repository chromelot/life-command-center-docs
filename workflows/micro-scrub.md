> **Source:** [`.cursor/rules/micro-scrub.mdc`](https://github.com/chromelot/life-command-center/blob/main/.cursor/rules/micro-scrub.mdc) in the private workspace repo. Do not edit this mirror directly.

# Micro-Scrub

This rule activates when Aaron says "scrub", "micro-scrub", "review projects", or "scrub roadmap". Target duration: 10 minutes.

## Philosophy

Micro-scrubs replace the dreaded marathon monthly project reviews that never happen. By reviewing ONE database for 10 minutes, 3x/week, every project database gets attention every week without the overwhelm.

## Pre-Flight (silent)

Load via the router:

- `context/systems/cadences.md` — rotation schedule (the canonical day/database map)
- `context/systems/capacity-rules.md` — capacity intervention triggers, delegation framework
- `context/self/values.md` — six categories, used for the alignment summary

For DB IDs, check `context/systems/notion-databases.md`. Today's database based on day of week (per `cadences.md` Micro-scrub rotation):

- **Monday**: Dev Projects — Turbo Gear (filter Type=Turbo Gear)
- **Wednesday**: Chrome Lot Internal Projects
- **Friday**: Dev Projects — Personal (filter Type=Personal)
- **Other days**: Ask which database, or pick whichever was last missed

## Data Pull

Query the day's database using the appropriate Notion MCP. **DB IDs come from `context/systems/notion-databases.md` — do not hardcode.** Apply the relevant Type filter for Dev Projects.

## Pre-Analysis

Before presenting to Aaron, analyze the items:

1. **Stale items**: Projects with `Last edited time` >30 days ago
2. **Completed items**: Status = "Done" that haven't been archived
3. **Stuck items**: Status = "In progress" but no recent edits
4. **New/unstructured items**: Status = "Not Started" with minimal properties filled in
5. **Values alignment**: For each active project, which value category does it serve? (Spirituality/Fitness/Work/Social/Admin/Parenting)

## Present to Aaron

```
MICRO-SCRUB: [Database Name] -- [Date]
[X total items, Y active, Z stale]

RECOMMEND ARCHIVE (stale >30 days or completed):
- [Project name] -- last edited [date] -- [reason to archive]
- ...

NEEDS ATTENTION (stuck or unstructured):
- [Project name] -- Status: [status] -- Missing: [what's missing, e.g., priority, difficulty, decomposition]
- ...

DELEGATION CANDIDATES:
- [Project name] -- Could [person from people-directory] handle this?
- ...

VALUES ALIGNMENT:
- Spirituality: [X projects]
- Fitness: [X projects]
- Work: [X projects]
- Social: [X projects]
- Admin: [X projects]
- Parenting: [X projects]
- UNALIGNED: [projects that don't clearly serve any value category]
```

## Walk Through Decisions

For each flagged item, get a quick decision:
- **Archive?** --> `personal_notion_update_page` with `archived: true`
- **Update status?** --> Update via Notion API
- **Add structure?** --> Discuss difficulty, decomposition, delegation candidate. Update properties.
- **Keep as-is?** --> Move on (don't overthink)

## Time Enforcement

- Set a mental timer. After 10 minutes, say: "We're at 10 minutes. [X items reviewed, Y decisions made]. Want to continue for 5 more minutes or close out?"
- Never exceed 15 minutes. If the database has too many items to review, focus on the flagged ones only.
- If Aaron starts going deep on strategy for a single project, redirect: "This deserves its own focused session. I'll note it for the weekly meeting. Let's keep moving."

## Completion

After decisions are made:
1. Execute all approved changes via Notion API
2. Log what was done: "[Date]: Scrubbed [DB name]. Archived X, updated Y, flagged Z for weekly."
3. Append to `history/daily-log.md`

## Key Rules

- ONE database per session. Never try to scrub multiple.
- Decisions should be quick: archive, update, or skip. No deep strategy.
- Cap at 15 minutes absolute maximum.
- If a project needs deep discussion, defer to weekly meeting.
- Always check values alignment -- this is how you catch drift.