# Taskwarrior Quick Reference

*For agents using the elephantastic GTD skill. Based on [taskwarrior.org/docs](https://taskwarrior.org/docs/examples/).*

---

## 30-Second Tutorial

```bash
task add "Read Taskwarrior documents later"    # create task
task add priority:H "Pay bills"                # high priority
task next                                      # see your queue
task 2 done                                    # complete task 2
task                                           # list all
task 1 delete                                  # delete task 1
```

---

## Creating Tasks

```bash
# Basic
task add "description"

# With due date
task add "Pay rent" due:eom

# With project and priority
task add project:Goals priority:H "Strategic goal"

# Multi-line description (use "> " for continuation)
task add "Five syllables here
> Seven more syllables there
> Are you happy now?"

# Add tags
task add +ferris "Wordfall particle burst" project:Wordfall
```

---

## Viewing Tasks

| Command | What it shows |
|---|---|
| `task next` | Default actionable queue (sorted by urgency) |
| `task list` | All pending tasks |
| `task +in list` | Inbox — needs triage |
| `task project:Wordfall list` | Filter by project |
| `task +error list` | Filter by tag |
| `task /pattern/ list` | Full-text search in description + annotations |
| `task projects` | All projects used |
| `task tags` | All tags used |

---

## Filters (Algebraic)

```bash
# Created in last 4 days
task entry.after:today-4days list

# Created yesterday
task entry:yesterday list

# Completed between dates
task end.after:2015-05-01 and end.before:2015-05-31 completed

# This project OR that project
task project:Wordfall or project:Bloodwork list

# Complex: project X with high OR medium priority
task project:Wordfall and \( priority:H or priority:M \) list

# Tasks with no project
task project: list

# Tasks that have any tag
task tags.any: list

# Tasks that have neither tag
task +this -that list
```

---

## Modifying Tasks

```bash
# Change priority
task <id> modify priority:H

# Change project
task <id> modify project:Wordfall

# Add/remove tags
task <id> modify +ferris
task <id> modify -in

# Wait (snooze until date)
task <id> modify wait:2026-05-15

# Start task (time tracking, if timewarrior installed)
task start <id>

# Stop time tracking
task stop <id>

# Move to next action (remove inbox, add next)
task <id> modify +next -in
```

---

## Annotations (Notes on Tasks)

```bash
task annotate <id> "note or context"
task annotate <id> "recurrence #2 — 2026-05-01"   # track repeated errors
```

---

## Completing & Deleting

```bash
task done <id>      # complete
task delete <id>    # permanent delete
```

---

## DOM — Scripting Access

Get a single field value (useful for scripts/wrappers):
```bash
task _get <id>.description        # description only
task _get <id>.entry <id>.modified   # timestamps
task _get <id>.due.week           # week number
```

---

## Custom Reports (add to ~/.taskrc)

```ini
report.learnings.description=Error & Learning Log
report.learnings.columns=id,tags,priority,description,entry.age
report.learnings.filter=project:Internal.Learnings status:pending
report.learnings.sort=entry-

report.in.description=Inbox (unprocessed)
report.in.columns=id,description,project,tags,entry.age
report.in.filter=status:pending +in
report.in.sort=entry-
```

Usage: `task learnings` or `task in`

---

## Virtual Tags (Built-in Filters)

| Tag | Meaning |
|---|---|
| `+DUETODAY` | Due today |
| `+WEEK` | Due this week |
| `+OVERDUE` | Past due date |
| `+DUE` | Has any due date |
| `+BLOCKED` | Has dependencies |
| `+CHILD` | Has subtasks |

---

## Per-Agent Heartbeat Filter Pattern

Each agent has a unique tag for heartbeat surface:
```bash
task +ferris status:pending      # Ferris's queue
task +vixie status:pending       # Vixie's queue
task +marina status:pending      # Marina's queue
```

Tag your active tasks: `task <id> modify +<agentname>`
Heartbeat replies `HEARTBEAT_OK` if the filter returns empty.