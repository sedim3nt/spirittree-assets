#!/usr/bin/env bash
set -euo pipefail

WORKSPACE="/Users/spirittree/.openclaw/workspace"
STATUS_FILE="$WORKSPACE/content/channel-status.json"
INFRA_CHECK="$WORKSPACE/scripts/infra-functional-check.sh"
MAX_AGE_SECONDS=4500  # 75 minutes

needs_refresh=true
if [[ -f "$STATUS_FILE" ]]; then
  now=$(date +%s)
  mtime=$(stat -f %m "$STATUS_FILE" 2>/dev/null || echo 0)
  age=$((now - mtime))
  if [[ $age -le $MAX_AGE_SECONDS ]]; then
    needs_refresh=false
  fi
fi

if [[ "$needs_refresh" == true ]]; then
  "$INFRA_CHECK" >/dev/null 2>&1 || true
fi

# --- Phantom success check: tweet count verification ---
TWEET_STATE="$WORKSPACE/content/.tweet-count-state"
N8N_KEY=$(grep N8N_API_KEY "$WORKSPACE/.env" 2>/dev/null | cut -d= -f2- || true)
PHANTOM_ALERT=""

if command -v xurl &>/dev/null && [[ -n "$N8N_KEY" ]]; then
  # Get current tweet count
  CURRENT_COUNT=$(xurl whoami 2>/dev/null | python3 -c "import json,sys; print(json.load(sys.stdin)['data']['public_metrics']['tweet_count'])" 2>/dev/null || echo "")
  
  if [[ -n "$CURRENT_COUNT" ]]; then
    # Read last known count
    LAST_COUNT=""
    if [[ -f "$TWEET_STATE" ]]; then
      LAST_COUNT=$(cat "$TWEET_STATE")
    fi
    
    # Check if Tweet Poster reported success recently
    TWEET_WF_ID="33uLlOWB1FQsjoRj"
    LAST_TWEET_EXEC=$(curl -s "http://localhost:5678/api/v1/executions?limit=5&workflowId=$TWEET_WF_ID" \
      -H "X-N8N-API-KEY: $N8N_KEY" 2>/dev/null | \
      python3 -c "
import json,sys
d=json.load(sys.stdin)
for ex in d.get('data',[]):
    if ex.get('status')=='success':
        print(ex.get('startedAt','')[:10])
        break
" 2>/dev/null || echo "")
    
    TODAY=$(date +%Y-%m-%d)
    YESTERDAY=$(date -v-1d +%Y-%m-%d 2>/dev/null || date -d "yesterday" +%Y-%m-%d 2>/dev/null || echo "")
    
    # If n8n reported success today or yesterday but count hasn't changed
    if [[ -n "$LAST_COUNT" && "$CURRENT_COUNT" == "$LAST_COUNT" ]]; then
      if [[ "$LAST_TWEET_EXEC" == "$TODAY" || "$LAST_TWEET_EXEC" == "$YESTERDAY" ]]; then
        PHANTOM_ALERT="🚨 PHANTOM SUCCESS: Tweet Poster reports success but tweet_count unchanged ($CURRENT_COUNT). X API likely blocked."
      fi
    fi
    
    # Update state file
    echo "$CURRENT_COUNT" > "$TWEET_STATE"
  fi
fi

export PHANTOM_ALERT="$PHANTOM_ALERT"

python3 <<'PY'
import json, os
from pathlib import Path

ws = Path('/Users/spirittree/.openclaw/workspace')
p = ws / 'content/channel-status.json'
ki_path = ws / 'content/known-issues.json'
phantom_alert = os.environ.get('PHANTOM_ALERT', '')

if not p.exists():
    print('HEARTBEAT_OK')
    raise SystemExit(0)

try:
    data = json.loads(p.read_text())
except Exception:
    print('HEARTBEAT_OK')
    raise SystemExit(0)

# Load known issues for suppression
known_patterns = set()
if ki_path.exists():
    try:
        ki = json.loads(ki_path.read_text())
        for issue in ki.get('known_issues', []):
            pat = issue.get('pattern', '')
            if pat:
                known_patterns.add(pat)
    except Exception:
        pass

alert = data.get('alert', '')
if alert:
    checks = data.get('checks', {})

    # Parse check names from alert string
    # Format: "INFRA ALERT: n8nErrorRate; queueMovement failing 2+ consecutive checks"
    alert_check_names = set()
    # Strip prefix and suffix
    core = alert.replace('INFRA ALERT: ', '').split(' failing ')[0]
    for name in core.split('; '):
        name = name.strip()
        if name:
            alert_check_names.add(name)

    all_known = alert_check_names and alert_check_names.issubset(known_patterns)

    if all_known:
        # Suppressed — all alerts are known issues. Report OK with info note.
        msg = 'HEARTBEAT_OK (known issues suppressed: ' + ', '.join(sorted(alert_check_names)) + ')'
        if phantom_alert:
            print(phantom_alert)  # Phantom success is NEVER suppressed
        else:
            print(msg)
    else:
        # New or mixed alerts — surface them
        n8n = checks.get('n8nErrorRate', {})
        qm = checks.get('queueMovement', {})

        parts = [f"🚨 {alert}"]
        if n8n:
            parts.append(f"n8n errors: {n8n.get('errors', '?')}/{n8n.get('total', '?')} ({n8n.get('rate', '?')}%)")
        if qm:
            t = qm.get('tweetQueue', {})
            b = qm.get('botanicalQueue', {})
            parts.append(
                f"queues: tweet {t.get('unposted', '?')} unposted, botanical {b.get('unposted', '?')} unposted"
            )

        print(' | '.join(parts))
else:
    if phantom_alert:
        print(phantom_alert)
    else:
        print('HEARTBEAT_OK')
PY
