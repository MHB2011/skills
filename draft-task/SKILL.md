---
name: draft-task
description: Quickly create a task in Ralph's local backlog (ralph/backlog/) without the full PRD process. Use when user wants to add a quick task, bug, or small piece of work to the Ralph backlog, or mentions "draft task".
---

This skill creates a lightweight task file in `ralph/backlog/` — no interview, no PRD ceremony. Just a title, type, priority, and a short description.

## Before starting

Check that `ralph/backlog/` exists. If it doesn't, suggest running `/setup-ralph` first.

Check existing backlog items to avoid duplicates — if a similar task already exists (open or in-progress), tell the user and ask if they still want to create a new one.

## Workflow

1. If the user provided a description inline (e.g., `/draft-task add error toasts on failed API calls`), use it directly. Otherwise, ask them to describe the task in one or two sentences.

2. From the description, auto-detect:

   - **title** — short, descriptive
   - **type** — infer from keywords:
     - `bug`, `fix`, `broken`, `crash`, `error`, `regression` → `bug`
     - `refactor`, `rename`, `extract`, `clean up` → `refactor`
     - `test`, `lint`, `ci`, `types`, `tooling`, `config` → `infrastructure`
     - everything else → `task`
   - **priority** — infer from type and urgency:
     - `bug` with urgent language (crash, broken, blocker) → 1
     - `infrastructure` → 2
     - `task` → 3
     - words like `polish`, `tweak`, `minor`, `nice to have` → 4
     - `refactor` → 5

3. Write the file to `ralph/backlog/<kebab-case-title>.md`:

```markdown
---
title: "<title>"
type: <type>
status: open
priority: <priority>
created: <today's date YYYY-MM-DD>
---

<the user's description, cleaned up if needed but not expanded into a PRD>
```

4. Show the user what was created: path, type, and priority. No confirmation step — this is meant to be fast.

## Batch mode

If the user provides multiple tasks at once (e.g., a list), create a separate backlog file for each one. Show a summary table at the end:

| File | Type | Priority |
|------|------|----------|
| `add-error-toasts.md` | task | 3 |
| `fix-login-crash.md` | bug | 1 |
