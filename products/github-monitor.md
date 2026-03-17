# GitHub Monitor — Repository Intelligence System

**Version:** 1.0 | **Level:** Beginner | **Setup Time:** 20-30 minutes

Track open issues, PR status, CI runs, and contributor activity. Daily digest of what changed. Automated issue triage suggestions. Built on the gh CLI.

---

## What's In This Package

- gh CLI setup
- Daily repo digest cron
- Open PR summary
- CI run monitoring
- Issue triage framework
- Contributor activity tracking
- Release notes generator
- Stale issue detector

---

## gh CLI Setup

### Install

```bash
# macOS
brew install gh

# Linux
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install gh
```

### Authenticate

```bash
gh auth login
# Choose: GitHub.com → HTTPS → Yes to authenticate → Login with browser
```

### Test

```bash
gh auth status
gh repo list --limit 5
```

---

## Repository Configuration

Create `github/repos.md`:

```markdown
# Monitored Repositories

## Daily Monitor
- owner/repo-name — [description]
- owner/another-repo — [description]

## Weekly Monitor
- owner/less-active-repo — [description]

## My Repos (I'm the owner)
- myusername/my-project

## Contribution Tracking
Repos where I'm an active contributor:
- owner/open-source-project
```

---

## Daily Repo Digest

### Digest Prompt (runs at 8 AM)

```
Daily GitHub digest. Check all repos in github/repos.md.

For each DAILY MONITOR repo:

1. Open PRs needing review:
   - `gh pr list --repo [owner/repo] --state open --json number,title,createdAt,author`
   - Flag any older than 7 days

2. New issues (last 24h):
   - `gh issue list --repo [owner/repo] --state open --created "$(date -v-1d '+%Y-%m-%d')"` 
   - Categorize: bug / feature / question / other

3. CI status (latest run):
   - `gh run list --repo [owner/repo] --limit 5 --json status,conclusion,name,createdAt`
   - Flag any failures

4. Merged PRs (last 24h):
   - What shipped yesterday?

DIGEST FORMAT:
---
## GitHub Digest — [DATE]

### [REPO NAME]
PRs: [X] open ([X] waiting >7 days)
Issues: [X] new today — [bug/feature breakdown]
CI: ✅ passing / ❌ [N] failing / ⚠️ [status]
Shipped: [merged PR titles]
---

Write to github/digests/[DATE].md
Send Telegram summary
```

### LaunchAgent

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>com.openclaw.github-digest</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/openclaw</string>
        <string>cron</string>
        <string>--message</string>
        <string>Daily GitHub digest for all repos in github/repos.md. PRs, issues, CI status, recent merges.</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict><key>Hour</key><integer>8</integer><key>Minute</key><integer>0</integer></dict>
    <key>StandardOutPath</key><string>/tmp/github-digest.log</string>
</dict>
</plist>
```

---

## PR Summary and Review

### PR Overview Prompt

```
Summarize open PRs for [owner/repo].

Run: `gh pr list --repo [owner/repo] --state open --json number,title,body,createdAt,author,reviewDecision,additions,deletions`

For each PR:
1. **Title and number**
2. **Age** — how many days open
3. **Size** — lines added/deleted
4. **Status** — needs review / approved / changes requested
5. **Quick summary** — what does this PR do? (from title + body)

Sort by: oldest first (most urgent to review)

Flag:
- PRs open >7 days with no activity
- Large PRs (>500 lines) that need breaking up
- PRs with requested changes that haven't been addressed
```

### PR Review Prompt

For a specific PR:

```
Review PR #[NUMBER] in [owner/repo].

Steps:
1. `gh pr view [NUMBER] --repo [owner/repo]` — get full description
2. `gh pr diff [NUMBER] --repo [owner/repo]` — get the diff

Review for:
1. **Does the code do what the PR says it does?**
2. **Are there obvious bugs or edge cases missed?**
3. **Are there tests for new functionality?**
4. **Does it follow the existing code patterns?**
5. **Anything security-sensitive?**

Output:
- Summary of what the PR does
- Issues found (CRITICAL / IMPORTANT / MINOR)
- Approval recommendation: APPROVE / REQUEST CHANGES / NEEDS DISCUSSION
```

---

## CI Monitoring

### CI Status Check Prompt

```
CI status check for all repos in github/repos.md.

For each repo, run:
`gh run list --repo [owner/repo] --limit 10 --json status,conclusion,name,createdAt,headBranch`

Report:
- Latest run: ✅ passing / ❌ failing / ⏳ in progress
- If failing: which workflow and what branch
- If pattern of failures: note the trend

For any failing CI:
`gh run view [run-id] --repo [owner/repo] --log-failed`
Summarize the error.

