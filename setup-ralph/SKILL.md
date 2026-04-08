---
name: setup-ralph
description: Set up Ralph (HITL + AFK) autonomous coding loops in any project. Creates scripts (once.sh, afk.sh), prompt template, and README guide. Use when user wants to set up Ralph, add autonomous coding loops, or mentions "ralph" in the context of project setup.
---

# Setup Ralph

Set up Ralph — an autonomous coding loop that runs Claude Code with a plan until the task is complete.

## Existing setup detection

Before doing anything, check if `ralph/` (or a custom ralph directory) already exists in the project.

**If it exists**, read the existing files to understand the current setup:
- Check if `once.sh` exists → HITL is set up
- Check if `afk.sh` exists → AFK is set up
- Check if `backlog/` exists → Local mode
- Check if `once.sh` references `gh issue list` → GitHub mode

Then tell the user what's already set up and offer the specific next step:

**Missing pieces:**
- Has HITL but no AFK → "Want to add **AFK**?"
- Has AFK but no HITL → "Want to add **HITL**?" (once.sh may have been deleted)
- Has both → tell the user: "Ralph is already fully set up (**<mode> HITL + AFK**)." Then ask: "Want a clean reinstall?" If yes, delete the existing scripts (`once.sh`, `afk.sh`, `prompt.md`, `README.md`) and proceed with full setup. Preserve `backlog/` and `history.md` — those contain user data.

**Mode switch:**
- Has GitHub, wants Local (or vice versa) → warn that this will overwrite `prompt.md`, `once.sh`, and `afk.sh` (if it exists). Ask for confirmation before proceeding. Preserve any existing `backlog/` or `history.md` files.

Only create/overwrite files that are needed. Do NOT recreate files that already exist and are correct. Do NOT ask open-ended questions about changing the existing setup — just offer the specific missing piece.

**If it doesn't exist**, proceed with full setup below.

## Prerequisites check

Before proceeding, verify the project has feedback loops. Ralph MUST have at minimum:

- **Typecheck command** (e.g., `tsc --noEmit`, `pnpm run typecheck`) — REQUIRED
- **Lint command** (e.g., `eslint .`, `npm run lint`) — REQUIRED
- **Test command** (e.g., `npm test`, `pnpm run test`) — RECOMMENDED but optional

If typecheck or lint commands don't exist, **stop and tell the user** they need to set up feedback loops first. Ralph without feedback loops will produce broken code. Suggest `/setup-pre-commit` if available.

If there's a single command that runs all checks (e.g., `npm run check-for-errors`), that works too — use it as the feedback loop command.

## Environment checks

**Before asking any questions**, check what's available:

1. **Docker** — run `docker --version`
   - If missing: "Docker Desktop is not installed. AFK Ralph requires Docker to run Claude in an isolated sandbox." Provide install link: https://docs.docker.com/get-started/get-docker/. Only offer HITL.
   - Also check `jq --version` — if missing, tell them to install it (`brew install jq` on macOS, `apt install jq` on Linux).

2. **GitHub CLI** — run `gh --version`
   - If missing: GitHub mode is not available. Only offer Local mode.
   - If present but not authenticated (`gh auth status`): tell the user to run `gh auth login` first.

Only present options that are actually available based on these checks.

## Setup workflow

### 1. Ask the user

Two questions drive everything:

1. **GitHub or Local?**

   - **GitHub** — Full autonomous mode. Tasks come from GitHub issues. Ralph auto-commits, closes issues, and leaves comments. Visible to the team. Requires `gh` CLI.
   - **Local** — Everything stays on your machine. Tasks come from MD files in `<ralph-dir>/backlog/`. Progress tracked in `<ralph-dir>/history.md` instead of git commits. Gitignored by default — use Ralph without your team having to know about it. Use `/draft-task`, `/draft-prd`, `/draft-plan`, or `/grill-to-plan` to add work to the backlog.

2. **HITL or also AFK?** — HITL (once.sh) only, or also set up AFK (afk.sh)? **Strongly recommend starting with HITL** — see how Ralph behaves before letting it run unsupervised. **If Docker is not installed, only offer HITL.**

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
- Replace `<feedback-loop-commands>` in `prompt.md` with the user's actual commands as a markdown bullet list (e.g., `- \`pnpm run test\` to run the tests`). Do NOT use code blocks.
- Replace `<ralph-dir>` in all scripts with the actual ralph directory path

### 3. Make scripts executable

```bash
chmod +x <ralph-dir>/once.sh
# Only if AFK:
chmod +x <ralph-dir>/afk.sh
```

### 4. Docker setup (AFK only)

**Skip this step entirely for HITL.** HITL Ralph does NOT require Docker.

If AFK was chosen, **tell the user they must set up the sandbox before running afk.sh for the first time**:

1. Run `docker sandbox run claude .` — this pulls the image (can take several minutes on first run) and opens Claude Code inside the container.
2. Log in to Claude Code inside the sandbox (subscription or console account).
3. Accept the safety check and trust the folder.
4. Exit the sandbox (Ctrl+C).

After this one-time setup, `afk.sh` will work. Without it, afk.sh will fail with "Not logged in."

**GitHub mode additional step:** also run `! gh auth login` inside the sandbox to authenticate the GitHub CLI.

### 5. Verify and point to README

Show the user the exact command they would run and explain what it does.

- **HITL**: `./<ralph-dir>/once.sh`
- **AFK**: `./<ralph-dir>/afk.sh 5`

Then tell the user: **Read `<ralph-dir>/README.md` for the full guide** — it explains how Ralph works, how to add tasks, the priority system, and tips for getting the best results.
