# Agent Skills

Claude Code configuration and agentic development skills: structured planning (`/superplan`), read-only Q&A (`/ask`), and autonomous plan execution (`plan-execution`).

## Installation

```bash
npx skills add Armagnac/agent-skills -g
```

That's it. The `npx skills` CLI installs all three skills into `~/.claude/skills/` globally.

**Alternative: symlink from a local clone**

```bash
git clone https://github.com/Armagnac/agent-skills ~/git/agent-skills
ln -s ~/git/agent-skills/skills ~/.claude/skills
```

---

## Skills Overview

| Skill | Purpose | When to Use |
|-------|---------|------------|
| **`/ask`** | Read-only Q&A — answer questions about code, architecture, or anything else without modifying files | Understanding existing code before making changes, researching before a task |
| **`/superplan`** | Break large features into milestones and tasks, track progress in markdown, auto-execute with `/loop` | Starting a multi-day feature, complex refactors, want autonomous execution |
| **`plan-execution`** (auto) | Auto-activates during plan work — executes tasks, marks progress, creates commits | Works alongside `/superplan` and `/loop`; requires no explicit invocation |

---

## Quick Examples

### `/ask` — Understand code before changing it

```bash
/ask How does the auth system work?
# → Claude reads code, explains architecture without modifying anything

/ask What files implement database queries?
# → Research-only; no files touched

/ask Read online about Karpathy's approach for knowledge management. How could we apply it in our repo?
```

### `/superplan` — Build a feature methodically

```bash
/superplan Add JWT authentication
# → Claude researches codebase, breaks it into milestones and tasks
# → Writes plans/jwt-auth.plan.md with files, specs, acceptance criteria

/superplan next
# → Execute task 1.1, verify, commit, mark done

/loop /superplan next --yes plans/jwt-auth.plan.md
# → Run all remaining tasks autonomously while you take a break
```


### `plan-execution` — Automatic during `/superplan` work

No invocation needed. Activates automatically when you:
- Run `/superplan next` or `/sp next`
- Say "continue the plan"
- Reference a `plans/*.plan.md` file

---

## `/ask` Command

Read-only Q&A mode — Claude answers questions without modifying any files. Useful for understanding code, architecture, or investigating before making changes.

```bash
/ask How does the auth system work?
/ask What's the difference between X and Y?
/ask Which files implement feature Z?
```

No files are modified. Use `/ask` for research and learning; use `/superplan` when you're ready to implement.

---

## `/superplan` in Detail

`/superplan` breaks large features into **milestones** (product deliverables) and **tasks** (developer work items), tracks progress in a markdown file, and executes each task with acceptance criteria.

### Subcommands

| Command | What it does |
|---------|-------------|
| `/superplan <description>` | Research the codebase and generate a structured plan file |
| `/superplan <TICKET-ID>` | Same, but fetches ticket details from Linear/GitHub/Jira first |
| `/superplan next` | Execute the next unchecked task (asks for confirmation) |
| `/superplan next --yes` | Execute the next task without confirmation |
| `/superplan next <file>` | Target a specific plan file |
| `/superplan status` | Show progress across all active plans |
| `/superplan review` | Re-evaluate the plan against current code state |

### Structured vs. Unstructured Loops

`/loop /superplan next --yes` is a **structured autonomy pattern** — each loop executes a named task with explicit acceptance criteria and makes a git commit before advancing.

**Comparison with Ralph Loop:**

| | Ralph Loop | `/loop /superplan next --yes` |
|--|--|--|
| **Structure** | Unstructured — same prompt re-fed each iteration | Structured — next `[ ]` task with files, deps, acceptance criteria |
| **Progress tracking** | None | `[x]` checkboxes + completion timestamps in plan file |
| **Completion signal** | String match (e.g., `"DONE"`) or `--max-iterations` | "All tasks complete!" when no `[ ]` remain |
| **Commits** | None | One commit per task: `Plan 2.1: <title>` |
| **Dependency ordering** | None | Won't run a task until its deps are `[x]` |
| **Resumability** | Restart from scratch if interrupted | Stop and resume anywhere — progress persists in the plan file |
| **Loop mechanism** | Stop hook (inside session) | `/loop` skill (new prompt each iteration) |

**Bottom line:** Ralph wins on zero-setup for exploratory tasks. `/superplan` + `/loop` wins on auditability, resumability, and correctness — you get a full git history and can always inspect exactly which task failed and why.

### Auto-executing with `/loop`

**Example 1: Run an entire plan unattended**

```bash
/loop /superplan next --yes plans/dark-mode.plan.md
```