Alert if: main branch CI has been failing for >24h
```

### CI Failure Debug Prompt

```
Debug CI failure for [owner/repo], run [RUN-ID].

1. `gh run view [RUN-ID] --repo [owner/repo] --log-failed`
2. Identify the failing step
3. Extract the error message
4. Suggest the most likely fix

Common CI failures:
- Lint errors → specific code fix
- Test failures → which test and what it expected vs. got
- Build errors → dependency or config issue
- Timeout → test is too slow or hangs

Provide a concrete fix suggestion, not just a diagnosis.
```

---

## Issue Triage Framework

### Issue Triage Prompt

```
Triage new issues for [owner/repo].

Fetch: `gh issue list --repo [owner/repo] --state open --sort created --json number,title,body,labels,createdAt`

For each unlabeled issue (or issues from last 48h):

Categorize as:
- 🐛 **bug** — something is broken
- ✨ **feature** — new functionality requested
- 📖 **docs** — documentation improvement
- ❓ **question** — user needs help (not a bug)
- 🔧 **maintenance** — internal/technical debt

Assess priority:
- P1: Critical — blocking users, security issue, data loss
- P2: High — significant UX impact, affects many users
- P3: Medium — improvement, affects some users
- P4: Low — nice-to-have, cosmetic

Output suggested labels and priority for each issue.
Ask before applying labels — deliver as recommendations.
```

### Stale Issue Detector

```
Find stale issues in [owner/repo].

Definition of stale:
- Open issues with no activity (comments/updates) in 30+ days
- Issues with labels: question, waiting-for-response

Run: `gh issue list --repo [owner/repo] --state open --json number,title,updatedAt,labels`

Filter to stale ones. For each:
1. Issue number and title
2. Days since last activity
3. Current labels
4. Suggested action: CLOSE (no response in 30d) / BUMP (ask for update) / KEEP (active discussion)

Output as a table for review before any action.
```

---

## Contributor Activity Tracking

### Contribution Stats Prompt

```
Contributor activity for [owner/repo] — last 30 days.

Data to fetch:
- Commits by author: `gh api repos/[owner/repo]/stats/contributors`
- PRs by author: `gh pr list --repo [owner/repo] --state all --limit 50 --json author,createdAt,mergedAt`

Report:
1. Most active contributors (commits + PRs)
2. New contributors this month (first PR merged)
3. Contributors who've gone quiet (active before, not this month)
4. Bus factor: how many contributors account for 80% of commits?

Write to github/contributors/[MONTH].md
```

---

## Release Notes Generator

```
Generate release notes for [owner/repo] — since [last-tag/date].

1. `gh release list --repo [owner/repo]` — find last release
2. `gh pr list --repo [owner/repo] --state merged --base main --limit 50 --json number,title,body,mergedAt`
   Filter to PRs merged since last release

Categorize merged PRs as:
- 🚀 New Features
- 🐛 Bug Fixes
- 🔧 Improvements
- 📖 Documentation
- ⚠️ Breaking Changes

Format as a GitHub release body:
---
## What's Changed

### 🚀 New Features
- [Feature description] (#[PR number])

### 🐛 Bug Fixes
- [Fix description] (#[PR number])
---

Output is ready to paste into a GitHub release.
```

---

## Open Source Contribution Finder

Find good issues to contribute to:

```
Find open source contribution opportunities.

I want to contribute to projects in: [DOMAIN/LANGUAGE/TOPIC]

Search GitHub for:
`gh search issues --label "good first issue" --label "help wanted" --language [LANGUAGE] --state open --limit 20`

For each result:
1. Repository name and stars
2. Issue title
3. Complexity estimate (from description)
4. Activity (last comment date)

Filter for:
- Repos with >100 stars (established projects)
- Issues updated in last 30 days (active)
- Not already claimed by another contributor

Output a ranked list with reasoning.
```

---

## Troubleshooting

**gh auth fails**
- Try: `gh auth logout && gh auth login`
- For CI environments: use `gh auth login --with-token` with a PAT

**API rate limits**
- gh CLI uses authenticated requests (5,000/hour vs 60/hour unauthenticated)
- If hitting limits: cache gh output to files, don't re-fetch same data

**PR diff too large**
- Some diffs exceed context limits
- Use: `gh pr diff [NUMBER] --repo [owner/repo] | head -200`
- Or focus on specific files: `gh pr diff [NUMBER] -- src/specific-file.js`

**CI logs not found**
- Run IDs change — always get the latest: `gh run list --repo [owner/repo] --limit 1`
- Some repos require explicit permission to view logs

---

*Built on OpenClaw and gh CLI. Requires GitHub CLI authenticated.*
