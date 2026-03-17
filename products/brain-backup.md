#!/bin/bash
# daily-brain-backup.sh — Commit and push workspace to GitHub daily
# Each commit = one recoverable snapshot. Restore any day via git checkout.

WORKSPACE="/Users/spirittree/.openclaw/workspace"
LOG="/tmp/openclaw/brain-backup.log"
mkdir -p /tmp/openclaw

log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG"
}

cd "$WORKSPACE" || { log "🚨 Cannot cd to workspace"; exit 1; }

# Stage all changes
git add -A 2>/dev/null

# Check if there are changes
if git diff --cached --quiet 2>/dev/null; then
  log "No changes to commit"
  exit 0
fi

# Count changed files
CHANGED=$(git diff --cached --name-only | wc -l | tr -d ' ')
TODAY=$(date '+%Y-%m-%d')
TIME=$(date '+%H:%M')

# Commit with date-stamped message
git commit -m "🧠 Daily backup ${TODAY} ${TIME} — ${CHANGED} files changed" \
  --author="Sedim3nt <terraloam.eye@gmail.com>" 2>&1 >> "$LOG"

# Push to remote
if git push backup main 2>&1 >> "$LOG"; then
  log "✅ Backup pushed: ${CHANGED} files (${TODAY} ${TIME})"
else
  log "🚨 Push failed — will retry next run"
fi
