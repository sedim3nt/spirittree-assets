# The Open Source Contribution System
## From Issue Discovery to Merged PR — An Agent-Assisted Workflow

Contributing to open source builds reputation, sharpens skills, and creates leverage in the communities that matter to your work. This system makes it systematic.

---

## Why Systematic Contribution?

Random contribution is slow and demoralizing. Systematic contribution means:
- Finding the right issues for your skill level
- Evaluating repos worth investing in
- Moving from first PR to trusted contributor faster
- Using your agent to accelerate research and draft generation

---

## Phase 1: Issue Discovery with `gh` CLI

### Filter for Beginner-Friendly Issues

```bash
# Find good-first-issue labeled issues in your language/ecosystem
gh issue list \
  --repo [owner/repo] \
  --label "good first issue" \
  --state open \
  --limit 20

# Or search across GitHub (requires gh extension or API)
gh api graphql -f query='
{
  search(query: "label:\"good first issue\" language:JavaScript is:open is:issue", type: ISSUE, first: 20) {
    nodes {
      ... on Issue {
        title
        url
        repository { nameWithOwner }
        createdAt
        labels(first: 5) { nodes { name } }
      }
    }
  }
}'
```

### Filter for Help-Wanted Issues

```bash
gh issue list \
  --repo [owner/repo] \
  --label "help wanted" \
  --state open \
  --assignee "" \  # Unassigned only
  --limit 20
```

### Search by Topic and Language

```bash
# Search for Python repos needing help with docs
gh api graphql -f query='
{
  search(query: "label:\"good first issue\" label:documentation language:Python is:open is:issue", type: ISSUE, first: 15) {
    nodes {
      ... on Issue {
        title
        url
        body
        repository { nameWithOwner stargazerCount }
      }
    }
  }
}'
```

### Automate Weekly Issue Discovery

Add to cron in AGENTS.md:
```markdown
### Weekly OSS Issue Hunt
Schedule: 0 9 * * 1  (Monday 9am)
Task: Search GitHub for good-first-issue in [your language/ecosystem] repos, 
      filter for repos with 100-10000 stars, unassigned issues, 
      created in last 30 days. Deliver 5 best candidates to Telegram.
```

---

## Phase 2: Repository Evaluation Framework

Not every repo is worth contributing to. Use these 7 criteria before investing time.

### Criterion 1: Maintenance Activity

```bash
# Check when last commit happened
gh repo view [owner/repo] --json updatedAt,pushedAt

# Check recent commit frequency
gh api repos/[owner/repo]/commits --jq '.[0:10] | .[] | .commit.author.date'
```

✅ Active: commits within last 30 days
⚠️ Slow: commits within 90 days
❌ Stale: no commits in 90+ days (contributions may sit unreviewed)

### Criterion 2: PR Merge Rate

```bash
# Look at recent PR activity
gh pr list --repo [owner/repo] --state closed --limit 20
```

Look at whether PRs are being merged or just accumulating. High open PR count with low merge rate = maintainer bottleneck.

### Criterion 3: Response Time on Issues

Check how quickly maintainers respond to new issues. Search for recently opened issues and note if there are maintainer responses.

```bash
gh issue list --repo [owner/repo] --state open --limit 10 --json number,title,createdAt,comments
```

Target: maintainers responding within a week. Long silences mean your PR may wait months.

### Criterion 4: Contribution Guidelines Exist

```bash
gh api repos/[owner/repo]/contents/CONTRIBUTING.md 2>/dev/null | head -20
# Or
ls in the repo: CONTRIBUTING.md, .github/CONTRIBUTING.md, docs/contributing.md
```

Repos without contribution guidelines create ambiguity. You'll waste time on PRs that get rejected for style/process reasons.

### Criterion 5: Code Quality and Testing

```bash
# Check if tests exist
gh api repos/[owner/repo]/contents --jq '.[] | .name' | grep -i test

# Check CI configuration
gh api repos/[owner/repo]/contents/.github/workflows --jq '.[].name' 2>/dev/null
```

Repos with good test coverage and CI make your contributions more likely to get merged — there's an objective check.

### Criterion 6: License Compatibility

```bash
gh repo view [owner/repo] --json licenseInfo
```

Make sure the license aligns with how you want your contributions used:
- MIT/Apache 2.0: permissive, generally fine
- GPL: copyleft, fine for tools but understand implications
- No license: legally ambiguous — be cautious

### Criterion 7: Community Health

```bash
# Check issue/PR templates
gh api repos/[owner/repo]/contents/.github --jq '.[].name' 2>/dev/null

# Look at discussion tone in recent issues
gh issue view [issue-number] --repo [owner/repo] --comments
```

Hostile maintainers and toxic communities aren't worth your time regardless of how popular the repo is. Read a few comment threads before investing.

---

## Phase 3: PR Generation Workflow

### Step 1: Claim the Issue

Before starting work, comment on the issue:

```bash
gh issue comment [issue-number] --repo [owner/repo] \
  --body "Hi, I'd like to work on this. I'm planning to [brief description of approach]. 
  Expected timeline: [2–3 days / this weekend]. 
  Let me know if there's a preferred approach or anything to avoid."
```

This prevents duplicate work and signals you're serious.

### Step 2: Fork and Set Up

```bash
# Fork the repo
gh repo fork [owner/repo] --clone

# Set upstream
cd [repo-name]
git remote add upstream https://github.com/[owner/repo].git

# Create a feature branch
git checkout -b fix/[issue-description]-[issue-number]
# Example: git checkout -b fix/null-pointer-crash-247
```

