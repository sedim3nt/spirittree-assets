# Ops Monitor — AI Operations Agent Package

**Version:** 1.0 | **Level:** Intermediate | **Setup Time:** 30-45 minutes

Your AI ops agent that never sleeps. Monitors system health, auto-restarts failing services, runs heartbeat checks, verifies deploys. Built on the Grove ops agent architecture.

---

## What's In This Package

- `ROLE.md` for your ops agent
- Heartbeat configuration
- Watchdog config
- Deploy verification workflow
- LaunchAgent templates for all daemons
- Infrastructure check script docs

---

## ROLE.md — Ops Agent

Copy to `agents/ops/ROLE.md`:

```markdown
# ROLE.md — Ops Agent

**Role:** Ops / Monitoring Agent
**Model:** Claude Haiku (recommended for cost — runs frequently)
**Channel:** Cron-driven + sub-agent

## Responsibilities
- System health monitoring (gateway, n8n, dashboards, APIs)
- Service restart when needed
- Daily context logging
- Infrastructure checks
- Alert on anomalies

## Health Check Protocol
For each monitored service:
1. Ping the endpoint or check process
2. If unhealthy: attempt recovery (restart)
3. If recovery fails: send alert
4. Log all checks to memory/[DATE].md

## Service Recovery Order
1. Kill ALL related processes: `pkill -f <service>`
2. Wait 3 seconds
3. Verify port is clear: `lsof -ti:<port>`
4. Load fresh: `launchctl bootstrap gui/$(id -u) <plist>`
5. Test with actual operation (not just port check)

## Constraints
- Never touch client systems
- Log everything — if it matters, write it down
- Append-only to shared files
- Alert before destructive actions
- Prefer `trash` over permanent delete
```

---

## Heartbeat Configuration

### heartbeat-check.sh

Create `scripts/heartbeat-check.sh`:

```bash
#!/bin/bash
# Heartbeat check — verify critical services are running

set -e

TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
LOG_FILE="${HOME}/.openclaw/workspace/memory/heartbeat.log"
ALERT_NEEDED=false
ISSUES=()

log() {
    echo "[$TIMESTAMP] $1" | tee -a "$LOG_FILE"
}

check_port() {
    local service=$1
    local port=$2
    if curl -s --connect-timeout 3 "http://localhost:$port" > /dev/null 2>&1; then
        log "✅ $service (port $port) — OK"
        return 0
    else
        log "❌ $service (port $port) — FAILED"
        ISSUES+=("$service")
        ALERT_NEEDED=true
        return 1
    fi
}

check_process() {
    local service=$1
    local pattern=$2
    if pgrep -f "$pattern" > /dev/null 2>&1; then
        log "✅ $service — running"
        return 0
    else
        log "❌ $service — not found"
        ISSUES+=("$service")
        ALERT_NEEDED=true
        return 1
    fi
}

check_gateway() {
    local status=$(openclaw gateway status 2>/dev/null | grep -c "running" || echo "0")
    if [ "$status" -gt "0" ]; then
        log "✅ OpenClaw gateway — running"
        return 0
    else
        log "❌ OpenClaw gateway — down, attempting fix"
        openclaw doctor --fix 2>&1 | tee -a "$LOG_FILE"
        sleep 3
        # Re-check
        local status2=$(openclaw gateway status 2>/dev/null | grep -c "running" || echo "0")
        if [ "$status2" -gt "0" ]; then
            log "✅ OpenClaw gateway — recovered"
            return 0
        else
            log "🚨 OpenClaw gateway — UNRECOVERABLE"
            ISSUES+=("OpenClaw Gateway")
            ALERT_NEEDED=true
            return 1
        fi
    fi
}

# --- CONFIGURE YOUR SERVICES BELOW ---

check_gateway

# n8n (adjust port if different)
check_port "n8n" "5678"

# Dashboard (adjust for your setup)
# check_port "Dashboard" "3333"

# Custom services
# check_process "Custom Service" "service-pattern"

# --- ALERT IF NEEDED ---

if [ "$ALERT_NEEDED" = true ]; then
    ISSUE_LIST=$(IFS=', '; echo "${ISSUES[*]}")
    log "🚨 ALERT: Issues detected — $ISSUE_LIST"
    # Uncomment to send Telegram alert:
    # openclaw message "🚨 System alert: $ISSUE_LIST — check logs"
fi

log "Heartbeat check complete"
```

