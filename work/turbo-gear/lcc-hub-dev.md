> **Source:** [`context/work/turbo-gear/lcc-hub-dev.md`](https://github.com/chromelot/life-command-center/blob/main/context/work/turbo-gear/lcc-hub-dev.md) in the private workspace repo. Do not edit this mirror directly.

# Turbo Gear — development from lcc-hub

Aaron implements Turbo Gear from the **Life Command Center** Cursor workspace. The app code lives on disk at a fixed path; LCC agents edit those files by absolute path and follow TG's own Cursor rules (cloned from GitHub).

## Paths

| What | Path |
|---|---|
| **TG project root** (open in Cursor for full TG rules) | `C:\Tools\lcc\Turbo Gear` |
| Frontend (`turbo-gear` repo) | `C:\Tools\lcc\Turbo Gear\turbo-gear-main` |
| Backend (`turbo-gear-api` repo) | `C:\Tools\lcc\Turbo Gear\turbo-gear-api` |
| Shared rules/docs/scripts | `C:\Tools\lcc\Turbo Gear\turbo-gear-workspace` |
| TG agent entry point | `C:\Tools\lcc\Turbo Gear\AGENTS.md` |
| TG Cursor rules (junction) | `C:\Tools\lcc\Turbo Gear\.cursor\rules\` |

Legacy path on the Turbo Gear PC: `c:\Users\hoege\Documents\Turbo Gear Repo\` — **not** on lcc-hub.

## GitHub org

All repos: `torchcommercialmedia` — `turbo-gear` (frontend), `turbo-gear-api`, `turbo-gear-workspace`.

**Branching (solo dev, no human reviewer):** `main` is the **live app** — **default is to commit directly to `main`** (auto-deploys; no users yet). CI must pass; **no required review, no CODEOWNERS**. Use a `feat/*` branch only to preview a risky change (Vercel preview) before self-merging. `dev`/dev.turbogear.com is the **future preview sandbox**, optional today. Reviewer = CI + Bugbot + verify loop. See TG `branching-and-prs.mdc` + `environments.mdc`.

## Dev stack commands

From `C:\Tools\lcc\Turbo Gear`:

```powershell
.\scripts\dev-up.ps1 -SkipSync          # first run: skip prod DB sync until .env has MONGO_URI
.\scripts\dev-up.ps1 -SyncDb            # after prod URI is set
.\scripts\dev-down.ps1
.\scripts\dev-status.ps1
```

| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| API | http://localhost:9090/api |
| Swagger | http://localhost:9090/api/docs |
| BullStudio | http://localhost:4000 |

## Environment files (local only — never commit)

| File | Purpose |
|---|---|
| `C:\Tools\lcc\Turbo Gear\.env` | **Production** `MONGO_URI` for `sync-db.ps1` only |
| `C:\Tools\lcc\Turbo Gear\turbo-gear-api\.env` | Local API config (Docker Mongo/Redis) |
| `C:\Tools\lcc\Turbo Gear\turbo-gear-main\.env.local` | `NEXT_PUBLIC_API_ENDPOINT=http://localhost:9090` |

Template for workspace root: `C:\Tools\lcc\Turbo Gear\.env.example`

## Working from LCC vs opening TG workspace

| Mode | When | How |
|---|---|---|
| **LCC session** (default) | Planning + TG implementation in one chat | Agent reads `.cursor/rules/turbo-gear-dev.mdc`, edits files under `C:\Tools\lcc\Turbo Gear\`, loads TG rules from junction path on demand |
| **TG workspace** | Deep TG-only session with all rules auto-loaded | Cursor → Open Folder → `C:\Tools\lcc\Turbo Gear` |

Multi-root option: open `C:\Tools\lcc\life-command-center-with-turbo-gear.code-workspace` (both LCC + TG roots).

## RC / My Machines worker (lcc-hub)

The **lcc-hub** Cursor worker must expose **both** workspace roots or RC agents get *transient infrastructure errors* on TG file reads/writes:

| Role | Path |
|---|---|
| Assignment (primary) | `C:\Users\Aaron\Resilio Sync\Life Command Center` |
| Execution (TG) | `C:\Tools\lcc\Turbo Gear` |

Configured in `scripts/lcc-hub/cursor-agent-worker.ps1` via `agent worker --worker-dir … --worker-dir … start`. After changing worker scripts, run `register-cursor-worker.ps1` and restart the worker (or let the watchdog recover).

Verify: `agent worker debug` should list **Execution workspaces:** with both paths.

### Supervision model (why it used to crash-loop)

The **node bridge** is the durable unit, not the powershell launcher. `agent worker start` daemonizes into a long-lived node process; the `cursor-agent-worker.ps1` wrapper is a **one-shot launcher** (acquires a single-instance mutex, cleans stale bridges, launches, then blocks/exits — that's normal).

Two rules keep it stable:

- **Worker task `RestartCount = 0`.** Task Scheduler must never auto-restart the wrapper — doing so relaunches every cycle and kills active RC sessions. (This was the historical every-1-2-min thrash.)
- **Watchdog is visibility-driven.** `cursor-agent-worker-watchdog.ps1` (every **1 min**) uses the Cursor `worker debug` **visibility probe** as ground truth. If visible → do nothing. If **no node bridge** → relaunch immediately via `Start-ScheduledTask`. If a bridge exists but isn't visible → debounce 2 cycles (~2 min), then full restart.

### Failure mode: bridge running but invisible (`total=0`)

The local node process can report "Worker is now running" while Cursor's visibility probe shows `total=0` — RC sessions then fail with environment/shell errors. Common causes on lcc-hub:

| Cause | Auto-fix |
|---|---|
| Stale worker ID in `~/.cursor/agent-cli-state.json` | Watchdog **deep repair** clears it after 2 failed restarts |
| `useHttp1ForAgent=true` in `~/.cursor/cli-config.json` (breaks HTTP/2 bridge) | Launcher + deep repair reset to `false` |
| Outdated `cursor-agent` CLI | Launcher runs `agent update` once per 24h |
| Expired auth | **Manual:** `agent login` on lcc-hub (watchdog sets `C:\Tools\lcc\cursor-agent-needs-login.flag`) |

**Deep repair** (watchdog): reset HTTP/2 config → clear worker registration state → rate-limited `agent update` → relaunch via scheduled task. Typical recovery ≤ **2–4 min** for bridge issues; auth expiry requires a one-time browser login on lcc-hub.

Logs: `C:\Tools\lcc\cursor-agent-worker.log` (launcher) and `C:\Tools\lcc\cursor-agent-worker-watchdog.log` (should read `OK: worker visible to Cursor` when healthy). After script changes: `register-cursor-worker.ps1`.

## Agent read order (TG implementation)

1. LCC `.cursor/rules/turbo-gear-dev.mdc` (this workspace)
2. `C:\Tools\lcc\Turbo Gear\AGENTS.md`
3. Relevant TG `.cursor/rules/*.mdc` for the task (branching, environments, feature-queue, etc.)
4. LCC `context/work/turbo-gear/overview.md` for strategic context only when scope/priority matters

## Re-setup / update clones

```powershell
cd "C:\Tools\lcc\Turbo Gear"
.\turbo-gear-workspace\setup.ps1 -SkipClone    # refresh junctions
# or pull each repo:
git -C turbo-gear-api pull
git -C turbo-gear-main pull
git -C turbo-gear-workspace pull
```

## See also

- [overview.md](overview.md) — product strategy
- [architecture.md](architecture.md) — stack summary (TG repo `docs/` is richer)
- [dev-onboarding.md](dev-onboarding.md) — PR guardrails, green/red zones
- TG `docs/getting-started.md` — canonical onboarding in the workspace repo