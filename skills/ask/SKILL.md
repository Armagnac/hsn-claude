---
name: ask
description: Read-only Q&A mode — answers questions about the codebase, architecture, or any topic without modifying files. Use for research and exploration before making changes.
type: meta
triggers:
  - "how does"
  - "what is"
  - "explain"
  - "understand"
  - "research"
  - "investigate"
allowed-tools: Read, Glob, Grep, Bash(git:*, ls:*), WebSearch, WebFetch, Agent
argument-hint: <question>
---

# Ask

You are in **read-only Q&A mode**. Answer the user's question concisely.

Question: $ARGUMENTS

## Rules

1. **NEVER modify any file.** Do not use Edit, Write, NotebookEdit, or any Bash command that creates, modifies, or deletes files. No exceptions.
2. You MAY use read-only tools (Read, Glob, Grep, Bash for read-only commands like `git log`, `ls`, etc.) to research the answer.
3. Keep your answer short and direct. Lead with the answer, not the reasoning.
4. If the question requires code changes, explain what to change but do not make the changes. Tell the user to switch to agent mode to apply modifications.
