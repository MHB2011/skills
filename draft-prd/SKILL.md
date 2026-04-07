---
name: draft-prd
description: Create a PRD through user interview and codebase exploration, saved as a local backlog file in ralph/backlog/. Use when user wants to write a PRD for Ralph's local backlog instead of GitHub issues, or mentions "draft prd" or "local prd".
---

This skill creates a PRD and saves it to the Ralph local backlog (`ralph/backlog/`) instead of GitHub issues. Same interview and exploration process as `/write-a-prd`, different output destination.

You may skip steps if you don't consider them necessary.

## Before starting

Check that `ralph/backlog/` exists. If it doesn't, ask the user where their Ralph backlog directory is, or suggest running `/setup-ralph` first.

## Workflow

1. Ask the user for a long, detailed description of the problem they want to solve and any potential ideas for solutions.

2. Explore the repo to verify their assertions and understand the current state of the codebase.

3. Interview the user relentlessly about every aspect of this plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

4. Sketch out the major modules you will need to build or modify to complete the implementation. Actively look for opportunities to extract deep modules that can be tested in isolation.

A deep module (as opposed to a shallow module) is one which encapsulates a lot of functionality in a simple, testable interface which rarely changes.

Check with the user that these modules match their expectations. Check with the user which modules they want tests written for.

5. Ask the user if they want to adjust the priority. Default is **3** (new feature). Options:
   - 1 = Critical bugfix
   - 2 = Development infrastructure
   - 3 = New feature (default for PRDs)
   - 4 = Polish / quick win
   - 5 = Refactor

6. Once you have a complete understanding of the problem and solution, write the PRD using the template below and save it as a markdown file in `ralph/backlog/`.

**Filename:** kebab-case of the title (e.g., `instructor-analytics-dashboard.md`).

**Frontmatter:**

```yaml
---
title: "<PRD title>"
type: prd
status: open
priority: <chosen priority, default 3>
created: <today's date YYYY-MM-DD>
---
```

**Body** — use the PRD template below:

<prd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this PRD.

## Further Notes

Any further notes about the feature.

</prd-template>
