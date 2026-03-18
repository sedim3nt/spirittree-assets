# Gateway Watchdog System

**Never wake up to a dead AI agent again.**

Your OpenClaw gateway crashes at 2 AM. Nobody notices until you open Telegram at 8 AM and realize your agent has been silent for 6 hours. Your morning briefing didn't run. Your heartbeat alerts didn't fire. Your crons are all missed. You lost half a day.

This product eliminates that failure mode permanently. A lightweight watchdog checks your gateway health every 5 minutes. If it's down, it runs `openclaw doctor --fix`, restarts the gateway, verifies recovery, and logs everything. If it can't recover, it alerts you immediately instead of failing silently.

We built this after losing 13 hours of uptime because the gateway died overnight and our heartbeat system (which depends on the gateway) couldn't alert us that the gateway was dead. The watchdog runs outside the gateway — it's the independent safety net that monitors the monitor.

---

## What You Get

- **gateway-watchdog.sh** — The watchdog script (copy-paste ready)
- **LaunchAgent plist** — macOS auto-start configuration
- **Linux systemd unit** — For VPS/server deployments
- **Setup guide** — 5-minute installation
- **Customization guide** — Alert integrations, check intervals, recovery strategies

---

## gateway-watchdog.sh

Copy this to `~/.openclaw/workspace/scripts/gateway-watchdog.sh` and run `chmod +x` on it.

```bash
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
  # Optional: send alert via curl to Telegram, Slack, Discord, etc.
  # curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  #   -d "chat_id=$CHAT_ID&text=🚨 Gateway STILL DOWN after watchdog restart. Check manually."
fi
```

---

## macOS LaunchAgent

Copy this to `~/Library/LaunchAgents/com.openclaw.gateway-watchdog.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.gateway-watchdog</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/YOUR_USERNAME/.openclaw/workspace/scripts/gateway-watchdog.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>300</integer>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/openclaw/gateway-watchdog-stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/openclaw/gateway-watchdog-stderr.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string>
    </dict>
</dict>
</plist>
```

**Install:**
```bash
# Replace YOUR_USERNAME with your actual username
sed -i '' 's/YOUR_USERNAME/'"$(whoami)"'/' ~/Library/LaunchAgents/com.openclaw.gateway-watchdog.plist

# Load the agent
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.openclaw.gateway-watchdog.plist

# Verify it's running
launchctl list | grep gateway-watchdog
```

---

## Linux systemd Unit

For VPS or server deployments. Save to `/etc/systemd/system/openclaw-watchdog.service`:

```ini
[Unit]
Description=OpenClaw Gateway Watchdog
After=network.target

[Service]
Type=oneshot
ExecStart=/home/YOUR_USER/.openclaw/workspace/scripts/gateway-watchdog.sh
User=YOUR_USER
Environment=PATH=/usr/local/bin:/usr/bin:/bin

[Install]
WantedBy=multi-user.target
```

And a timer at `/etc/systemd/system/openclaw-watchdog.timer`:

```ini
[Unit]
Description=Run OpenClaw Gateway Watchdog every 5 minutes

[Timer]
OnBootSec=60
OnUnitActiveSec=300
AccuracySec=30

[Install]
WantedBy=timers.target
```

**Install:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-watchdog.timer
systemctl status openclaw-watchdog.timer
```

---

## Customization

### Change check interval
- **macOS:** Edit `StartInterval` in the plist (300 = 5 minutes, 60 = 1 minute)
- **Linux:** Edit `OnUnitActiveSec` in the timer (300 = 5 minutes)

### Add Telegram alerts
Uncomment the curl lines in the script and set your bot token and chat ID:

```bash
BOT_TOKEN="your-telegram-bot-token"
CHAT_ID="your-chat-id"

# In the "STILL DOWN" block:
curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
  -d "chat_id=$CHAT_ID&text=🚨 OpenClaw gateway STILL DOWN after watchdog restart. Check manually."
```

### Add Slack alerts
```bash
SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
curl -s -X POST "$SLACK_WEBHOOK" \
  -H 'Content-Type: application/json' \
  -d '{"text":"🚨 OpenClaw gateway STILL DOWN after watchdog restart."}'
```

### Add Discord alerts
```bash
DISCORD_WEBHOOK="https://discord.com/api/webhooks/YOUR/WEBHOOK"
curl -s -X POST "$DISCORD_WEBHOOK" \
  -H 'Content-Type: application/json' \
  -d '{"content":"🚨 OpenClaw gateway STILL DOWN after watchdog restart."}'
```

### Pre-flight port clearing
If your gateway fails because a stale process holds the port, add this before the restart:

```bash
# Clear stale processes on the gateway port (default 18789)
lsof -ti :18789 | xargs kill -9 2>/dev/null
sleep 2
```

### Cooldown between restarts
Prevent restart loops by adding a cooldown check:

```bash
COOLDOWN_FILE="/tmp/openclaw/watchdog-last-restart"
if [ -f "$COOLDOWN_FILE" ]; then
  LAST=$(cat "$COOLDOWN_FILE")
  NOW=$(date +%s)
  DIFF=$((NOW - LAST))
  if [ "$DIFF" -lt 300 ]; then
    log "⏳ Cooldown active (${DIFF}s since last restart). Skipping."
    exit 0
  fi
fi
date +%s > "$COOLDOWN_FILE"
```

---

## Log Management

Logs write to `/tmp/openclaw/gateway-watchdog.log`. View them:

```bash
# Tail live
tail -f /tmp/openclaw/gateway-watchdog.log

# Last 20 entries
tail -20 /tmp/openclaw/gateway-watchdog.log

# Count restarts this week
grep "recovered\|STILL DOWN" /tmp/openclaw/gateway-watchdog.log | wc -l
```

Logs rotate automatically when `/tmp` is cleared (macOS clears on reboot). For persistent logs, change the `LOG` path to `~/.openclaw/logs/gateway-watchdog.log`.

---

## Troubleshooting

**Watchdog isn't running:**
```bash
# macOS
launchctl list | grep watchdog
# If not listed, re-bootstrap:
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.openclaw.gateway-watchdog.plist
```

**Gateway restarts but dies immediately:**
Check if another process holds the port:
```bash
lsof -i :18789
```
Add the pre-flight port clearing (see Customization above).

**Doctor --fix hangs:**
Add a timeout wrapper:
```bash
timeout 30 openclaw doctor --fix --non-interactive --yes >> "$LOG" 2>&1
```

**Too many restarts:**
Add the cooldown check (see Customization above) to prevent restart loops.

---

## How We Use It

This watchdog has been running in our production system since March 2026. It checks every 5 minutes via a macOS LaunchAgent. In the first week, it caught and auto-recovered 3 gateway crashes that would have gone unnoticed for hours. Combined with our heartbeat system (which checks service health every 6 hours), it provides two layers of monitoring — the watchdog catches gateway crashes within 5 minutes, and the heartbeat catches everything else every 6 hours.

The key insight: **your heartbeat system depends on the gateway to function**. If the gateway dies, the heartbeat can't fire. The watchdog is the independent safety net that runs outside the gateway and monitors the thing that monitors everything else.
