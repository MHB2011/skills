---
name: setup-ralph
description: Set up Ralph (HITL + AFK) autonomous coding loops in any project. Creates scripts (once.sh, afk.sh), prompt template, and README guide. Use when user wants to set up Ralph, add autonomous coding loops, or mentions "ralph" in the context of project setup.
---

# Setup Ralph

Set up Ralph — an autonomous coding loop that runs Claude Code with a plan until the task is complete.

## Prerequisites check

Before proceeding, verify the project has feedback loops. Ralph MUST have at minimum:

- **Typecheck command** (e.g., `tsc --noEmit`, `pnpm run typecheck`) — REQUIRED
- **Lint command** (e.g., `eslint .`, `npm run lint`) — REQUIRED
- **Test command** (e.g., `npm test`, `pnpm run test`) — RECOMMENDED but optional

If typecheck or lint commands don't exist, **stop and tell the user** they need to set up feedback loops first. Ralph without feedback loops will produce broken code. Suggest `/setup-pre-commit` if available.

If there's a single command that runs all checks (e.g., `npm run check-for-errors`), that works too — use it as the feedback loop command.

## Setup workflow

### 1. Ask the user

Two questions drive everything:

1. **GitHub or Local?**

   - **GitHub** — Full autonomous mode. Tasks come from GitHub issues. Ralph auto-commits, closes issues, and leaves comments. Visible to the team. Requires `gh` CLI.
   - **Local** — Everything stays on your machine. Tasks come from MD files in `<ralph-dir>/backlog/`. Progress tracked in `<ralph-dir>/history.md` instead of git commits. Gitignored by default — use Ralph without your team having to know about it. Use `/draft-prd` to create PRDs into the backlog.

2. **HITL or also AFK?** — HITL (once.sh) only, or also set up AFK (afk.sh)? **Strongly recommend starting with HITL** — see how Ralph behaves before letting it run unsupervised.

Then auto-detect or confirm:

3. **Feedback loop commands** — detect from package.json scripts, pre-commit hooks. Ask user to confirm.
4. **Ralph directory path** — default: `ralph/`.

### 2. Create files

See [REFERENCE.md](REFERENCE.md) for exact file contents.

**GitHub mode** creates:
- `prompt.md` — prompt template (GitHub issues variant)
- `once.sh` — HITL script (reads GitHub issues + git commits)
- `README.md` — usage guide

**Local mode** creates:
- `prompt.md` — prompt template (local backlog variant)
- `once.sh` — HITL script (reads local backlog + history.md)
- `history.md` — empty file (Ralph appends progress entries here)
- `backlog/` — directory for backlog MD files
- `README.md` — usage guide
- Add `<ralph-dir>/` to `.gitignore`

**If AFK chosen**, also create:
- `afk.sh` — loops Claude until done or max iterations

Adapt the templates:
- Replace `<feedback-loop-commands>` in `prompt.md` with the user's actual commands
- Replace `<ralph-dir>` in all scripts with the actual ralph directory path

### 3. Make scripts executable

```bash
chmod +x <ralph-dir>/once.sh
# Only if AFK:
chmod +x <ralph-dir>/afk.sh
```

### 4. Docker setup (AFK only)

**Skip this step entirely for HITL.** HITL Ralph does NOT require Docker.

If AFK was chosen, tell the user it requires Docker Desktop and `jq`. Walk them through the setup — see [REFERENCE.md](REFERENCE.md) for Docker sandbox setup instructions and verification steps.

**GitHub mode additional step:** authenticate `gh` CLI inside the Docker sandbox (`gh auth login`).

### 5. Verify and point to README

Show the user the exact command they would run and explain what it does.

- **HITL**: `./<ralph-dir>/once.sh`
- **AFK**: `./<ralph-dir>/afk.sh 5`

Then tell the user: **Read `<ralph-dir>/README.md` for the full guide** — it explains how Ralph works, how to add tasks, the priority system, and tips for getting the best results.
