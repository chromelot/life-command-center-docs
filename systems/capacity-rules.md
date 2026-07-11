> **Source:** [`context/systems/capacity-rules.md`](https://github.com/chromelot/life-command-center/blob/main/context/systems/capacity-rules.md) in the private workspace repo. Do not edit this mirror directly.

# Capacity Rules

These rules exist because Aaron's #1 failure mode is overcommitment: trying to do too much, system overload, then doing nothing. Every workflow in this system enforces these limits.

## Core principle

**Do less, better.** If the system is working correctly, Aaron should feel slightly under-committed, not slightly behind.

## Daily limits

### Task execution

- **Max 5 must-do** Todoist tasks per day (P1/P2 priority). If the day looks heavier, defer.
- **Pipedrive activities**: not capped per day. Open activities mirror in real time to the shared Todoist `Pipedrive` project; `Team Evening Defer` (5pm Mon–Sat) rolls anything unfinished forward.
- **Max 1 self-initiated meeting** per day. Protect focus time.
- **If total committed work exceeds 8 hours**, flag as overcommitted. Something must move.

### Time blocks (ideal day structure)

- 5:30-7:00 AM — Reflection, reading, morning routine (protected, no work)
- 7:00-8:30 AM — Gym workout (protected, no distractions)
- On demand — CL Bot `today` / `deal gaps` (replaces retired 9am Morning DM)
- 9:00-12:00 PM — **Dev block** (current task from dev queue, see `../work/turbo-gear/` and `../work/chrome-lot/` Dev Projects)
- 12:00-1:00 PM — Lunch + Todoist quick tasks
- 1:00-4:00 PM — Operations, meetings, Pipedrive activities, field work
- 4:00-5:00 PM — Todoist cleanup, respond to messages, handle subordinate needs
- 5:00 PM — n8n Team Evening Defer auto-fires (Mon–Sat)
- After 5:00 PM — Social, Bus time. Work is done.

### Responsiveness

- Subordinate messages: respond within 2 hours during work hours
- Use **batched response windows** (10 AM, 1 PM, 4 PM) rather than constant checking

## Weekly limits

### Dev block

- Dev block works on projects selected for the week via the **This Week** checkbox in Dev Projects (`341f40c2-487b-80ac`)
- Weekly meeting selects 1-3 projects using the One Thing Framework
- Selection pulls only from current quarter's docket

### Project intake

- Max 2 new projects added to any single project database per week
- New ideas go to inbox first, never directly to active projects (see [routing-rules.md](routing-rules.md))

### Scrub cadence

- Micro-scrubs: 3x/week, 10 min each, one database per session
- Replaces marathon monthly reviews that never happened

## Overcommitment triggers

When detected, the system **must** intervene with a specific cut recommendation, not just data.

### Immediate intervention

- Morning briefing shows >8 hours of committed work today
- >5 Todoist tasks with today's due date at P1/P2
- >10 Pipedrive activities overdue

### Weekly intervention (raised in weekly meeting)

- >5 tasks rescheduled 3+ times this week
- Pipedrive activities not touched in >5 days
- 1:1 meetings overdue >2 weeks for any team member
- No Dev Projects had This Week checked last week (no project focus)

### Monthly intervention (raised in strategy review)

- Any project database not scrubbed in >4 weeks
- >10 items sitting in any inbox unprocessed
- Any value category (Spirituality / Fitness / Work / Social / Admin / Parenting) has zero associated active projects OR is marked Unhealthy in Values DB

## Behavioral patterns & wellness signals

Observations from Aaron's journaling. Use these to guide recommendations and flag when patterns break.

### What works (reinforce these)

- **Minimum-effort gym days**: Gym at intensity 2/10 is dramatically better than skipping. When sick, recovering, or low-energy, recommend "just show up" rather than suggesting a skip. The habit of going matters more than the output.
- **Morning gym before work**: Even when the morning is "off plan" (came home instead of going to work), getting the gym in first is a net positive.
- **Sun exposure + outdoors**: Consistently correlates with better mood. Motorcycle, walks, anything outside.
- **Casual social contact**: Gym conversations, friend hangouts, group events, coffee shop work sessions. Track via Small Talk DB (`121f40c2-487b-802d`). Query for most recent entry's Created Date. Nudge if >7 days since last entry. The signal is "Am I isolated?" not "Am I dating?"

### Watch for (patterns that lead to ruts)

- **Poor sleep → late wake → skip morning routine → guilt → low productivity spiral**: When sleep is bad, **protect the gym habit above all else**. Skip dev time if needed, not the gym.
- **Bus caregiving vs. personal habits**: Taking care of Bus makes it harder to sustain habits like motorcycle rides. When Bus is with Aaron, suggest lower-friction versions of wellness activities rather than skipping entirely.
- **Compulsive transfer pattern**: Minecraft was the most recent compulsive behavior (deleted March 2026). The underlying pattern may transfer to dating apps, porn, doomscrolling, or other dopamine sources. Signs: >1 hour/day on dating apps, late-night swiping instead of sleep, skipping morning routine for a hookup, canceling Bus time for a date. If detected during the weekly meeting: **name it directly**, recommend a 48-hour cooldown from the behavior.
  - **Media-server / self-hosting containment (implemented June 2026 — considered handled):** the media-acquisition transfer is now structurally boxed: (1) a Todoist "Download" section captures acquisition ideas instead of acting on them immediately; (2) the media-acquisition PC stays powered off and is only remoted into on weekends to batch downloads; (3) library organization is done/clean; (4) Freedom blocks the Plex + Audiobookshelf web UIs to add friction against random tinkering. Don't re-open this as an active guardrail to build — just watch for the pattern migrating elsewhere.
- **Post-breakup drift**: After ending a relationship, the freed time can easily become unstructured and dissolve into avoidance. Channel reclaimed evenings into social, creative, or rest — not screens.
- **Afternoon crash + night-wired loop** (observed June 2026): a hard 4-7pm crash (dizzy, drained, uncoordinated, zombie) followed by being wired until ~1am, which wrecks sleep and feeds the next day's crash. Aaron is caffeine-adapted, so it's not caffeine. First **rule out the cheap causes — hydration, electrolytes, calories** (worse in heat) — before treating it as burnout. The lever is breaking the late-night wired stretch (evening downshift + the melatonin/magnesium stack). **Protect sleep the night before any morning that matters** (ties to the relationship guardrail: Warrior + Householder before the Lover).

> **Dating guardrails are now in `../self/dating.md`.** When a workflow phase explicitly handles dating (weekly Phase 7, monthly Phase 1), load that file instead of duplicating here.

## Weekly plan capacity enforcement (Phase 8: Commit)

The weekly plan must check **before finalizing**:

- **Todoist task count**: If >5 must-do tasks for any single day, flag and trim
- **Pipedrive activities**: If >3 activities suggested for any day, cap at 3
- **Total weekly hours**: Sum of all planned work (dev + ops + field + meetings) **must not exceed 40h**; recommend cuts if over
- **Deferred task check**: Any task deferred 3+ times must route through delegation framework before rescheduling again
- **CS stop delegation**: Weekly CS check-in batch (6-9 stops) **must distribute across team**, not pile on Aaron
- **Dev projects**: 1-3 selected via This Week. One Thing Framework applied.

## Intervention protocol

When a trigger fires, the AI **must**:

1. **Name the trigger** specifically: "You have 12 hours of work planned for an 8-hour day."
2. **Recommend what to cut**, not what to add. Default to defer or delegate.
3. **Suggest delegation** by referencing [`../people/index.md`](../people/index.md): "Can [person] handle [task]?"
4. **Never just show the problem** without a specific recommendation.

## Project management workflow

Aaron's projects are tracked in **Dev Projects DB** (`341f40c2-487b-80ac`), unified across Personal / Chrome Lot / Turbo Gear. CL Internal Projects (`30df40c2`) remains a separate lightweight DB for CL ops improvements.

### Quarterly → Weekly → Daily flow

1. **Quarterly plan:** Roadmap → Roadmapped + Target Quarter; themes on Outcomes; ▶ Start top items to Dev Projects
2. **Monthly plan:** `roadmap-replot-report.mjs`; extend Behind item dates + align quarters; promote into Dev Projects + `🌙 Month`
3. **Weekly plan:** Dev Projects `This Week` only (execution)
4. **Daily dev block:** work on `This Week` + Status = In progress

Roadmap layer: [horizon-roadmap.md](horizon-roadmap.md)

### Rules

- Projects without sub-items are unactionable. During reviews, always break them down.
- If a project has been "In progress" with no sub-item movement >2 weeks, flag it.
- Each weekly plan should include at least 1 project (prevents pure ops/reactive weeks).
- Don't overload: max 2 project tasks per day alongside ops work.
- Weekly selection pulls only from current quarter's assigned projects.

## Delegation decision framework

For any task being deferred for the **3rd time**, ask:

1. Does this require Aaron's specific expertise? If no → delegate.
2. Is there someone in the people directory with the right skills? If yes → delegate.
3. Can this be automated via n8n? If yes → automate.
4. Is this actually important? If not → drop it.

Only if all four answers point to "Aaron must do this himself" does the task stay on his plate.

## See also

- [cadences.md](cadences.md) — when each workflow runs
- [../self/dating.md](../self/dating.md) — dating-specific guardrails
- [../people/index.md](../people/index.md) — delegation matrix
- [../router.md](../router.md)