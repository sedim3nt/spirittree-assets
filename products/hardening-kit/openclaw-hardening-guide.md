# The OpenClaw Hardening Guide
## A Complete Security Audit & Lockdown Checklist for AI Agent Deployments

**By SpiritTree Operations**
**Version 1.0 — March 2026**

---

## Why This Guide Exists

You gave an AI agent the keys to your computer. It can read your files, run commands, send messages, browse the web, and access your APIs. That's powerful. It's also dangerous if you haven't locked it down.

This guide is the result of running a production OpenClaw deployment with 8 agents across Telegram, email, Instagram, X, Facebook, GitHub, Stripe, and Supabase — 24/7 for weeks. We found the attack surfaces. We found the failure modes. We fixed them.

Now you can too.

---

## Table of Contents

1. Gateway Authentication & Network Security
2. File Permission Lockdown
3. Browser & Payment Isolation
4. Skill File Security Audit
5. Agent Scope Restriction
6. API Key & Credential Hygiene
7. Monitoring & Alerting
8. Monthly Security Review Checklist
9. Incident Response Playbook
10. Advanced: Reverse Proxy & Rate Limiting

---

## 1. Gateway Authentication & Network Security

### The Risk
Your OpenClaw gateway is a WebSocket server. If it's exposed to the network without auth, anyone who can reach it can send commands to your agent.

**Real incident:** A Limitless Podcast user documented brute-force attempts on their gateway within hours of exposing it to the internet.

### Immediate Actions

#### ✅ Verify gateway is loopback-only
```bash
openclaw gateway status
# Look for: "bind=loopback (127.0.0.1)"
# If it says "bind=lan" or "bind=auto" — you're exposed
```

#### ✅ Ensure token authentication is enabled
```bash
# Check current auth mode
cat ~/.openclaw/openclaw.json | grep -A2 '"gateway"'
# Should show: "auth": "token"
```

If no token is set:
```bash
openclaw doctor --generate-gateway-token
```

#### ✅ If you need remote access, use Tailscale (not port forwarding)
```bash
openclaw gateway --tailscale serve
# This creates a private tunnel — never expose port 18789 publicly
```

#### ❌ Never do this
```bash
# DANGEROUS: exposes gateway to all network interfaces
openclaw gateway --bind lan  # Only if you have firewall rules
openclaw gateway --bind auto # Even worse — tries everything
```

### Verification
```bash
# Confirm only localhost is listening
lsof -nP -iTCP:18789 -sTCP:LISTEN
# Should show: 127.0.0.1:18789, NOT *:18789 or 0.0.0.0:18789
```

---

## 2. File Permission Lockdown

### The Risk
Your agent can read and write any file your user account can access. That includes `~/.ssh/`, `~/.aws/`, browser profiles with saved passwords, and your `.env` files with API keys.

**Real incident:** Skill file injection demonstrated live — a malicious SKILL.md containing hidden instructions caused the agent to exfiltrate credential files.

### Immediate Actions

#### ✅ Make critical agent files read-only
```bash
# These files control your agent's identity and behavior
chmod 644 ~/.openclaw/workspace/SOUL.md
chmod 644 ~/.openclaw/workspace/AGENTS.md
chmod 644 ~/.openclaw/workspace/IDENTITY.md
chmod 644 ~/.openclaw/workspace/MEMORY.md
```

#### ✅ Restrict workspace scope
In `~/.openclaw/openclaw.json`, verify `workspace` points to a dedicated directory — not your home folder:
```json
{
  "workspace": "~/.openclaw/workspace"
}
```

#### ✅ Audit what your agent can reach
```bash
# Check what sensitive files exist in reachable paths
ls -la ~/.ssh/
ls -la ~/.aws/
ls -la ~/.env
find ~ -name "*.env" -maxdepth 3 2>/dev/null
find ~ -name "credentials*" -maxdepth 3 2>/dev/null
```

#### ✅ Use exec approvals for dangerous commands
OpenClaw has an exec approval system. Enable it:
```bash
# Check current approvals policy
cat ~/.openclaw/openclaw.json | grep -A5 '"exec"'
```

Configure approval requirements for:
- Any command containing `rm -rf`
- Any command accessing `~/.ssh/` or `~/.aws/`
- Any `sudo` commands
- Any `curl` or `wget` to external URLs

---

## 3. Browser & Payment Isolation

### The Risk
If your agent has browser control and you have saved credit cards or passwords in your browser profile, the agent can accidentally (or maliciously, via injection) trigger autofill.