Make executable:
```bash
chmod +x ~/.openclaw/workspace/scripts/heartbeat-check.sh
```

### Heartbeat LaunchAgent

Save as `~/Library/LaunchAgents/com.openclaw.heartbeat.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.heartbeat</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/YOUR_USERNAME/.openclaw/workspace/scripts/heartbeat-check.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>300</integer>
    <key>StandardOutPath</key>
    <string>/tmp/heartbeat.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/heartbeat.err</string>
    <key>KeepAlive</key>
    <false/>
</dict>
</plist>
```

Load it:
```bash
launchctl load ~/Library/LaunchAgents/com.openclaw.heartbeat.plist
```

---

## Watchdog Configuration

### gateway-watchdog.sh

Create `scripts/gateway-watchdog.sh`:

```bash
#!/bin/bash
# Gateway watchdog — restart OpenClaw if it goes down

TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
LOG_FILE="/tmp/gateway-watchdog.log"

log() {
    echo "[$TIMESTAMP] $1" | tee -a "$LOG_FILE"
}

# Check if gateway is healthy
is_healthy() {
    openclaw gateway status 2>/dev/null | grep -q "running"
}

if is_healthy; then
    log "Gateway healthy"
    exit 0
fi

log "Gateway unhealthy — attempting recovery"

# Step 1: Run doctor
openclaw doctor --fix 2>&1 | tee -a "$LOG_FILE"
sleep 5

if is_healthy; then
    log "Gateway recovered via doctor --fix"
    # Optional: send Telegram alert
    # openclaw message "✅ Gateway self-healed at $TIMESTAMP"
    exit 0
fi

# Step 2: Force restart
log "Doctor failed — forcing restart"
pkill -f "openclaw" 2>/dev/null || true
sleep 3

# Restart gateway
openclaw gateway start 2>&1 | tee -a "$LOG_FILE"
sleep 5

if is_healthy; then
    log "Gateway recovered via force restart"
    # openclaw message "⚠️ Gateway restarted at $TIMESTAMP"
    exit 0
fi

log "🚨 Gateway UNRECOVERABLE — manual intervention required"
# This will alert via another channel if you have one configured
exit 1
```

### Watchdog LaunchAgent

Save as `~/Library/LaunchAgents/com.openclaw.watchdog.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.watchdog</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/YOUR_USERNAME/.openclaw/workspace/scripts/gateway-watchdog.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>300</integer>
    <key>StandardOutPath</key>
    <string>/tmp/watchdog.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/watchdog.err</string>
</dict>
</plist>
```

---

## Deploy Verification Workflow

### deploy-check.sh

Create `scripts/deploy-check.sh`:

```bash
#!/bin/bash
# Post-deploy verification

SERVICE=$1
PORT=$2
HEALTH_PATH=${3:-"/health"}

if [ -z "$SERVICE" ] || [ -z "$PORT" ]; then
    echo "Usage: deploy-check.sh <service-name> <port> [health-path]"
    exit 1
fi

echo "Verifying deploy: $SERVICE on port $PORT"

# Wait for service to come up
MAX_ATTEMPTS=12
ATTEMPT=0

while [ $ATTEMPT -lt $MAX_ATTEMPTS ]; do
    if curl -s --connect-timeout 5 "http://localhost:$PORT$HEALTH_PATH" > /dev/null 2>&1; then
        echo "✅ $SERVICE is responding on port $PORT"
        exit 0
    fi
    ATTEMPT=$((ATTEMPT+1))
    echo "Waiting... ($ATTEMPT/$MAX_ATTEMPTS)"
    sleep 5
done

echo "❌ $SERVICE failed to come up after $((MAX_ATTEMPTS * 5)) seconds"
exit 1
```

