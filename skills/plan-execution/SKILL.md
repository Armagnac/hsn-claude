---
name: plan-execution
description: Auto-activates when working with implementation plans. Triggers on "continue the plan", "next task", "what's the plan status", "run task 2.1", or when user references plans/*.plan.md files. Not for creating plans - use /plan command for that.
type: meta
triggers:
  - "continue the plan"
  - "next task"
  - "plan status"
  - "run task"
  - "mark task"
  - "plans/*.plan.md"
allowed-tools: Bash(git:*, ls:*), Read, Glob, Grep, Edit, Write, Agent, Task
argument-hint: (auto-activating; no explicit arguments)
---

# Plan Execution Skill

This skill auto-activates when the user references an existing plan or wants to continue working from one.

**Trigger phrases:**
- "continue the plan"
- "next task"
- "run task 2.1"
- "what's the plan status"
- "mark task X done"
- References to `plans/*.plan.md` files

**For creating new plans:** Use `/plan <description>` command instead.

---

## Core Behaviors

### When User Says "next task" or "continue the plan"

Execute the `/plan next` flow:

1. Find the active plan (`plans/*.plan.md` with status `in-progress`)
2. Parse for the first unchecked `[ ]` task whose dependencies are all `[x]`
3. Show task details and ask for confirmation
4. Implement, verify, mark complete, commit

### When User Says "run task X.Y"

1. Find the active plan
2. Locate the specific task by ID (e.g., `2.1`)
3. Check dependencies are met
4. If dependencies unmet, warn: "Task 2.1 depends on 1.3 which is incomplete. Proceed anyway? (y/n)"
5. Implement, verify, mark complete, commit

### When User Says "what's the plan status"

Execute the `/plan status` flow — show progress table for all plans.

### When User Says "mark task X done"

1. Find the task in the plan file
2. Update `[ ]` to `[x]` with completion date
3. Show updated progress

---

## Plan File Parsing

### Reading a Plan

Plans follow this structure:
- `# Plan: <title>` — main heading
- `| Status | <value> |` — in metadata table
- `## Milestone N: <name>` — milestone headings
- `### N.M [ ] <title>` — pending task
- `### N.M [x] <title> *(completed YYYY-MM-DD)*` — completed task

### Finding Dependencies

Each task has a `**Dependencies:**` field:
- `None` — can run immediately
- `1.1` — depends on task 1.1
- `1.1, 1.2` — depends on both tasks 1.1 and 1.2
- `Milestone 1` — depends on all tasks in milestone 1

A dependency is met when the referenced task(s) have `[x]`.

### Updating Task Status

When marking a task complete, replace:
```
### 2.1 [ ] Add API endpoint
```
with:
```
### 2.1 [x] Add API endpoint *(completed 2026-03-21)*
```

### Updating Plan Status

- When first task starts: change `| Status | draft |` to `| Status | in-progress |`
- When all tasks complete: change to `| Status | complete |`

---

## Task Execution Flow

When implementing a task from the plan:

### 1. Read Full Context

- Read the **entire plan file** (not just the current task)
- Understand what was built before (completed tasks)
- Understand what comes next (future tasks)
- Check architecture decisions section for relevant context

### 2. Implement

- Follow the task's **What** specification
- Modify only the **Files** listed (unless the task clearly requires additional files)
- Follow existing codebase patterns and style guides

### 3. Verify

- Check against the task's **Acceptance** criteria
- Run relevant tests
- For UI changes: use browser automation, take screenshots
- For backend changes: make API calls, check database

### 4. Run Code Quality Checks

- Run whatever code quality tooling the project provides (linting, type checking, tests)
- If the project has a dedicated code-checks command or script, use it

### 5. Update Plan

- Mark task `[x]` with completion date
- Update plan status if needed
- Add notes to Architecture Decisions if you learned something important

### 6. Commit

Create a commit with message format:
```
Plan <N.M>: <Task Title>
```

Include both the code changes AND the plan file update in the same commit.

### 7. Report Progress

Show:
- What was completed
- Current progress (X/Y tasks, percentage)
- What's next (next executable task)

---

## Multi-Agent Execution

For tasks that have independent subtasks (e.g., frontend + backend work), use parallel agents:

```
Task 2.1: Add user settings page
+-- Agent 1 (backend-specialist): Create API endpoint
+-- Agent 2 (frontend-specialist): Create React component
+-- Merge results, verify integration
```

Only parallelize when the task spec explicitly mentions independent frontend/backend work.

---

## Handling Plan Changes

If during implementation you discover the plan needs changes:

1. **Minor adjustment** (file paths wrong, small scope change): Update the plan file directly and note it
2. **Significant change** (new task needed, task should be removed, reordering): Flag to user before changing
3. **Architecture change**: Update the Architecture Decisions section and discuss with user

Never silently deviate from the plan. If you need to do something different from what's specified, say so.

---

## Integration Points

The plan system works with whatever tools and commands the project provides:

| What | How |
|------|-----|
| Creating plans | `/plan <description>` command |
| Code quality | Run project-specific checks after each task |
| PR creation | Use project's PR workflow when milestone or full plan is complete |
| Self-review | Review changes before marking milestone complete |
| Ticket tracking | Plan metadata links to issue tracker ticket if applicable |