What happens:
1. Finds task 1.1, reads full plan for context, implements, runs tests, commits `Plan 1.1: Define CSS variable palettes`
2. Marks `[ ]` → `[x]`, finds task 2.1
3. Implements, commits `Plan 2.1: Add theme toggle to settings panel`
4. No more tasks → prints "All tasks complete!" and the loop stops

**Example 2: Run only the first milestone, then review**

```bash
/loop /superplan next --yes plans/jwt-auth.plan.md
# Watch output — when milestone 1 tasks are all done, press Ctrl+C
# Review the code, test it, adjust the plan if needed
/loop /superplan next --yes plans/jwt-auth.plan.md   # resume milestone 2
```

Each task was committed separately, so `git log` shows clean progress and any milestone is easy to roll back.

**Example 3: Resume a plan that's already in-progress**

```bash
/superplan status                        # see which tasks are done and what's next
/loop /superplan next --yes              # auto-detects the in-progress plan, resumes from first [ ]
```

Claude re-reads the full plan on each iteration — completed tasks give context about what was already built, future tasks show what constraints to keep in mind.

### Safety

- Review the plan file before looping — it's plain markdown, easy to edit
- `--yes` skips confirmation prompts; omit it to approve each task manually
- Each task commits separately — easy to `git revert` a single bad task without losing everything
- Cancel anytime with `Ctrl+C`; the plan file preserves progress so the next `/superplan next` picks up where you left off

---

## Plan File Format

Plans live in `plans/*.plan.md` in your **project repo** (not this config repo). The `/superplan` commands creates and manages them.

### Anatomy of a Plan File

```markdown
# Plan: <Title>

| Field   | Value        |
|---------|--------------|
| Status  | draft        |   ← draft | in-progress | complete
| Created | 2026-04-10   |
| Ticket  | ENG-123      |   ← or N/A
| Branch  | feature/foo  |   ← or TBD

## Context
Why this work exists — the problem, the constraint, the goal.

## Architecture Decisions
Key choices made upfront so implementers don't re-litigate them mid-task.

## Diagrams (optional)
Mermaid flowcharts or sequence diagrams.

## Milestones Overview
Numbered list of major phases.

---

## Milestone 1: <Name>

### 1.1 [ ] Task title

- **Files:** path/to/file.py, path/to/other.py
- **What:** Step-by-step instructions for the implementer.
- **Acceptance:** How to verify this task is done (command to run, behavior to observe).
- **Dependencies:** None | 1.1, 2.3

### 1.2 [x] Completed task title *(completed 2026-04-11)*

...

## Verification

End-to-end checklist — commands to run, behaviors to confirm — after all milestones are done.
```

### Task Checkboxes

| Symbol | Meaning |
|--------|---------|
| `[ ]` | Not started |
| `[x]` | Complete (add `*(completed YYYY-MM-DD)*` inline) |

### Example: `/superplan status` Output

```
Plan: Add JWT Authentication System
Status: in-progress | Created: 2026-04-10

Progress: 5/14 tasks complete (36%)

Milestones
✓ 1. Data Layer (3/3) — COMPLETE
  ✓ 1.1 Add User model with hashed password field
  ✓ 1.2 Add password hashing utilities
  ✓ 1.3 Add JWT creation and validation utilities

✓ 2. Auth Endpoints (2/3) — IN PROGRESS
  ✓ 2.1 Implement POST /api/auth/register
  ✓ 2.2 Implement POST /api/auth/login
  ◯ 2.3 Implement POST /api/auth/refresh and logout

◯ 3. Route Protection (0/2)
◯ 4. Frontend Integration (0/2)

Next task: 2.3 Implement POST /api/auth/refresh and logout
Files: src/routers/auth.py
Dependencies: 2.2
```

### Why This Format Is Powerful

1. **Resumability** — Stop mid-task, come back weeks later, pick up exactly where you left off.
2. **Parallelism** — Dependencies are explicit. Tasks with no deps can run concurrently across people or agents.
3. **Auditability** — Every architectural decision is documented with the reasoning. Future maintainers understand *why*, not just *what*.
4. **Incremental progress** — A 50-task feature becomes 50 small, testable units. Each one verifiable before the next starts.

### Example Plans

See `plans/examples/` in this repo for annotated examples:

| Example | Complexity | What it shows |
|---------|-----------|---------------|
| [`add-feature.plan.md`](./plans/examples/add-feature.plan.md) | 2 milestones, 2 tasks | Minimal structure for a simple UI feature |
| [`api-refactor.plan.md`](./plans/examples/api-refactor.plan.md) | 3 milestones, 5 tasks | Dependency tracking, flowchart diagram, incremental migration |
| [`auth-system.plan.md`](./plans/examples/auth-system.plan.md) | 4 milestones, 9 tasks | Full-stack feature with parallel milestone tracks |

---

## License

MIT
