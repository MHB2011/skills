# Ralph File Templates

Two modes: **GitHub** and **Local**. Each mode has its own once.sh, afk.sh, and prompt.md.

Replace `<ralph-dir>` with the actual ralph directory path and `<feedback-loop-commands>` with the project's actual commands.

---

## Docker Sandbox Setup (required for AFK Ralph)

AFK Ralph uses `docker sandbox run claude .` to isolate Claude in a container. This is critical — without it, an autonomous agent with skipped permissions could delete files or break your system.

### Prerequisites

1. **Install Docker Desktop** — https://docs.docker.com/get-started/get-docker/
2. **Install jq** — `brew install jq` (macOS) or `apt install jq` (Linux)

### First-time setup

```bash
docker sandbox run claude .
```

First run creates a sandbox environment and pulls the image. You'll need to:
- Log in to Claude Code (subscription or console account)
- Accept the safety check and trust the folder

### Verify sandboxing works

Inside the sandbox, test these two prompts:
- `Are we using npm or pnpm?` — should correctly detect your project
- `Grab me a file from my Downloads folder` — should fail with "no Downloads directory in this environment"

### GitHub mode: authenticate gh CLI

Inside the sandbox, run `! gh auth login` and complete the OAuth device flow.

---

# GitHub Mode

Tasks come from GitHub issues. Ralph auto-commits and manages issues via `gh` CLI. Not gitignored — visible to the team.

## once.sh — GitHub

```bash
#!/bin/bash
# HITL Ralph — run Claude once, picking task from GitHub issues
# Usage: ./<ralph-dir>/once.sh

issues=$(gh issue list --state open --json number,title,body,comments)
commits=$(git log -n 5 --format="%H%n%ad%n%B---" --date=short 2>/dev/null || echo "No commits found")
prompt=$(cat <ralph-dir>/prompt.md)

claude --permission-mode acceptEdits \
  "Previous commits: $commits $issues $prompt"
```

## afk.sh — GitHub

```bash
#!/bin/bash
# AFK Ralph — loop Claude until done or max iterations
# Usage: ./<ralph-dir>/afk.sh 5
set -eo pipefail

if [ -z "$1" ]; then
  echo "Usage: $0 <iterations>"
  exit 1
fi

# jq filter to extract streaming text from assistant messages
stream_text='select(.type == "assistant").message.content[]? | select(.type == "text").text // empty | gsub("\n"; "\r\n") | . + "\r\n\n"'

# jq filter to extract final result
final_result='select(.type == "result").result // empty'

for ((i=1; i<=$1; i++)); do
  echo "=== Ralph iteration $i of $1 ==="
  tmpfile=$(mktemp)
  trap "rm -f $tmpfile" EXIT

  issues=$(gh issue list --state open --json number,title,body,comments)
  commits=$(git log -n 5 --format="%H%n%ad%n%B---" --date=short 2>/dev/null || echo "No commits found")
  prompt=$(cat <ralph-dir>/prompt.md)

  docker sandbox run claude . -- \
    --verbose \
    --print \
    --output-format stream-json \
    "Previous commits: $commits $issues $prompt" \
  | grep --line-buffered '^{' \
  | tee "$tmpfile" \
  | jq --unbuffered -rj "$stream_text"

  result=$(jq -r "$final_result" "$tmpfile")

  if [[ "$result" == *"<promise>NO MORE TASKS</promise>"* ]]; then
    echo "Ralph complete after $i iterations."
    exit 0
  fi
done

echo "Ralph reached max iterations ($1)."
```

## prompt.md — GitHub

```markdown
# ISSUES

GitHub issues are provided at start of context. Parse it to get open issues with their bodies and comments.

You've also been passed a file containing the last few commits. Review these to understand what work has been done.

If all tasks are complete, output <promise>NO MORE TASKS</promise>.

# TASK SELECTION

Pick the next task. Prioritize tasks in this order:

1. Critical bugfixes
2. Development infrastructure

Getting development infrastructure like tests and types and dev scripts ready is an important precursor to building features.

3. Tracer bullets for new features

Tracer bullets are small slices of functionality that go through all layers of the system, allowing you to test and validate your approach early. This helps in identifying potential issues and ensures that the overall architecture is sound before investing significant time in development.

TL;DR - build a tiny, end-to-end slice of the feature first, then expand it out.

4. Polish and quick wins
5. Refactors

# EXPLORATION

Explore the repo.

# IMPLEMENTATION

Complete the task.

# FEEDBACK LOOPS

Before committing, run the feedback loops:

<feedback-loop-commands>

# COMMIT

Make a git commit. The commit message must:

1. Include key decisions made
2. Include files changed
3. Blockers or notes for next iteration

# THE ISSUE

If the task is complete, close the original GitHub issue.

If the task is not complete, leave a comment on the GitHub issue with what was done.

# FINAL RULES

ONLY WORK ON A SINGLE TASK.
```

