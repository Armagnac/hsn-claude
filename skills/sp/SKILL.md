---
name: sp
description: "Shortcut alias for /superplan. Produce higher-quality code by breaking a feature into small, focused tasks the coding agent can nail one at a time. Works like an engineering team: feature → milestones → ~30-min tasks with specific files, acceptance criteria, and dependencies. Each task runs in a fresh context — narrow scope, full attention, one git commit per task."
when_to_use: "create a plan, plan this feature, break this down, I want higher quality code, /sp, sp"
allowed-tools: Bash(git:* ls:* cat:*) Read Glob Grep Edit Write Agent Task
argument-hint: <description or ticket_id> | status | next <plan_file> --yes | review <plan_file>
---

This is a shortcut alias for `/superplan`. Read and follow the instructions in the `superplan` skill (installed alongside this one as `~/.claude/skills/superplan/SKILL.md`).

Execute exactly as if the user typed `/superplan` with the same arguments — same routing, same subcommands, same behavior.
