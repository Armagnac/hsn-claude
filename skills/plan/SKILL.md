---
name: plan
description: Create and manage incremental implementation plans for large features. Subcommands: create (default), status, next, review.
type: meta
triggers:
  - "create a plan"
  - "plan this"
  - "break down"
  - "planning"
  - "structured approach"
allowed-tools: Bash(git:*, ls:*, cat:*), Read, Glob, Grep, Edit, Write, Agent, Task
argument-hint: <description or ticket_id> | status | next [plan_file] | review [plan_file]
---

# Plan Command

When user runs `/plan`, route to the appropriate subcommand based on the argument.

## Routing

| Argument | Action |
|----------|--------|
| `status` | Show progress across all active plans |
| `next` or `next <plan_file>` | Execute next unchecked task |
| `next --yes` or `next <plan_file> --yes` | Execute next unchecked task without confirmation prompt |
| `review` or `review <plan_file>` | Re-evaluate plan against current code state |
| Anything else | Create a new plan (treat argument as feature description or ticket ID) |

---

## Subcommand: Create (Default)

**Trigger:** `/plan <feature description>` or `/plan <TICKET-ID>`

### Steps

#### 1. Gather Context

- If argument looks like a ticket ID or issue tracker URL:
  - Fetch ticket details using whatever issue tracker tools are available (Linear, GitHub Issues, Jira, etc.)
  - Extract title, description, attachments, acceptance criteria
- If argument is a free-text description:
  - Use it as the feature description directly

#### 2. Research the Codebase

Use **parallel Explore agents** to understand what exists:

- **Agent 1:** Search for related code, patterns, and existing implementations
- **Agent 2:** Identify files that will likely need changes
- **Agent 3:** Check for database schema implications, API routes, shared types

Synthesize findings into a clear understanding of what needs to change.

#### 3. Design the Plan

Break the feature into **milestones** (product deliverables) and **tasks** (developer work items):

**Milestones** should be:
- Independently valuable (each delivers something usable)
- Ordered from most foundational to most advanced
- Testable end-to-end after completion

**Tasks** should be:
- Small enough to complete in one conversation (~30 min of Claude work)
- Written as **instructions for the developer** — describe what needs to happen, expected behavior, and constraints (not step-by-step implementation code)
- Fully working and testable after completion
- Have clear acceptance criteria
- List specific files to create/modify
- Declare dependencies on other tasks

**Diagrams:** When the feature involves architecture, data flow, or multi-component interactions, include **mermaid diagrams** in the plan (e.g., flowcharts for data flow, sequence diagrams for API interactions, component diagrams for architecture). Add these in a `## Diagrams` section or inline within the relevant milestone.

#### 4. Generate Plan File

- Derive a kebab-case name from the feature (e.g., `user-api-keys`, `slack-integration`)
- Create the file at `plans/<name>.plan.md`

**Use this exact template:**

```markdown
# Plan: <Feature Title>

| Field | Value |
|-------|-------|
| Status | draft |
| Created | <YYYY-MM-DD> |
| Ticket | <ticket ID or N/A> |
| Branch | <branch name or TBD> |

## Context

<2-3 sentences: what we're building, why, and key constraints>

## Architecture Decisions

<Key technical decisions made during planning. Update as implementation progresses.>

- **Decision 1:** <description> — <rationale>

## Diagrams

<Include mermaid diagrams when relevant — architecture, data flow, component interactions, sequences, etc. Omit this section if diagrams aren't helpful for the feature.>

```mermaid
<diagram here>
```

## Milestones Overview

<High-level numbered list of all milestones for quick scanning>

1. **<Milestone 1 Name>** — <one-line description of what it delivers>
2. **<Milestone 2 Name>** — <one-line description of what it delivers>

---

## Milestone 1: <Product Deliverable Name>

<1 sentence describing what's usable after this milestone>

### 1.1 [ ] <Task Title>
- **Files:** `path/to/file1`, `path/to/file2`
- **What:** <Instructions for the developer — describe what needs to happen, expected behavior, and any constraints>
- **Acceptance:** <How to verify it works>
- **Dependencies:** None

### 1.2 [ ] <Task Title>
- **Files:** `path/to/file`
- **What:** <Instructions for the developer — describe what needs to happen, expected behavior, and any constraints>
- **Acceptance:** <How to verify>
- **Dependencies:** 1.1

---

## Milestone 2: <Product Deliverable Name>

<1 sentence describing what's usable after this milestone>

### 2.1 [ ] <Task Title>
- **Files:** `path/to/file`
- **What:** <Instructions for the developer — describe what needs to happen, expected behavior, and any constraints>
- **Acceptance:** <How to verify>
- **Dependencies:** 1.2

...
```

#### 5. Present to User

Show the user:
- Milestones overview (the high-level list)
- Total task count
- Estimated complexity per milestone
- Ask: "Review and edit `plans/<name>.plan.md`, then run `/plan next` to start executing."

**Do NOT start implementing.** The plan is a draft for user review.

---

## Subcommand: Status

**Trigger:** `/plan status`

### Steps

1. Glob for `plans/*.plan.md` files
2. For each plan file, parse:
   - Plan title (from `# Plan:` heading)
   - Status field
   - Count of `[x]` (completed) vs `[ ]` (pending) tasks
   - Current milestone (first milestone with incomplete tasks)