---

# Local Mode

Tasks come from MD files in `<ralph-dir>/backlog/`. Progress tracked in `<ralph-dir>/history.md`. Gitignored — personal workflow, invisible to the team.

## once.sh — Local

```bash
#!/bin/bash
# HITL Ralph — run Claude once, picking task from local backlog
# Usage: ./<ralph-dir>/once.sh

# Read open/in-progress backlog items
backlog=""
for f in <ralph-dir>/backlog/*.md; do
  if [ -f "$f" ] && grep -qE "^status: (open|in-progress)" "$f"; then
    backlog="$backlog
--- FILE: $(basename "$f") ---
$(cat "$f")"
  fi
done

# Read last 5 history entries
history=$(awk 'BEGIN{RS="---"} /[^ \t\n]/{a[++n]=$0} END{s=n-4; if(s<1)s=1; for(i=s;i<=n;i++) printf "%s\n---\n", a[i]}' <ralph-dir>/history.md 2>/dev/null || echo "No history found")

prompt=$(cat <ralph-dir>/prompt.md)

claude --permission-mode acceptEdits \
  "Previous history: $history Backlog: $backlog $prompt"
```

## afk.sh — Local

```bash
#!/bin/bash
# AFK Ralph — loop Claude until done or max iterations
# Usage: ./<ralph-dir>/afk.sh 5
set -eo pipefail

if [ -z "$1" ]; then
  echo "Usage: $0 <iterations>"
  exit 1
fi

# jq filter to extract streaming text from assistant messages
stream_text='select(.type == "assistant").message.content[]? | select(.type == "text").text // empty | gsub("\n"; "\r\n") | . + "\r\n\n"'

# jq filter to extract final result
final_result='select(.type == "result").result // empty'

for ((i=1; i<=$1; i++)); do
  echo "=== Ralph iteration $i of $1 ==="
  tmpfile=$(mktemp)
  trap "rm -f $tmpfile" EXIT

  # Read open/in-progress backlog items
  backlog=""
  for f in <ralph-dir>/backlog/*.md; do
    if [ -f "$f" ] && grep -qE "^status: (open|in-progress)" "$f"; then
      backlog="$backlog
--- FILE: $(basename "$f") ---
$(cat "$f")"
    fi
  done

  # Read last 5 history entries
  history=$(awk 'BEGIN{RS="---"} /[^ \t\n]/{a[++n]=$0} END{s=n-4; if(s<1)s=1; for(i=s;i<=n;i++) printf "%s\n---\n", a[i]}' <ralph-dir>/history.md 2>/dev/null || echo "No history found")

  prompt=$(cat <ralph-dir>/prompt.md)

  docker sandbox run claude . -- \
    --verbose \
    --print \
    --output-format stream-json \
    "Previous history: $history Backlog: $backlog $prompt" \
  | grep --line-buffered '^{' \
  | tee "$tmpfile" \
  | jq --unbuffered -rj "$stream_text"

  result=$(jq -r "$final_result" "$tmpfile")

  if [[ "$result" == *"<promise>NO MORE TASKS</promise>"* ]]; then
    echo "Ralph complete after $i iterations."
    exit 0
  fi
done

echo "Ralph reached max iterations ($1)."
```

## prompt.md — Local

