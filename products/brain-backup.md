# Automated Brain Backup

**Restore your AI's memory to any point in time.**

Your AI agent's brain lives in files — SOUL.md, MEMORY.md, daily logs, agent configs, scripts, research notes. If your machine dies, a bad edit corrupts something, or a sub-agent accidentally deletes shared state, everything is gone. There's no undo button.

This product sets up automated nightly backups of your entire workspace to a private GitHub repo. Every commit is a recoverable snapshot. If anything breaks, you restore to last night (or any previous night) with one command. We've been running this since March 2026, and it has already saved us from a corrupted memory file that would have lost 2 weeks of operational context.

---

## What You Get

- **daily-brain-backup.sh** — The backup script
- **macOS LaunchAgent** — Runs automatically at 11:55 PM every night
- **Linux cron configuration** — For VPS/server deployments
- **GitHub repo setup guide** — Private repo, remote configuration
- **.gitignore template** — Exclude secrets, node_modules, temp files
- **Restore procedure** — How to recover to any point in time
- **Verification script** — Confirm backups are working

---

## Step 1: Create a Private GitHub Repo

```bash
# Create a private repo for your backups
gh repo create your-brain-backup --private --description "AI workspace backup — nightly snapshots"

# Or do it on github.com → New Repository → Private
```

This repo stores ONLY your workspace files. No secrets (those are excluded by .gitignore). Keep it private — your MEMORY.md and agent configs contain operational details you don't want public.

---

## Step 2: Configure Your Workspace as a Git Repo

```bash
cd ~/.openclaw/workspace

# Initialize git if not already
git init

# Add your backup repo as a remote named "backup"
git remote add backup https://github.com/YOUR_USERNAME/your-brain-backup.git

# Verify
git remote -v
```

---

## Step 3: Create .gitignore

Save this as `~/.openclaw/workspace/.gitignore`:

```gitignore
# Secrets — NEVER commit these
.env
*.key
*.pem
*.secret

# Node modules
node_modules/
.node_modules/

# Temp files
tmp/
*.tmp
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Large binary files
*.pdf
*.zip
*.tar.gz
*.mp4
*.mov

# OpenClaw internals (managed by the gateway, not your workspace)
.openclaw-sessions/

# Build artifacts
dist/
build/
```

Customize as needed. The principle: commit everything that's YOUR content (configs, memory, scripts, research), exclude everything that's generated or secret.

---

## Step 4: Install the Backup Script

Save this as `~/.openclaw/workspace/scripts/daily-brain-backup.sh`:

```bash
#!/bin/bash
# daily-brain-backup.sh — Commit and push workspace to private GitHub repo
# Each commit = one recoverable snapshot

WORKSPACE="$HOME/.openclaw/workspace"
LOG="/tmp/openclaw/brain-backup.log"
mkdir -p /tmp/openclaw

log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG"
}

cd "$WORKSPACE" || { log "🚨 Cannot cd to workspace"; exit 1; }

# Stage all changes
git add -A 2>/dev/null

# Check if there are changes to commit
if git diff --cached --quiet 2>/dev/null; then
  log "No changes to commit — workspace unchanged"
  exit 0
fi

# Count changed files for the commit message
CHANGED=$(git diff --cached --name-only | wc -l | tr -d ' ')
TODAY=$(date '+%Y-%m-%d')
TIME=$(date '+%H:%M')

# Commit with date-stamped message
git commit -m "🧠 Daily backup ${TODAY} ${TIME} — ${CHANGED} files changed" \
  --author="YourAgent <your-email@example.com>" 2>&1 >> "$LOG"

# Push to remote
if git push backup main 2>&1 >> "$LOG"; then
  log "✅ Backup pushed: ${CHANGED} files (${TODAY} ${TIME})"
else
  log "🚨 Push failed — will retry next run"
fi
```

Make it executable:
```bash
chmod +x ~/.openclaw/workspace/scripts/daily-brain-backup.sh
```

**Customize:** Change the `--author` email to yours.

---

## Step 5: Set Up Auto-Start

### macOS (LaunchAgent)

Save this as `~/Library/LaunchAgents/com.openclaw.brain-backup.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.brain-backup</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/YOUR_USERNAME/.openclaw/workspace/scripts/daily-brain-backup.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>23</integer>
        <key>Minute</key>
        <integer>55</integer>
    </dict>
    <key>RunAtLoad</key>
    <false/>
    <key>StandardOutPath</key>
    <string>/tmp/openclaw/brain-backup-stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/openclaw/brain-backup-stderr.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string>
        <key>HOME</key>
        <string>/Users/YOUR_USERNAME</string>
    </dict>
</dict>
</plist>
```

Install:
```bash
# Replace YOUR_USERNAME
sed -i '' 's/YOUR_USERNAME/'"$(whoami)"'/g' ~/Library/LaunchAgents/com.openclaw.brain-backup.plist

# Load
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.openclaw.brain-backup.plist

# Verify
launchctl list | grep brain-backup
```

### Linux (cron)

```bash
# Run at 11:55 PM every night
crontab -e
# Add this line:
55 23 * * * /bin/bash $HOME/.openclaw/workspace/scripts/daily-brain-backup.sh
```

