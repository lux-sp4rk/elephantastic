---
version: 1.0.0
last_updated: 2026-05-01
---

# Queue Heartbeat

Prevents "stale queue syndrome" — an agent that doesn't notice when the task landscape shifts: new blockers appeared, items completed, priorities shuffled.

Unlike the **vitality heartbeat** (am I alive?), this answers: **did the world change while I was sleeping?**

## Concept

1. Snapshot `task next` IDs to a state file after each run
2. On next run, diff current IDs against state file
3. If delta is empty → `HEARTBEAT_OK` (quiet)
4. If delta has changes → surface what changed, one line per item

## Why This Matters

Bugwarrior syncs GitHub issues, PRs, and external events into Taskwarrior continuously. Without a queue heartbeat, an agent only sees those changes when a human asks or when manually checking. The queue heartbeat closes that gap — passive surveillance of the external world.

## State File

```
memory/<agent>-task-state.txt
```

One ID per line, sorted. Written at end of each heartbeat run.

## Heartbeat Template (runs every 30 min, light context)

```bash
# ── Step 1: Capture current queue ──────────────────────────────────────────
FILTER="status:pending -WAITING (+next or +blocked or +publish or +distribution)"
task next rc.report.next.filter:"$FILTER" rc.report.next.limit:20 \
    2>/dev/null | grep -E "^[0-9]" | awk '{print $1}' | sort \
    > /tmp/<agent>-ids-current.txt

# ── Step 2: Diff against state ───────────────────────────────────────────────
PREV="$HOME/.openclaw/agents/<agent>/workspace/memory/<agent>-task-state.txt"
CURR="/tmp/<agent>-ids-current.txt"

if [[ -f "$PREV" && -s "$CURR" ]]; then
    NEW=$(comm -23 "$CURR" "$PREV")
    GONE=$(comm -13 "$CURR" "$PREV")
    if [[ -n "$NEW" || -n "$GONE" ]]; then
        echo "QUEUE DELTA:"
        [[ -n "$NEW" ]] && echo "  + New: $(echo "$NEW" | tr '\n' ' ')"
        [[ -n "$GONE" ]] && echo "  - Gone: $(echo "$GONE" | tr '\n' ' ')"
    else
        echo "HEARTBEAT_OK"
    fi
else
    echo "HEARTBEAT_OK"
fi

# ── Write state for next run ─────────────────────────────────────────────────
cp "$CURR" "$PREV" 2>/dev/null
```

### Step 3 — Blocked check (always runs, cheap)

```bash
task +blocked status:pending list 2>/dev/null | head -10
```

If output → append to alert.

---

## Response Rules

| Outcome | Reply |
|---|---|
| Nothing surfaced | `HEARTBEAT_OK` only |
| Alert | Lead with delta — one line, specific. No filler. |

---

## Filter Tuning

Replace `FILTER` with tags relevant to the agent's domain:

| Agent | Filter tags |
|---|---|
| Vixie (growth) | `+vixie or +next or +blocked or +publish or +distribution` |
| Ferris (game dev) | `+ferris or +next or +blocked or +enhancement or +epic` |
| Talena (household) | `+talena or +next or +blocked or +uli or +health` |

---

## Initializing the State File

On first run, the diff will find nothing (no prev state). That's correct — just capture and continue. The state file should contain the IDs from the first successful run.

```bash
# Initialize empty state (first run will auto-capture)
touch memory/<agent>-task-state.txt
```

---

## Alert Format

```
QUEUE DELTA:
  + New: 5 12 18
  - Gone: 3 7
BLOCKED: [Hospital bill payment plan — notes:Denied by USPCA]
```

Keep under 3 lines total. Uli hates filler.

---

## Cron Setup

```bash
# Every 30 min, openrouter/free, isolated session
# Name: <Agent> Next — queue watcher
```

Schedule: `every 30 min` | Model: `openrouter/free` | Session: `isolated` | Delivery: `announce → telegram`