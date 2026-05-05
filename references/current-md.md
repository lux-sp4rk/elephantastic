---
version: 1.0.0
last_updated: 2026-05-01
---

# Current.md — Scratch Checkpoint

*Volatile scratch state for rapid crash recovery.*

## Problem

Agent crashes mid-session. On restart, it has no idea what it was doing. The human has to re-explain. Context is lost.

## Solution

A single `current.md` file per agent — lightweight, always current, machine-readable-ish. Written by the agent during work, not just by cron.

## Location

```
~/.openclaw/agents/<agent>/workspace/memory/current.md
```

## Template

```markdown
# <Agent> — Current

**Updated:** YYYY-MM-DD HH:MM UTC
**Working on:** [one line — what you're doing right now]
**Next step:** [next atomic action]
**Decisions:** [key decisions made this session, if any]
**Blockers:** [if any]
```

## Update Triggers

Write on:
- Before a long-running command
- After making a decision
- Before yielding the session
- When blocked by a dependency
- When the heartbeat fires (writes current snapshot)

## Read Triggers

Read on:
- Session start — wake from crash
- First tool call of a new session
- Before doing anything substantive

## Example

```markdown
# Vixie — Current

**Updated:** 2026-05-01 14:32 UTC
**Working on:** Audit Vixie heartbeat, replaced vague checks with task next diff
**Next step:** Wire Ferris heartbeat cron with same pattern
**Decisions:** State file = one ID per line, not JSON (task export bug with id=0)
**Blockers:** None
```

## Naming Convention

| File | Purpose | Volatility |
|---|---|---|
| `memory/current.md` | What am I doing *now* | High — updated constantly |
| `memory/<agent>-task-state.txt` | What IDs in queue | Low — updated by heartbeat |
| `memory/YYYY-MM-DD.md` | Session log | Permanent — never overwritten |

## Alert Format for Heartbeat Write

When the heartbeat writes to `current.md`:

```
QUEUED: task IDs snapshot taken at HH:MM
CURRENT: <one-liner from Working on>
```

---

*Principle: if the agent disappears right now, what would the human need to recreate this context? That's what goes in current.md.*