---

## Step 6: Do Your First Backup

```bash
# Run manually to verify everything works
bash ~/.openclaw/workspace/scripts/daily-brain-backup.sh

# Check the log
cat /tmp/openclaw/brain-backup.log

# Verify it pushed
gh repo view YOUR_USERNAME/your-brain-backup --web
```

You should see a commit with the 🧠 emoji and file count.

---

## How to Restore

### Restore entire workspace to a specific date

```bash
cd ~/.openclaw/workspace

# List all backup commits
git log --oneline backup/main

# Output looks like:
# abc1234 🧠 Daily backup 2026-03-18 23:55 — 12 files changed
# def5678 🧠 Daily backup 2026-03-17 23:55 — 8 files changed
# ghi9012 🧠 Daily backup 2026-03-16 23:55 — 23 files changed

# Restore to a specific date (creates a new branch from that point)
git checkout -b restore-mar-17 def5678

# Or hard reset to that commit (WARNING: overwrites current files)
git reset --hard def5678
```

### Restore a single file from a specific date

```bash
# Restore just MEMORY.md from 2 days ago
git checkout def5678 -- MEMORY.md

# Restore a daily log
git checkout def5678 -- memory/2026-03-17.md
```

### Browse what changed on a specific day

```bash
# See what files changed in a commit
git show --stat abc1234

# See the actual diff
git show abc1234
```

---

## Verification Script

Save this as `~/.openclaw/workspace/scripts/verify-backup.sh`:

```bash
#!/bin/bash
# Verify brain backup is running and recent

WORKSPACE="$HOME/.openclaw/workspace"
cd "$WORKSPACE" || exit 1

# Check last backup time
LAST_COMMIT=$(git log backup/main -1 --format="%ci" 2>/dev/null)
if [ -z "$LAST_COMMIT" ]; then
  echo "🚨 No backups found. Run daily-brain-backup.sh manually."
  exit 1
fi

# Parse timestamp
LAST_TS=$(date -j -f "%Y-%m-%d %H:%M:%S %z" "$LAST_COMMIT" "+%s" 2>/dev/null || date -d "$LAST_COMMIT" "+%s" 2>/dev/null)
NOW_TS=$(date "+%s")
AGE_HOURS=$(( (NOW_TS - LAST_TS) / 3600 ))

if [ "$AGE_HOURS" -gt 36 ]; then
  echo "🚨 Last backup was ${AGE_HOURS} hours ago: ${LAST_COMMIT}"
  exit 1
else
  COMMIT_COUNT=$(git rev-list --count backup/main 2>/dev/null)
  echo "✅ Backup healthy: ${AGE_HOURS}h ago (${COMMIT_COUNT} total snapshots)"
fi
```

Run it anytime: `bash scripts/verify-backup.sh`

Add it to your heartbeat checks so your agent monitors backup health automatically.

---

## Customization

### Change backup time
Edit the `StartCalendarInterval` in the plist (Hour/Minute) or the cron expression.

### Multiple backups per day
Change from `StartCalendarInterval` to `StartInterval`:
```xml
<key>StartInterval</key>
<integer>28800</integer>  <!-- Every 8 hours -->
```

### Backup multiple workspaces
If you run multiple OpenClaw agents with different workspaces, create one script per workspace or loop through them:

```bash
for ws in ~/.openclaw/workspace ~/other-agent/workspace; do
  cd "$ws" && git add -A && git diff --cached --quiet || \
    git commit -m "🧠 Backup $(date '+%Y-%m-%d %H:%M')" && git push backup main
done
```

### Size monitoring
Keep an eye on repo size:
```bash
du -sh ~/.openclaw/workspace/.git
# If it grows past 500MB, consider:
git gc --aggressive
```

---

## Troubleshooting

**Push fails with "authentication required":**
```bash
# Re-authenticate gh CLI
gh auth login
# Or set up SSH keys for the backup repo
```

**"Nothing to commit" every night:**
Your workspace isn't changing. This is normal on quiet days. The script skips the commit/push when there are no changes.

**LaunchAgent not firing:**
```bash
# Check if it's loaded
launchctl list | grep brain-backup
# If not, re-bootstrap
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.openclaw.brain-backup.plist
```

**Repo getting too large:**
Memory logs accumulate. Options:
1. Add old daily logs to `.gitignore` after archiving
2. Run `git gc --aggressive` monthly
3. Start a fresh repo annually and archive the old one

---

## Why This Matters

Your AI agent's value compounds over time. MEMORY.md accumulates decisions. Daily logs capture context. Scripts get refined. Configs get battle-tested. Losing this to a disk failure or bad edit erases months of compound knowledge.

A nightly backup to a private GitHub repo means:
- **Disaster recovery** — machine dies, restore to new hardware in minutes
- **Undo mistakes** — bad edit, corrupted file, rogue sub-agent → restore the file
- **Audit trail** — see exactly what changed when (git blame, git log)
- **Migration** — move to a new machine, new framework, or new agent by pulling the repo

$9 for permanent peace of mind. We run this every night at 11:55 PM and have never lost a byte.
