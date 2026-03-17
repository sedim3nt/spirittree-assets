#!/bin/bash
# Gateway Watchdog — detects gateway down, runs doctor --fix, restarts
# Designed to run via LaunchAgent every 5 minutes

LOG="/tmp/openclaw/gateway-watchdog.log"
mkdir -p /tmp/openclaw

log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG"
}

# Check if gateway is responding
if openclaw health --json 2>/dev/null | grep -q '"ok"'; then
  # Gateway healthy — nothing to do
  exit 0
fi

log "⚠️ Gateway not responding. Running doctor + restart..."

# Run doctor with non-interactive fix
openclaw doctor --fix --non-interactive --yes >> "$LOG" 2>&1

# Give it a moment
sleep 3

# Restart the gateway service
openclaw gateway restart >> "$LOG" 2>&1

# Wait for startup
sleep 5

# Verify recovery
if openclaw health --json 2>/dev/null | grep -q '"ok"'; then
  log "✅ Gateway recovered successfully (pid $(pgrep -f 'openclaw.*gateway' | head -1))"
else
  log "🚨 Gateway STILL DOWN after doctor + restart. Manual intervention needed."
  # Could add Telegram alert here in the future
fi