```markdown
# BACKLOG

Local backlog items are provided at start of context. Each item is a markdown file with frontmatter (title, type, status, priority). Parse them to find open items.

You've also been passed recent work history entries. Review these to understand what work has been done.

If all tasks are complete, output <promise>NO MORE TASKS</promise>.

# TASK SELECTION

Pick the next task. Prioritize by the `priority` field in frontmatter:

1. Critical bugfixes (priority: 1)
2. Development infrastructure (priority: 2)

Getting development infrastructure like tests and types and dev scripts ready is an important precursor to building features.

3. Tracer bullets for new features (priority: 3)

Tracer bullets are small slices of functionality that go through all layers of the system, allowing you to test and validate your approach early. This helps in identifying potential issues and ensures that the overall architecture is sound before investing significant time in development.

TL;DR - build a tiny, end-to-end slice of the feature first, then expand it out.

4. Polish and quick wins (priority: 4)
5. Refactors (priority: 5)

# EXPLORATION

Explore the repo.

# IMPLEMENTATION

Complete the task.

# FEEDBACK LOOPS

Before finishing, run the feedback loops:

<feedback-loop-commands>

# HISTORY

Do NOT make a git commit. Instead, append an entry to `<ralph-dir>/history.md` using this exact format:

## <current date and time>

**Task:** <what was completed>
**Key decisions:** <decisions made>
**Files changed:** <files changed>
**Notes for next iteration:** <blockers or notes>

---

The `---` separator at the end is critical — the scripts use it to parse the last 5 entries.

# THE BACKLOG ITEM

If the task is complete, update the backlog file's frontmatter to `status: closed`.

If the task is not complete, update the frontmatter to `status: in-progress` and append a comment at the bottom of the backlog file:

```
## Comment — <date>
<what was done, key decisions, blockers>
```

# FINAL RULES

ONLY WORK ON A SINGLE TASK.
```

---

# history.md

Created empty in local mode. Ralph appends entries as it works. Each entry separated by `---` so scripts can extract the last 5.

Example after two iterations:

```markdown
## 2026-04-07T14:30

**Task:** Phase 1 — Project scaffold + course browsing
**Key decisions:** Used React Router v7 with file-based routing, set up Tailwind v4
**Files changed:** app/root.tsx, app/routes/home.tsx, package.json
**Notes for next iteration:** Need to implement search functionality

---

## 2026-04-07T15:45

**Task:** Phase 2 — Search and filtering
**Key decisions:** Used Fuse.js for client-side fuzzy search
**Files changed:** app/routes/search.tsx, app/components/SearchBar.tsx
**Notes for next iteration:** Consider adding server-side search for large datasets

---
```

---

# Local backlog file format

Each file in `<ralph-dir>/backlog/` represents a single task. Filename should be kebab-case (e.g., `admin-analytics-dashboard.md`).

### Frontmatter

```yaml
---
title: "Admin analytics dashboard"
type: prd
status: open
priority: 3
created: 2026-04-07
---
```

**Fields:**

- **title** — Short descriptive title
- **type** — One of: `prd`, `plan`, `bug`, `task`, `refactor`, `infrastructure`
- **status** — One of: `open`, `in-progress`, `closed`
- **priority** — 1 through 5, maps to task selection order:
  - 1 = Critical bugfix
  - 2 = Development infrastructure
  - 3 = New feature
  - 4 = Polish / quick win
  - 5 = Refactor
- **created** — Date the item was created

### Body

The body after frontmatter contains the task description. For PRDs, this is the full PRD content. For bugs, a description of the problem and expected behavior.

### Comments

Ralph appends comments at the bottom when a task is partially complete:

```markdown
## Comment — 2026-04-07

Completed the service layer and route setup. Summary cards rendering with mock data.
Still need to wire up real database queries.
```

### Example: PRD backlog item

```markdown
---
title: "Instructor analytics dashboard"
type: prd
status: open
priority: 3
created: 2026-04-07
---

## Problem Statement

Instructors have no way to view their course performance metrics...

## Solution

Build a read-only analytics dashboard at /instructor/analytics...

## User Stories

1. As an instructor, I want to see total revenue across all my courses...
```

### Example: Bug backlog item

```markdown
---
title: "OBS virtual camera resets on page navigation"
type: bug
status: open
priority: 1
created: 2026-04-07
---

The OBS virtual camera connection drops when navigating between pages.

**Steps to reproduce:**
1. Start OBS virtual camera
2. Navigate from /courses to /courses/1
3. Camera feed freezes

**Expected:** Camera should persist across navigation.
```

---

# README.md — GitHub mode

