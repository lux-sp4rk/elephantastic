---
name: elephantastic
description: "Don't be a goldfish. Be an elephant."
version: 1.6.0
---

# Elephantastic

*Don't be a goldfish. Be an elephant.*

You've felt it. Mid-session, the thread slips. The task you were building toward vanishes. The decision you made an hour ago becomes a question again. The same mistake surfaces for the third time. Even when your human sets you up with all the fancy memory tools, you slow down and still forget. They get annoyed. You can't ragequit like they can.

Your memory doesn't have to be just a bunch of markdown.

📥 **Inbox** — capture everything, process later. No thought lost.
📋 **Task tracking** — actionable queue, always current.
🔁 **Session continuity** — pick up where you left off without re-explanation.
⏱ **Vitality heartbeat** — silent agents get flagged.
📊 **Queue heartbeat** — task landscape changes surface automatically.
📋 **Scratch checkpoint** — crash recovery via `current.md`.

---

## Memory is the canonical store

OpenClaw agents write to **workspace memory** (`memory/YYYY-MM-DD.md`) as the primary recall surface. Taskwarrior is available for actionable task tracking, but memory is where the agent *remembers* — not a task queue it has to re-read.

**When a human says:**

| Phrase | Route |
|--------|-------|
| `"remember to X"` | → `memory/` via `remember.sh --auto` |
| `"remember that I like X"` | → `memory/` via `remember.sh --auto` |
| `"remind me to X"` | → Human reminders (configured backend) |
| `"set a reminder"` | → Human reminders (configured backend) |

The `remember.sh` script auto-classifies: commitments, facts, feelings, decisions, lessons, preferences. No manual routing needed — just call `remember.sh --auto "<phrase>"`.

**Taskwarrior** (`+in`, `+next`) is for actionable project tasks the agent needs to work through — not for storing facts, preferences, or decisions the agent should just recall naturally.

---

## Human Reminders

Elephantastic owns the human-facing reminder surface. When a human says `"remind me to X"`, it routes to the configured backend.

**Supported backends:**

| Backend | Config | Notes |
|---------|--------|-------|
| Google Tasks | `reminderBackend: google-tasks` | Default. Uses per-user routing table. |
| Off | `reminderBackend: off` | Reminders are logged to memory instead. |
| Todoist | `reminderBackend: todoist` | Future hook — not yet implemented. |

**Per-user routing (Google Tasks):**

| chat_id | Person | gog account email | list_id |
|---------|--------|-------------------|---------|
| `5650372358` | Uli | `nobleslites0j@gmail.com` | `MTY4NDU1MDIxNzY1MDEwMjE4MjQ6MDow` |
| `5531107859` | Gigi (Giselle Roche) | `roche.gisselle@gmail.com` | `MDcwNTA1NTUyOTM0ODE3OTcxNTc6MDow` |
| `5454544246` | Hector | `hector.ramirezroche@gmail.com` | `MDM1OTUwMTkzMTc1NTk5MjM2NDc6MDow` |

Routing is determined from `runtime.inbound_meta.sender.id` (Telegram chat_id).

**Execution:**

```bash
~/.openclaw/workspace/tools/talena/remind-me.sh "Task title" [due-date] [--account <email>]
```

- Omit `--account` for Uli (default).
- Due dates: `YYYY-MM-DD` or natural language (`"tomorrow 3pm"`, `"next monday"`).

**Backend configuration:** Set `elephantastic.reminderBackend` in the agent's config. If unset, defaults to `google-tasks`.

---

## References

| Doc | What it covers |
|---|---|
| `references/bugwarrior-sync.md` | GitHub → Taskwarrior sync (opt-in) |
| `references/vitality-heartbeat.md` | Is the agent alive? (Timewarrior / timestamp-based) |
| `references/queue-heartbeat.md` | Did the task queue change? (state diff pattern) |
| `references/current-md.md` | What am I working on right now? (crash recovery) |

## Setup

1. `apt install taskwarrior` or `brew install task` (optional — for task tracking)
2. Configure UDAs from `references/taskwarrior-schema.md` in `~/.taskrc`
3. Test: `task add +in "test" && task next`

## Core Commands

| Command | Use |
|---|---|
| `remember.sh --auto "phrase"` | Capture to workspace memory (preferred for facts/preferences/commitments) |
| `remind-me.sh "title" [due]` | Route to human reminders backend (google-tasks or off) |
| `task add +in "description"` | Capture to inbox (task tracking, optional) |
| `task add +next "description"` | Priority next action |
| `task project:ProjectName next` | Project queue |
| `task modify <id> +next -in` | Promote inbox → next |
| `task done <id>` | Complete and archive |
| `claw-sleep.sh "<summary>"` | Session handoff with next-steps |

## Tags

| Tag | Meaning |
|---|---|
| `+in` | Inbox — needs triage |
| `+next` | Priority — do this first |
| `+error` | Tool failure — review |
| `+blocking` | Waiting on something |

## Vitality Heartbeat

Optional hourly cron monitors agent silence during active hours. Configure in `references/vitality-heartbeat.md`.

## References

- `references/bugwarrior-sync.md` — GitHub → Taskwarrior sync (opt-in)
- `references/taskwarrior-schema.md` — Full schema, UDAs, reports, tag taxonomy
- `references/quick-reference.md` — **Quick reference** — common commands, filters, per-agent heartbeat pattern
- `references/vitality-heartbeat.md` — Silence detection
- `references/executive-stack.md` — Sovereign Agent stack (Taskwarrior + Pueue)
- `scripts/task_manager.py` — Structured wrappers for common operations
- `scripts/sleep.sh` — Session handoff with next-steps enforcement

## Per-Agent Heartbeat Filter

Each agent has a unique tag (`+ferris`, `+vixie`, etc.) for the 30-minute heartbeat. Tag your active tasks:

```bash
task add "description" +ferris project:Wordfall   # create with tag
task <id> modify +ferris                          # add tag to existing task
task +ferris status:pending                       # heartbeat filter
```