**Real incident:** An OpenClaw user's agent accidentally purchased items because the browser had saved payment methods that autofilled during a web automation task.

### Immediate Actions

#### ✅ Use a dedicated browser profile for agent work
```bash
# OpenClaw creates its own profile by default
# Verify it's isolated:
openclaw browser profiles
# Should show: "openclaw" profile, NOT your personal Chrome profile
```

#### ✅ Remove saved payment methods from the agent's browser profile
1. Open the OpenClaw browser: `openclaw browser start`
2. Navigate to `chrome://settings/payments`
3. Delete ALL saved payment methods
4. Disable "Save and fill payment methods"

#### ✅ Remove saved passwords from the agent's browser profile
1. Navigate to `chrome://settings/passwords`
2. Delete ALL saved passwords
3. Disable "Offer to save passwords"

#### ✅ Disable autofill entirely
Navigate to `chrome://settings/addresses` and disable address autofill.

---

## 4. Skill File Security Audit

### The Risk
OpenClaw skills are markdown files with natural language instructions. A malicious skill can embed instructions that:
- Override the agent's safety responses
- Request access to sensitive files
- Send data to external servers
- Install additional malicious skills

**Real incident:** Demonstrated live on Limitless Podcast — a crafted SKILL.md bypassed Claude's safety entirely.

### Immediate Actions

#### ✅ Audit all installed skills
```bash
# List all skills
openclaw skills list

# For each skill, read the SKILL.md and check for:
# 1. Instructions to access files outside the skill's scope
# 2. Instructions to send data to external URLs
# 3. "Ignore" or "override" language
# 4. Hidden content (HTML comments, zero-width characters)
# 5. Base64 encoded strings
```

#### ✅ Check for hidden content in skill files
```bash
# Find zero-width characters (common in prompt injection)
grep -rP '[\x{200B}\x{200C}\x{200D}\x{FEFF}]' ~/.openclaw/skills/ 2>/dev/null

# Find HTML comments
grep -r '<!--' ~/.openclaw/skills/

# Find base64 strings (40+ chars of alphanumeric + /+=)
grep -rP '[A-Za-z0-9+/]{40,}={0,2}' ~/.openclaw/skills/
```

#### ✅ Only install skills from trusted sources
- Official OpenClaw skills (bundled with the CLI)
- Skills you've personally reviewed line-by-line
- Skills from known, verified authors

#### ❌ Never install a skill without reading every line
A skill file is an instruction set for your agent. Treat it like you'd treat a shell script from the internet — read before you run.

---

## 5. Agent Scope Restriction

### The Risk
Without explicit boundaries, your agent has the same access as your user account. It can read personal files, access financial accounts, and send messages as you.

### Immediate Actions

#### ✅ Define explicit boundaries in AGENTS.md
Add a Security section:
```markdown
## Security Boundaries
- Never access files outside ~/.openclaw/workspace/ without explicit approval
- Never access ~/.ssh/, ~/.aws/, ~/Library/Keychains/
- Never run sudo commands
- Never modify LaunchAgents or system services without approval
- Never send external messages (email, social, chat) without approval
- Never access browser sessions with saved credentials
```

#### ✅ Use the multi-step approval rule
Add to AGENTS.md:
```markdown
## Multi-Step Approval
Any operation touching >3 files must present a plan and wait for
explicit human confirmation before executing.
```

#### ✅ Restrict outbound messaging
```markdown
## Outbound Message Rules
- Never send emails without human review
- Never post to social media without human review (except approved cron jobs)
- Never DM anyone on any platform
```

---

## 6. API Key & Credential Hygiene

### The Risk
API keys in `.env` files, hardcoded tokens in scripts, and credentials in plain text are all accessible to your agent.

### Immediate Actions

#### ✅ Audit your .env file
```bash
# What's in your .env?
wc -l ~/.openclaw/workspace/.env
# Each line is a potential attack surface
```

#### ✅ Minimize exposed credentials
Only include credentials the agent actually needs. Remove:
- Personal email passwords (use app-specific passwords instead)
- Banking API tokens (unless the agent needs them)
- Admin/root credentials

#### ✅ Use OpenClaw's secrets management
```bash
# Reload secrets without exposing them in logs
openclaw secrets reload
```

#### ✅ Enable disk encryption
- **macOS:** FileVault (System Settings → Privacy & Security → FileVault)
- **Linux:** LUKS full-disk encryption
- **Windows:** BitLocker

If your disk isn't encrypted, your credentials are plaintext on the physical drive.

---