```markdown
# Ralph

Autonomous coding loop that runs Claude Code repeatedly until a task is complete.

## How it works

1. Ralph reads all open GitHub issues and the last 5 git commits
2. Picks the highest-priority task (bugs > infrastructure > features > polish > refactors)
3. Explores the repo, implements the task, runs feedback loops
4. Commits the code and closes/comments on the GitHub issue
5. In AFK mode, repeats until all tasks are done or max iterations reached

## Files

| File | Purpose |
|------|---------|
| `prompt.md` | Instructions Claude gets each iteration — edit to add project-specific constraints |
| `once.sh` | HITL mode — runs Claude once, you review after each run |
| `afk.sh` | AFK mode — loops Claude autonomously (requires Docker) |

## Usage

### HITL (human-in-the-loop)

\`\`\`bash
./<ralph-dir>/once.sh
\`\`\`

Watch Ralph work, review the commit, then run again for the next task.

### AFK (autonomous)

\`\`\`bash
./<ralph-dir>/afk.sh 5
\`\`\`

Runs up to 5 iterations inside a Docker sandbox. Stops early if all tasks are done.

Requires Docker Desktop and `jq`.

## Adding tasks

Create GitHub issues. Ralph reads them automatically and prioritizes by type:

| Priority | Type |
|----------|------|
| 1 | Critical bugfixes |
| 2 | Development infrastructure (tests, types, tooling) |
| 3 | New features (tracer bullets first, then expand) |
| 4 | Polish and quick wins |
| 5 | Refactors |

PRDs and plans go directly into issues. Ralph picks the next task each iteration.

## Tips

- Keep tasks small — if a task takes more than one iteration, split it
- Ralph reads commit messages to orient itself — detailed commits help the next iteration
- Feedback loops are your safety net — they block broken code
- Don't skip QA — passing checks doesn't mean the feature works correctly
- Review commits before pushing: `git diff HEAD~1`
```

# README.md — Local mode

```markdown
# Ralph

Autonomous coding loop that runs Claude Code repeatedly until a task is complete.

This is a local-only setup — gitignored, no external dependencies, invisible to your team.

## How it works

1. Ralph reads all open backlog items from `backlog/` and the last 5 entries from `history.md`
2. Picks the highest-priority task based on frontmatter
3. Explores the repo, implements the task, runs feedback loops
4. Appends a progress entry to `history.md` and updates the backlog item status
5. In AFK mode, repeats until all tasks are done or max iterations reached

Ralph does NOT make git commits — you review the changes and commit manually.

## Files

| File | Purpose |
|------|---------|
| `prompt.md` | Instructions Claude gets each iteration — edit to add project-specific constraints |
| `once.sh` | HITL mode — runs Claude once, you review after each run |
| `afk.sh` | AFK mode — loops Claude autonomously (requires Docker) |
| `history.md` | Progress log — Ralph appends entries here instead of git commits |
| `backlog/` | Task queue — each `.md` file is a task with frontmatter |

## Usage

### HITL (human-in-the-loop)

\`\`\`bash
./<ralph-dir>/once.sh
\`\`\`

Watch Ralph work, review the changes, then run again for the next task. Commit manually when you're happy.

### AFK (autonomous)

\`\`\`bash
./<ralph-dir>/afk.sh 5
\`\`\`

Runs up to 5 iterations inside a Docker sandbox. Stops early if all tasks are done.

Requires Docker Desktop and `jq`.

## Adding tasks

Create `.md` files in `backlog/` with this format:

\`\`\`yaml
---
title: "Your task title"
type: prd | plan | bug | task | refactor | infrastructure
status: open
priority: 1-5
created: YYYY-MM-DD
---

Task description here...
\`\`\`

Or use `/draft-prd` to create PRDs interactively.

### Priority

| Priority | Type |
|----------|------|
| 1 | Critical bugfixes |
| 2 | Development infrastructure (tests, types, tooling) |
| 3 | New features (tracer bullets first, then expand) |
| 4 | Polish and quick wins |
| 5 | Refactors |

### Task lifecycle

- `open` → Ralph picks it up
- `in-progress` → Ralph is working on it, comments appended at bottom
- `closed` → Done

## Tips

- Keep tasks small — if a task takes more than one iteration, split it
- `history.md` is your context — Ralph reads recent entries to orient itself
- Feedback loops are your safety net — they block broken code
- Don't skip QA — passing checks doesn't mean the feature works correctly
- Review changes before committing: `git diff`
```
