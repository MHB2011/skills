---
name: grill-to-plan
description: First interview the user relentlessly about a feature (like grill-me), resolving every branch of the design tree one question at a time. When shared understanding is reached, generate a multi-phase implementation plan using tracer-bullet vertical slices and save it to ralph/backlog/. Use when user wants to go from idea to plan through conversation, mentions "grill to plan", or wants to skip the PRD and plan directly from discussion.
---

# Grill to Plan

Interview the user about a feature idea, then turn that shared understanding into a phased implementation plan saved to Ralph's backlog. No PRD needed — the conversation IS the requirements gathering.

## Before starting

Check that `ralph/backlog/` exists. If it doesn't, ask the user where their Ralph backlog directory is, or suggest running `/setup-ralph` first.

## Phase 1: Interview

Interview the user relentlessly about every aspect of this feature until reaching shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions **one at a time**.

If a question can be answered by exploring the codebase, explore the codebase instead of asking.

Cover at minimum:

- What problem does this solve? Who is the user?
- What is the happy path? What are the edge cases?
- What exists today that this builds on or replaces?
- What are the constraints (tech stack, performance, compatibility)?
- What is explicitly out of scope?

Keep going until there are no unresolved branches. Then say: **"I think we have shared understanding. Ready to generate the plan?"**

## Phase 2: Plan generation

Once the user confirms, generate the implementation plan. Do NOT create GitHub issues, push to remote, or interact with any external services.

### 1. Explore the codebase

If you have not already explored the codebase during the interview, do so now to understand the current architecture, existing patterns, and integration layers.

### 2. Identify durable architectural decisions

Before slicing, identify high-level decisions that are unlikely to change throughout implementation:

- Route structures / URL patterns
- Database schema shape
- Key data models
- Authentication / authorization approach
- Third-party service boundaries

These go in the plan header so every phase can reference them.

### 3. Draft vertical slices

Break the feature into **tracer bullet** phases. Each phase is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Do NOT include specific file names, function names, or implementation details that are likely to change as later phases are built
- DO include durable decisions: route paths, schema shapes, data model names
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each phase show:

- **Title**: short descriptive name
- **User stories covered**: which user stories emerged from the interview

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Should any phases be merged or split further?

Iterate until the user approves the breakdown.

### 5. Ask the user for priority

Ask the user to set the priority. Default is **3** (new feature). Options:

- 1 = Critical bugfix
- 2 = Development infrastructure
- 3 = New feature (default)
- 4 = Polish / quick win
- 5 = Refactor

### 6. Write the plan file

Save the plan to `ralph/backlog/` as a markdown file named `<feature-name>-plan.md`.

**Frontmatter:**

```yaml
---
title: "<Feature Name> — Plan"
type: plan
status: open
priority: <chosen priority, default 3>
created: <today's date YYYY-MM-DD>
---
```

**Body** — use the template below:

<plan-template>
# Plan: <Feature Name>

> Source: grill-to-plan conversation

## Architectural decisions

Durable decisions that apply across all phases:

- **Routes**: ...
- **Schema**: ...
- **Key models**: ...
- (add/remove sections as appropriate)

---

## Phase 1: <Title>

**User stories**: <list from interview>

### What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation.

### Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

---

## Phase 2: <Title>

**User stories**: <list from interview>

### What to build

...

### Acceptance criteria

- [ ] ...

<!-- Repeat for each phase -->
</plan-template>
