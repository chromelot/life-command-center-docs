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
- 9:00-12:00 PM — **Dev block** (current task from dev queue, see `../work/turbo-gear/` and `../work/chrome-lot/` Tasks)
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

- Dev block works on projects selected for the week via the **This Week** checkbox in Tasks (`341f40c2-487b-80ac`)
- Weekly meeting selects 1-3 projects using the One Thing Framework
- Selection pulls only from current quarter's docket

### Deep work vs Workshop vs Admin (the "One Thing" gate)

The dev goal is about **leverage**, not domain. Every project/time punch carries **one** domain label, and the label decides which block it belongs to.

**The gate question:** *If I don't do this, does something important become harder, slower, or impossible?*
- **Yes → Deep Work** (counts toward the dev goal): **Chrome Lot + Turbo Gear + Systems**. Systems = fundamental operating infrastructure that unblocks everything (this dev tracker rebuild, core tooling, a home/office move's setup).
- **No, it's cool / smoother / more "complete" → Workshop** (QoL/hobby — home automation, Plex, media, dashboards). Own **capped budget (~3 hr/wk)**; never in the One Thing block.
- **Personal-life / legal → Admin** (custody, will, disability, house). Standing category, its own block, **not** counted as dev.

**Failure mode this prevents:** tinkering (home automation, Plex) laundering itself into "dev" hours because it used to land in the catch-all Admin bucket. Workshop and Admin are now separate labels with separate blocks and budgets.

**Weekly plan schedules three distinct blocks:** Deep Work (One Thing — CL/TG/Systems), **Workshop** (capped ~3 hr), **Admin** — Workshop and Admin sit **outside** the deep-work block.

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
- No Tasks had This Week checked last week (no project focus)

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
- **High-octane work state → body foreclosure** (named 2026-07-13): engaging work (even just inbound comms) revs Aaron into a hyperfocused, high-output state carrying **physical anxiety + dissociation from the body**, which he cannot easily down-regulate. Two consequences the schedule must handle: **(1) movement must precede the first work/comms touch** — once the switch flips, the gym is gone (see fitness.md → "The ignition rule"); **(2) a deliberate down-shift is needed before state transitions** — especially before Bus pickup (don't carry anxious/dissociated high-octane into Bus evening; parenting wants calm/regulated/present) and before sleep (the high state feeds the night-wired loop above). Down-shift = physical discharge + parasympathetic cue: outdoor walk/sun, breathwork, cold water, light mobility — not just closing the laptop. Under single-parenting, a **dev curfew** is structural, not optional: sleep is load-bearing because Bus depends on Aaron's morning function.
- **Isolation is now the exception, not the default** (2026-07-13): the deep-isolation, cognitively-heavy work (designing the new dashboard/planning paradigm — the phase that produced a 3am night behind a shut office door) is **mostly done.** Remaining dev is **lower-cognitive-load fine-tuning** that tolerates distraction and interruption, so the schedule should **not** reserve monastic isolation blocks by default — fine-tuning slots into fragmented, interruptible single-parent windows. Treat a genuine return of hard-decision work as an **exception that gets a deliberate plan** (a camp/school day, a sitter), never an unbounded late-night lockdown.

> **Dating guardrails are now in `../self/dating.md`.** When a workflow phase explicitly handles dating (weekly Phase 7, monthly Phase 1), load that file instead of duplicating here.

## Weekly plan capacity enforcement (Phase 8: Commit)

The weekly plan must check **before finalizing**:

- **Todoist task count**: If >5 must-do tasks for any single day, flag and trim
- **Pipedrive activities**: If >3 activities suggested for any day, cap at 3
- **Total weekly hours**: Sum of all planned work (dev + ops + field + meetings) **must not exceed 40h**; recommend cuts if over
- **Deferred task check**: Any task deferred 3+ times must route through delegation framework before rescheduling again
- **CS stop delegation**: Weekly CS check-in batch (6-9 stops) **must distribute across team**, not pile on Aaron
- **Tasks**: 1-3 selected via This Week. One Thing Framework applied.

## Intervention protocol

When a trigger fires, the AI **must**:

1. **Name the trigger** specifically: "You have 12 hours of work planned for an 8-hour day."
2. **Recommend what to cut**, not what to add. Default to defer or delegate.
3. **Suggest delegation** by referencing [`../people/index.md`](../people/index.md): "Can [person] handle [task]?"
4. **Never just show the problem** without a specific recommendation.

## Project management workflow

Aaron's projects are tracked in **Tasks DB** (`341f40c2-487b-80ac`), unified across Personal / Chrome Lot / Turbo Gear. CL Internal Projects (`30df40c2`) remains a separate lightweight DB for CL ops improvements.

### Quarterly → Weekly → Daily flow

1. **Quarterly plan:** Projects → Roadmapped + Target Quarter; themes on Quarter Tracker page; ▶ Start top items to Tasks
2. **Monthly plan:** `roadmap-replot-report.mjs`; extend Behind item dates + align quarters; promote into Tasks + `🌙 Month`
3. **Weekly plan:** Tasks `This Week` only (execution)
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