### Deploy Verification Agent Prompt

After any service deployment:

```
Verify deploy for [SERVICE].

Steps:
1. Check if [SERVICE] is running: `pgrep -f [process-pattern]`
2. Verify port [PORT] is responding: `curl -s http://localhost:[PORT]/health`
3. Run a smoke test: [describe a test operation]
4. Check recent logs for errors: `tail -50 /tmp/[service].log`

If any step fails:
1. Show me the error output
2. Suggest the fix
3. Wait for my approval before making changes

Report: pass/fail for each step with details on any failures.
```

---

## n8n Health Check

### Check last 24h executions

```bash
# Get recent workflow executions
curl -s "http://localhost:5678/api/v1/executions?limit=50" \
  -H "X-N8N-API-KEY: $N8N_API_KEY" | \
  jq '.data[] | {workflowName: .workflowData.name, status: .status, startedAt: .startedAt}'
```

### n8n Health Check Agent Prompt

```
Check n8n health.

1. Fetch last 50 executions from the n8n API
2. Count: total / success / failed / waiting
3. If failure rate > 20%: list the failing workflows and error messages
4. Check if any workflows haven't run in 24h when they should have
5. Report status as: HEALTHY / DEGRADED / CRITICAL

If DEGRADED or CRITICAL: list specific workflows to investigate.
```

---

## LaunchAgent Templates

### Generic Service Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.YOUR_DOMAIN.SERVICE_NAME</string>
    <key>ProgramArguments</key>
    <array>
        <string>/path/to/executable</string>
        <string>arg1</string>
        <string>arg2</string>
    </array>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin</string>
        <key>YOUR_ENV_VAR</key>
        <string>value</string>
    </dict>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/SERVICE_NAME.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/SERVICE_NAME.err</string>
</dict>
</plist>
```

### Service Restart Protocol

When a service needs a full restart (follow this order exactly):

```bash
# Step 1: Kill all related processes
pkill -f "service-name"
sleep 3

# Step 2: Verify port is clear
lsof -ti:PORT | xargs kill -9 2>/dev/null || true
sleep 1

# Step 3: Load fresh
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.domain.service.plist

# Step 4: Verify new PIDs
ps aux | grep service-name | grep -v grep

# Step 5: Functional test
curl -s http://localhost:PORT/health
```

---

## Troubleshooting

**Heartbeat script not running**
- Check plist is loaded: `launchctl list | grep openclaw`
- Check logs: `cat /tmp/heartbeat.err`
- Verify script path in plist matches actual location
- Try running manually first: `bash scripts/heartbeat-check.sh`

**Watchdog restarts but gateway stays down**
- Check openclaw config: `openclaw config show`
- Look at full gateway logs: `openclaw gateway logs`
- Run `openclaw doctor --fix` manually and observe output
- Check if ports are in use: `lsof -i :PORT`

**LaunchAgent not loading**
- Validate plist XML: `plutil -lint ~/Library/LaunchAgents/com.openclaw.heartbeat.plist`
- Check permissions: `ls -la ~/Library/LaunchAgents/`
- Reload: `launchctl unload ... && launchctl load ...`

**n8n health check fails to authenticate**
- Generate an API key in n8n: Settings → API Keys → Create
- Store it: `openclaw secret set N8N_API_KEY "your-key"`
- Test: `curl -H "X-N8N-API-KEY: $N8N_API_KEY" http://localhost:5678/api/v1/workflows`

---

*Built on OpenClaw. Requires OpenClaw installed and configured on macOS.*
