---
name: prd-to-tasks
description: Break a PRD from Ralph's local backlog into independently grabbable task files using tracer-bullet vertical slices. Use when user wants to convert a local PRD to tasks, break down a PRD into work items, or mentions "prd to tasks".
---

# PRD to Tasks

Break a PRD into independently grabbable task files in `ralph/backlog/` using vertical slices (tracer bullets).

## Process

### 1. Locate the PRD

Ask the user which PRD to break down. Look for PRD files in `ralph/backlog/prd/` (files with `type: prd` in frontmatter).

If no PRDs exist, suggest running `/draft-prd` first.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code.

### 3. Draft vertical slices

Break the PRD into **tracer bullet** tasks. Each task is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be 'HITL' or 'AFK'. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories from the PRD this addresses

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Create the task files

For each approved slice, create a markdown file in `ralph/backlog/`. Filename is kebab-case of the title (e.g., `xp-service-and-schema.md`).

Create files in dependency order (blockers first) so you can reference real filenames in the "Blocked by" field.

<task-template>
---
title: "<Task title>"
type: task
status: open
priority: <inherit from source PRD, default 3>
created: <today's date YYYY-MM-DD>
source_prd: "<prd filename, e.g. gamification.md>"
mode: <hitl | afk>
blocked_by: [<task filenames this is blocked by, e.g. "xp-service-and-schema.md">]
---

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation. Reference specific sections of the source PRD rather than duplicating content.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## User stories addressed

Reference by number from the source PRD:

- User story 3
- User story 7

</task-template>

Do NOT modify the source PRD file.