3. Display summary table:

```
## Active Plans

| Plan | Status | Progress | Current Milestone |
|------|--------|----------|-------------------|
| User API Keys | in-progress | 5/12 (42%) | Milestone 2: Key Management UI |
| Slack Integration | draft | 0/8 (0%) | Milestone 1: Webhook Setup |
```

If no plan files found, show: "No plans found. Create one with `/plan <feature description>`"

---

## Subcommand: Next

**Trigger:** `/plan next` or `/plan next plans/<name>.plan.md` or either with `--yes` flag

### Steps

#### 1. Find the Plan

- If plan file specified, use it
- If not specified:
  - Glob for `plans/*.plan.md`
  - If exactly one `in-progress` plan exists, use it
  - If multiple exist, ask the user which one
  - If none exist, show error: "No active plans. Create one with `/plan <feature description>`"

#### 2. Find Next Task

- Parse the plan file for the first unchecked task (`[ ]`)
- Verify its dependencies are all completed (`[x]`)
- If dependencies are not met, find the next task whose dependencies ARE met
- If no executable task found, report: "All tasks complete!" or "Blocked — remaining tasks have unmet dependencies"

#### 3. Update Plan Status

- If plan status is `draft`, update it to `in-progress`

#### 4. Display Task Context

Show the user:
- Task ID and title (e.g., "Task 2.1: Add API endpoint for key creation")
- Files to modify
- Changes description
- Acceptance criteria
- Dependencies status

If the `--yes` flag was passed, skip the confirmation prompt and proceed directly to implementation. Otherwise ask: "Ready to implement this task? (y/n)"

#### 5. Implement the Task

After user confirms (or immediately if `--yes` was passed):

1. **Read the full plan** for context (understand what was built before and what comes next)
2. **Implement the changes** described in the task spec
3. **Verify acceptance criteria** — run relevant tests, make API calls, or take screenshots
4. **Run code quality checks** if the project has them (e.g., linting, type checking, tests)
5. **Mark the task complete** — update the plan file:
   - Change `[ ]` to `[x]`
   - Add completion timestamp: `[x] <Task Title> *(completed YYYY-MM-DD)*`
6. **Commit the work** with message: `Plan <N.M>: <Task Title>`
   - Include the plan file update in the commit

#### 6. Show Progress

After completing the task, display:
```
Completed: 2.1 - Add API endpoint for key creation
Progress: 6/12 tasks (50%)
Next up: 2.2 - Add key validation middleware
```

---

## Subcommand: Review

**Trigger:** `/plan review` or `/plan review plans/<name>.plan.md`

### Steps

#### 1. Find the Plan

Same logic as `next` subcommand for finding the plan file.

#### 2. Analyze Current State

Use parallel agents to:
- **Agent 1:** Read the plan file and understand what was planned vs. completed
- **Agent 2:** Read the actual code changes (git diff from branch base) to understand what was built
- **Agent 3:** Check if remaining tasks are still accurate (files may have moved, APIs changed)

#### 3. Generate Review Report

```
## Plan Review: <Feature Name>

### Progress
- Completed: X/Y tasks
- Current milestone: <name>

### Drift Analysis
<List any discrepancies between plan and reality>
- Task 2.1 specified `users_route.py` but implementation went in `api_keys_route.py`
- New dependency discovered: need migration before task 3.1

### Recommended Changes
- [ ] Update task 2.3 files list (route was renamed)
- [ ] Add new task 2.5 for migration
- [ ] Remove task 3.2 (handled by task 2.1 already)

### Updated Remaining Tasks
<Show the remaining tasks with any suggested modifications>
```

#### 4. Apply Updates

Ask user: "Apply these plan updates? (y/n)"

If yes, update the plan file with corrections.

---

## Plan File Conventions

- **Location:** Always in `plans/` directory at project root
- **Naming:** `<kebab-case-feature>.plan.md`
- **Status values:** `draft` -> `in-progress` -> `complete`
- **Task format:** `### <N.M> [ ] <Title>` for pending, `### <N.M> [x] <Title> *(completed YYYY-MM-DD)*` for done
- **Gitignore:** Plans are committed to git (useful for team visibility). Add to `.gitignore` only if user prefers.

---

## Integration with Project Workflow

The plan system is designed to work alongside any project's existing workflow:

- `/plan` handles the planning phase for large features
- Individual tasks follow the same implementation patterns as any dev task
- Run project-specific code quality checks after each task implementation
- When all tasks complete, create a PR for the full feature (or per milestone)
- `/plan review` can be run anytime to check plan health

If the project has specific commands for code checks, PR creation, or ticket management, use those as part of task execution.

---

## Anti-Patterns

- Starting implementation before user reviews the plan
- Creating tasks that are too large (>1 hour of work)
- Creating tasks that aren't independently testable
- Skipping acceptance criteria
- Not updating the plan file after completing tasks
- Ignoring dependency ordering
- Creating a plan for a trivial task (just do it directly)

## Good Practices

- Keep tasks small and focused (~30 min each)
- Make each task independently verifiable
- Update architecture decisions as you learn things during implementation
- Commit after each task (not after each milestone)
- Review the plan periodically with `/plan review`
- Edit the plan file directly when scope changes
