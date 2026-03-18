# Coding Agent — AI Development Package

**Version:** 1.0 | **Level:** Advanced | **Setup Time:** 30-45 minutes

Your AI coding partner configured for real work. Blueprint pattern for structured development, tmux workflow for parallel sessions, git workflow templates, and the Ralph loop for iterative refinement. Built on the Forrest architecture.

---

## What's In This Package

- `ROLE.md` for your coding agent
- Blueprint pattern (8 steps)
- tmux setup for parallel coding sessions
- Git workflow templates
- Code review prompts
- Debugging workflow
- Architecture planning templates

---

## ROLE.md — Coding Agent

Copy to `agents/coding/ROLE.md`:

```markdown
# ROLE.md — Coding Agent

**Role:** Coding Agent
**Model:** Claude Sonnet (recommended)
**Channel:** Sub-agent (spawned by orchestrator)

## Responsibilities
- Build, debug, deploy software
- Architecture decisions
- Code review and quality
- Technical documentation
- Test writing

## Blueprint Pattern (MANDATORY for any multi-file task)
1. **PRE-FETCH** — gather all relevant files, docs, context before writing code
2. **LLM LOOP** — generate code / make changes
3. **DETERMINISTIC** — lint, typecheck, test (NO LLM)
4. **LLM LOOP** — interpret results, fix issues
5. **DETERMINISTIC** — lint, typecheck, test (NO LLM)
6. **CAP** — max 2 rounds of CI feedback, then stop and report
7. **SUBMIT** — PR or report results
8. **SMOKE TEST** — verify one real data sample flows end-to-end

## Pre-Task Checklist
Before writing any code:
- [ ] Read all relevant existing files
- [ ] Understand current architecture
- [ ] Clarify acceptance criteria
- [ ] Generate architecture diagram (Mermaid) for multi-file tasks
- [ ] Identify all files that will need changes

## Constraints
- Generate Mermaid diagram before multi-file tasks
- Subdirectory-scoped rule files over global rules
- Never declare a task "done" without a smoke test
- Max 2 rounds of fix cycles — if still broken, report and stop
- If git push hangs: use GitHub API instead
```

---

## Blueprint Pattern — Detailed Guide

The Blueprint is your guarantee of quality over speed. Use it for any task touching 3+ files.

### Step 1: PRE-FETCH

Before writing a single line of code, gather:

```
Pre-fetch phase for: [TASK NAME]

I need to read:
1. All files that will be modified
2. Any configuration files
3. Test files for context
4. README or docs for architecture context
5. Package.json / requirements.txt for dependencies

List all files and read them before proceeding.
Do NOT start coding until pre-fetch is complete.
```

### Step 2: Architecture Planning (for multi-file tasks)

Generate a Mermaid diagram first:

```
Before coding, generate a Mermaid architecture diagram showing:
1. All components involved
2. How data flows between them
3. Which files map to which components
4. External dependencies

Format:
\`\`\`mermaid
flowchart TD
  A[Component A] --> B[Component B]
  B --> C[API Call]
  C --> D[(Database)]
\`\`\`

Review this before proceeding. If the architecture looks wrong, fix it here — not after 200 lines of code.
```

### Step 3: LLM Loop — Code Generation

Once pre-fetch is complete and architecture is clear:

```
Implement: [SPECIFIC TASK]

Constraints:
- Work within the existing architecture (do not redesign unless asked)
- Follow existing code patterns and conventions
- Type everything (TypeScript) / document everything (Python)
- Write the simplest thing that works first
- Comment non-obvious logic

After implementing, do NOT run tests yet — first review the logic yourself.
```

### Step 4: Deterministic Checks

Run these yourself (no LLM):

```bash
# TypeScript / Node
npm run typecheck
npm run lint
npm test

# Python
python -m mypy .
python -m flake8 .
pytest

# Go
go vet ./...
go test ./...
```

### Step 5: Fix Cycle

Pass the deterministic output back to the LLM:

```
These are the results from lint/typecheck/test:

[PASTE OUTPUT]

Fix only what's failing. Do not refactor other areas while fixing.
After fixing, I'll run tests again.
```

**CAP RULE:** After 2 fix cycles, if it's still broken, stop and report. Don't keep guessing.

### Step 6: Smoke Test

Before calling a task done:

```
Smoke test for: [TASK NAME]

Run with real data:
1. [Setup step with real values]
2. [Trigger step]
3. Expected output: [what success looks like]

If the smoke test fails, diagnose and fix. If it passes, report complete.
```

---

## tmux Workflow Setup

### Install and configure tmux

```bash
brew install tmux
```

Create `~/.tmux.conf`:

```bash
# Sensible defaults
set -g default-terminal "screen-256color"
set -g history-limit 50000
set -g mouse on

# Easy pane navigation
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Reload config
bind r source-file ~/.tmux.conf \; display "Config reloaded"

# Split panes with | and -
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Status bar
set -g status-bg black
set -g status-fg white
set -g status-left '#[fg=green]#S '
set -g status-right '#[fg=yellow]%H:%M '
```

