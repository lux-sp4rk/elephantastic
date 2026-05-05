# Bugwarrior Sync — GitHub Issues/PRs → Taskwarrior

*Opt-in extension for elephantastic. Syncs GitHub issues and pull requests into your Taskwarrior task queue, including automatic closure of issues referenced in merged PRs.*

---

## What It Does

- Pulls open issues and PRs from GitHub repos into Taskwarrior as tasks
- Pulls merged/closed PRs and marks their tasks complete
- Annotates each task with the GitHub URL, body excerpt, and labels
- Bidirectional: completed tasks stay synced; GitHub closures propagate

## Setup

### 1. Install Bugwarrior

```bash
# Linuxbrew (recommended on headless Linux)
brew install bugwarrior

# or via pip
pip install bugwarrior
```

### 2. Configure `.bugwarriorrc`

Create `~/.bugwarriorrc`:

```ini
[general]
home = ~/.local/share/bugwarrior
db = ~/.local/share/bugwarrior/bugwarrior.db
annotation_comments = False
annotation_length = 500

[github.<your-login>]
service = github
github.token = @oracle:eval:gh auth token    # or a GitHub PAT
github.login = <your-github-handle>

# Sync all repos under a org or user
include_repos = <org-or-user>/.*

# Sync settings
github.body_length = 500
github.state_length = 10
github.include_pull_requests = True
github.include_closed = True
github.include_closed_age = 14

# Import labels as Taskwarrior tags (enables agent routing)
github.import_labels_as_tags = true

# Default tag
tags = github
```

> **Note on `github.token`:** `@oracle:eval:gh auth token` resolves the GitHub CLI token at runtime. Replace with a GitHub Personal Access Token if not using `gh`.
> **Note on `include_closed_age`:** Tasks for closed issues/PRs older than N days are removed from Taskwarrior on sync. Set `14` to keep recently-closed items.

### 3. Create the sync script

```bash
#!/usr/bin/env bash
set -euo pipefail

BUGWARRIOR="/home/linuxbrew/.linuxbrew/bin/bugwarrior"
LOGFILE="$HOME/.local/share/bugwarrior/sync.log"
mkdir -p "$(dirname "$LOGFILE")"

export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"

echo "[$(date '+%Y-%m-%d %H:%M:%S')] Starting bugwarrior sync..." >> "$LOGFILE"

GITHUB_WEBHOOK=1 "$BUGWARRIOR" pull >> "$LOGFILE" 2>&1 &&
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ✓ Sync complete" >> "$LOGFILE" ||
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ✗ Sync failed" >> "$LOGFILE"
```

> The `GITHUB_WEBHOOK=1` flag enables webhook-aware behavior. Without it, bugwarrior pulls everything fresh on each run. With it, it respects what was already synced.

### 4. Add to cron

```crontab
# Run every 15 minutes
*/15 * * * * /path/to/bugwarrior_sync.sh
```

> Cron jobs need `PATH` extended to include your bugwarrior binary. The wrapper script above handles this internally.

### 5. Test it

```bash
# First sync — dry run
bugwarrior pull --debug

# Full sync
GITHUB_WEBHOOK=1 bugwarrior pull
```

## Task Naming

Bugwarrior creates tasks with prefixed IDs:

| GitHub Type | Task Description Prefix |
|---|---|
| Issue | `(bw)Is#<number>` |
| Pull Request | `(bw)PR#<number>` |

Example: `https://github.com/lux-sp4rk/digicard/issues/84` → Task description: `(bw)Is#84 - Restore Skills Tab`

## Tag Mapping

GitHub labels become Taskwarrior tags. A label `enhancement` on an issue becomes `+enhancement` in Taskwarrior.

Per-agent routing tags (e.g., `+ferris`, `+vixie`) must be added manually to tasks that need agent-specific heartbeat surfacing. Bugwarrior does not manage these.

## Elephantastic Integration

This doc is part of the elephantastic skill but is **opt-in**. Bugwarrior is not required for core elephantastic GTD — it's a GitHub sync layer on top of Taskwarrior.

See also:
- `references/taskwarrior-schema.md` — UDAs, reports, tag taxonomy
- `references/queue-heartbeat.md` — task queue change detection
- `references/quick-reference.md` — Taskwarrior command cheat sheet
