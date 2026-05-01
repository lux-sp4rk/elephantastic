---
name: elephantastic
description: "The agent's memory. Task tracking, session continuity, and structured capture using Taskwarrior. Use when: (1) tracking multi-step goals or internal agent tasks, (2) logging decisions or blockers across sessions, (3) managing continuity across sessions, (4) triaging a backlog of agent work."
version: 1.2.0
---

# Elephantastic

## What It Does

Taskwarrior as agent RAM — structured capture, urgency-based prioritization, session continuity.

## Setup

1. `apt install taskwarrior` or `brew install task`
2. Configure UDAs from `references/taskwarrior-schema.md` in `~/.taskrc`
3. Test: `task add +in "test" && task next`

## Core Commands

| Command | Use |
|---|---|
| `task add +in "description"` | Capture to inbox |
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

- `references/taskwarrior-schema.md` — Full schema, UDAs, reports
- `references/vitality-heartbeat.md` — Silence detection
- `scripts/task_manager.py` — Structured wrappers for common operations
- `scripts/sleep.sh` — Session handoff with next-steps enforcement