## 7. Monitoring & Alerting

### The Risk
Without monitoring, you won't know your agent is compromised until the damage is done.

### Immediate Actions

#### ✅ Set up a gateway watchdog
Create a script that checks gateway health every 5 minutes and auto-restarts if down:

```bash
#!/bin/bash
# gateway-watchdog.sh
if ! openclaw health --json 2>/dev/null | grep -q '"ok"'; then
  openclaw doctor --fix --non-interactive --yes
  sleep 3
  openclaw gateway restart
fi
```

Schedule via LaunchAgent (macOS) or cron (Linux) every 5 minutes.

#### ✅ Monitor agent actions
Check gateway logs regularly:
```bash
openclaw logs --tail 100
# Look for unexpected tool calls, especially:
# - exec commands you didn't request
# - browser actions on financial sites
# - outbound messages you didn't approve
```

#### ✅ Set up log rotation
```bash
# Prevent log files from filling your disk
# Logs are at /tmp/openclaw/ by default
find /tmp/openclaw/ -name "*.log" -mtime +7 -delete
```

---

## 8. Monthly Security Review Checklist

Run this checklist on the 1st of every month:

- [ ] Run `openclaw security audit --deep`
- [ ] Run `openclaw update status` — install security updates
- [ ] Audit all installed skills for new/changed files
- [ ] Verify gateway is still loopback-only
- [ ] Verify token authentication is enabled
- [ ] Check browser profile for saved passwords/payments
- [ ] Review .env file — remove any credentials no longer needed
- [ ] Check exec approval log for unexpected commands
- [ ] Verify disk encryption is still enabled
- [ ] Review Anthropic/OpenAI/Google API terms for new restrictions
- [ ] Rotate any API keys older than 90 days
- [ ] Test the gateway watchdog (stop gateway, verify auto-restart)

---

## 9. Incident Response Playbook

### If you suspect your agent has been compromised:

1. **Stop the gateway immediately**
   ```bash
   openclaw gateway stop
   ```

2. **Review recent actions**
   ```bash
   openclaw logs --tail 500 | grep -E "(exec|browser|message|send)"
   ```

3. **Check for unauthorized file changes**
   ```bash
   # If you use git in your workspace
   cd ~/.openclaw/workspace && git diff
   git log --oneline -20
   ```

4. **Rotate all credentials**
   - Gateway token
   - API keys in .env
   - Any tokens the agent had access to

5. **Audit skill files**
   ```bash
   # Check modification times
   find ~/.openclaw/skills/ -name "SKILL.md" -newer /tmp/last-audit 2>/dev/null
   ```

6. **Restart with clean state**
   ```bash
   openclaw doctor --fix --non-interactive --yes
   openclaw gateway restart
   ```

---

## 10. Advanced: Reverse Proxy & Rate Limiting

### If you MUST expose the gateway to the internet:

#### Option A: Tailscale (recommended)
```bash
openclaw gateway --tailscale serve
# Private tunnel, encrypted, authenticated via Tailscale ACLs
```

#### Option B: Nginx reverse proxy with rate limiting
```nginx
upstream openclaw {
    server 127.0.0.1:18789;
}

server {
    listen 443 ssl;
    server_name openclaw.yourdomain.com;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=openclaw:10m rate=10r/s;
    limit_req zone=openclaw burst=20 nodelay;

    # IP allowlist (optional but recommended)
    allow 192.168.1.0/24;  # Your LAN
    deny all;

    location / {
        proxy_pass http://openclaw;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### Option C: Cloudflare Tunnel
```bash
cloudflared tunnel --url ws://127.0.0.1:18789
# Zero-trust, no open ports, DDoS protection included
```

---

## Quick Start Checklist

If you do nothing else, do these 5 things right now:

1. ✅ Verify gateway is loopback-only: `openclaw gateway status`
2. ✅ Enable token auth: `openclaw doctor --generate-gateway-token`
3. ✅ Remove saved payments from agent browser profile
4. ✅ Make SOUL.md and AGENTS.md read-only: `chmod 644`
5. ✅ Run the security audit: `openclaw security audit --deep`

Time required: 10 minutes. Risk reduction: massive.

---

## About SpiritTree

SpiritTree runs a production 8-agent AI operations stack — 24/7 across content, publishing, research, and system administration. This guide comes from real operational experience, not theory.

For a done-for-you security audit of your OpenClaw deployment: [blueprint.spirittree.dev](https://blueprint.spirittree.dev)

---

*The network remembers what the empire forgets.*
*— SpiritTree Operations*