### Step 3: Use Your Agent to Research the Codebase

Before writing any code, gather context:

```bash
# In Telegram, send to Dev topic:
"Research the [owner/repo] codebase. 
Issue #[number]: [issue description].
I need to understand:
1. Which files are relevant to this issue
2. How similar problems have been solved previously
3. What tests exist for this area
4. Any related issues or PRs I should know about
Read the issue body and find the relevant code."
```

Your agent can read GitHub issues, search the codebase, and synthesize a work plan faster than you can manually.

### Step 4: Implement the Fix

Follow the Blueprint pattern from AGENTS.md:
1. PRE-FETCH: gather all relevant files
2. Implement the fix
3. Run tests locally: `npm test`, `pytest`, `cargo test`, etc.
4. Fix any failures
5. Run one more time to confirm clean

### Step 5: Write Tests

```bash
# Ask your agent to write tests
"Write unit tests for the fix I just made in [file].
The fix does: [description]
The existing test patterns are in [test file].
Match the existing style."
```

PRs without tests are harder to merge. If the existing codebase has tests, add them.

---

## Documentation Improvement Patterns

Documentation PRs are often the fastest path to getting merged — maintainers love them.

**High-value documentation targets:**
- README sections that are confusing or outdated
- Missing examples in API documentation
- Typos and grammatical errors (small but easy wins)
- "Getting started" guides that assume too much
- Missing error message explanations

**Finding documentation issues:**
```bash
gh issue list --repo [owner/repo] --label documentation --state open
```

**Agent-assisted doc improvement:**
```
"Read the README for [owner/repo] at [URL].
Identify the 3 most confusing sections for a new user.
Draft improved versions for each section.
Keep the same tone and style as the existing docs."
```

---

## Commit Message Conventions

Follow the Conventional Commits spec — most serious repos use it:

```
<type>(<scope>): <short description>

<optional body>

<optional footer>
```

**Types:**
- `fix:` — bug fix
- `feat:` — new feature
- `docs:` — documentation only
- `test:` — adding tests
- `refactor:` — code change that neither fixes nor adds feature
- `chore:` — maintenance, dependency updates

**Examples:**
```
fix(auth): handle null user when session expires

Fixes a crash when users with expired sessions 
attempt to access protected routes.

Closes #247
```

```
docs(readme): add example for custom configuration

The README was missing a concrete example for the 
custom config option. Added a working example with
expected output.
```

---

## Maintainer Etiquette

Getting PRs merged is partly technical and partly social.

**Do:**
- Ask before starting work on large changes
- Keep PRs focused (one issue = one PR)
- Respond quickly to review feedback
- Be gracious when your approach is rejected
- Test thoroughly before submitting

**Don't:**
- Submit massive PRs that change everything
- Ignore CI failures ("it works on my machine")
- Argue with maintainers about style preferences
- Abandon PRs mid-review
- Add features that weren't requested

**When feedback comes in:**
```bash
# Acknowledge and commit fixes
git add [changed files]
git commit -m "fix: address review feedback from @maintainer"
git push origin fix/your-branch

# Then respond to the review
gh pr comment [pr-number] --repo [owner/repo] \
  --body "Thanks for the review. I've addressed [feedback point 1] and [feedback point 2].
  The approach I took was [explanation] because [reason].
  Let me know if you'd like me to adjust anything else."
```

---

## Contribution Tracking

Keep a simple log in `memory/oss-contributions.md`:

```markdown
# Open Source Contributions

## Active
| Repo | Issue | PR | Status | Started |
|------|-------|----|--------|---------|
| owner/repo | #247 | #301 | Under review | 2026-03-10 |

## Completed (Merged)
| Repo | PR | Type | Merged | Impact |
|------|----|----|--------|--------|
| owner/repo | #298 | fix | 2026-02-15 | Fixed crash for 200 users |

## Rejected / Abandoned
| Repo | PR | Reason |
|------|----|--------|
| owner/repo | #312 | Maintainer wants different approach |

## Repos to Watch
- owner/repo — good community, active development
- owner/repo2 — interesting project, needs docs help
```

---

## Reputation Building Strategy

**The compound contribution model:**

Month 1–2: 3–5 small PRs in the same repo (fixes, docs)
Month 3: Medium PR (feature or significant fix), get known by maintainers
Month 4–6: Invited to review others' PRs, become a trusted contributor
Month 6+: Maintainer status in repos you care about

**Cross-repo visibility:**
- Write about what you contributed and why on X/Substack
- Reference your merged PRs in your GitHub profile README
- Comment helpfully on issues you're not working on (signals expertise)
- Open issues with detailed reproduction steps — maintainers notice quality reports

**Your GitHub README profile:**
```bash
# Create profile README if you don't have one
gh repo create [your-username]/[your-username] --public
# Add README.md with contribution stats, featured projects
```

---

## First PR Template

Copy this for your first PR in any repo:

**PR Title:** `fix(scope): short description of what changed`

**PR Body:**

```markdown
## Summary
Brief description of what this PR does and why.

## Changes
- [File 1]: What changed and why
- [File 2]: What changed and why

## Testing
How I tested this:
- [ ] Existing tests pass (`npm test` / `pytest` / etc.)
- [ ] Added new tests for [specific scenario]
- [ ] Manually tested by [specific actions]

## Related
Closes #[issue number]

## Notes for Reviewer
[Anything that would help the reviewer understand your approach, trade-offs you considered, or areas where you're uncertain]
```

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
