---
name: elephantastic
description: "The agent's memory. Task tracking, session continuity, and structured capture using Taskwarrior — never lose the thread."
---

# Elephantastic

## Quick Start

1. **Install Taskwarrior** — `apt install taskwarrior` / `brew install task`
2. **Test capture** — `task add +in "first item"`
3. **Configure UDAs** — copy UDAs from `references/taskwarrior-schema.md` into `~/.taskrc`
4. **Confirm it works** — `task next`
5. **Point the agent** at this skill's `SKILL.md`

The agent's memory. Never forget.

## Why Elephantastic?

Mainstream AI agents "drift" or plateau because they manage state inside a volatile context window. Elephantastic solves the fundamental scaling bottlenecks:

### 1. Structured Persistence vs. Markdown Bloat
Flat files grow noisy and expensive to parse. Taskwarrior is a structured queryable database — `task next`, `task +urgent`, `task project:Wordfall` — keeping the active context window lean and high-signal.

### 2. Multi-Session Continuity
Chatbots forget the "Big Picture" once the session resets. Elephantastic tracks blockers, `+next` actions, and dependencies across days or weeks. The agent wakes up, checks its queue, and makes progress without re-explanation.

### 3. Overload Protection
Native Taskwarrior urgency math replaces LLM waffling. Explicit weights — due dates, dependencies, project priorities — ensure the agent tackles high-leverage work.

### 4. Offline-First & Fully Auditable
No black-box state. The agent's intent lives in a plain-text, version-controllable database. Audit, fix, or reprioritize directly via CLI.

## Core Architecture

| Layer | Tool | Role |
|---|---|---|
| **RAM** | Taskwarrior | Actionable queue, urgency math, session continuity |

Taskwarrior is the single source of truth for what the agent needs to do next.

## References

- `references/taskwarrior-schema.md` — Installation, `.taskrc` config, UDAs, custom reports.
- `references/vitality-heartbeat.md` — Agent silence detection and alerting.