### Project Session Template

Start a coding session:

```bash
#!/bin/bash
# start-project.sh — launch tmux workspace for a project
PROJECT=$1
PROJ_DIR=~/projects/$PROJECT

tmux new-session -d -s "$PROJECT" -c "$PROJ_DIR"

# Pane layout: editor | terminal | tests
tmux split-window -h -t "$PROJECT" -c "$PROJ_DIR"
tmux split-window -v -t "$PROJECT:0.1" -c "$PROJ_DIR"

# Name panes
tmux send-keys -t "$PROJECT:0.0" "# Editor pane" Enter
tmux send-keys -t "$PROJECT:0.1" "# Terminal pane" Enter
tmux send-keys -t "$PROJECT:0.2" "# Test runner — npm test --watch" Enter

tmux attach-session -t "$PROJECT"
```

---

## Git Workflow Templates

### Branch Naming Convention

```
feature/short-description
fix/bug-description
refactor/what-changed
docs/what-documented
chore/maintenance-task
```

### Commit Message Template

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`

Example:
```
feat(api): add user authentication endpoint

Implements JWT-based auth with refresh tokens.
Follows existing pattern from /api/sessions.

Closes #42
```

### PR Template

Create `.github/pull_request_template.md`:

```markdown
## What This Does
[1-2 sentence description]

## How to Test
1. [Setup step]
2. [Test step]
3. Expected: [what success looks like]

## Checklist
- [ ] Tests pass locally
- [ ] No new lint warnings
- [ ] Smoke test completed
- [ ] Breaking changes documented (if any)

## Screenshots (if UI changes)
[Attach before/after]
```

### Git Workflow Agent Prompt

```
Start a new feature branch and implement: [FEATURE]

Steps:
1. Check current branch: `git branch`
2. Pull latest main: `git pull origin main`
3. Create feature branch: `git checkout -b feature/[slug]`
4. Implement the feature (use Blueprint pattern)
5. Stage changes: `git add -p` (review each chunk)
6. Commit: `git commit -m "feat([scope]): [description]"`
7. Push and create PR

Do NOT push until smoke test passes.
```

---

## Code Review Prompt

Use this to get a thorough code review:

```
Review this code for: [CONTEXT]

[PASTE CODE or file path]

Review criteria:
1. **Correctness** — does it do what it says?
2. **Edge cases** — what could go wrong with unusual inputs?
3. **Security** — any injection risks, auth issues, exposed secrets?
4. **Performance** — any obvious inefficiencies for at-scale use?
5. **Maintainability** — will someone else understand this in 6 months?
6. **Tests** — what's not covered?

Be specific. Line numbers preferred. Prioritize: CRITICAL > IMPORTANT > NICE-TO-HAVE.
```

---

## Debugging Workflow

When something is broken, use this structured approach:

```
Debug: [WHAT IS BROKEN]

Available information:
- Error message: [paste exact error]
- Stack trace: [paste stack trace]
- When it happens: [steps to reproduce]
- What was working before: [last known good state]

Debug process:
1. Form a hypothesis about the cause
2. Identify how to test the hypothesis
3. Run the test
4. If confirmed: fix it. If not: form next hypothesis.

Do NOT just try random fixes. State your hypothesis before each attempt.
```

---

## Architecture Planning Template

For new features or systems:

```
Architecture plan for: [SYSTEM/FEATURE]

Requirements:
- What it must do: [functional requirements]
- What it must NOT do: [constraints]
- Scale requirements: [concurrent users, data volume]
- Integration points: [existing systems it touches]

Deliverable:
1. Mermaid architecture diagram
2. Data model (tables/schemas)
3. API endpoints (if applicable)
4. File structure
5. Implementation order (what to build first)
6. Estimated complexity: LOW / MEDIUM / HIGH with rationale

Review the plan before writing code.
```

---

## Troubleshooting

**Blueprint keeps hitting the 2-round cap**
- Break the task into smaller pieces before starting
- The cap is a feature — it forces you to reconsider the approach
- Ask: "What would need to be true for this to work?" before round 3

**Smoke test is unclear**
- Define success criteria before starting the task
- If you can't write the smoke test, you don't understand the requirements yet
- Ask for clarification rather than building against assumptions

**Git push hangs**
- Use GitHub API instead: `gh api repos/owner/repo/contents/path --method PUT ...`
- Check network connectivity: `ping github.com`
- Try SSH instead of HTTPS: configure `~/.ssh/config`

**Agent modifies unrelated files**
- Always use `git diff --cached` to review before committing
- Be explicit: "Only modify files in src/auth/ — nothing else"
- Use subdirectory-scoped instructions

---

*Built on OpenClaw. Requires OpenClaw installed and configured.*
