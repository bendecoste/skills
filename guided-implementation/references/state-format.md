# Guided Implementation State File Format

The state file lives at `<project-root>/.claude/guided-impl-state.local.md`. The `.local` suffix ensures it is gitignored by Claude Code convention.

## Schema

Write the state file as markdown with this structure:

```markdown
---
goal: "<one-line description of what is being built>"
created: "<ISO 8601 timestamp>"
status: "in-progress" | "completed"
current_step: <number>
total_steps: <number>
---

# Implementation Plan: <goal>

## Step 1: <title>
- **Status**: pending | in-progress | completed
- **Files**: <comma-separated list of file paths to create or modify>
- **Depends on**: <step numbers this step depends on, or "none">
- **Notes**: <any observations from the collaborative check, or empty>

## Step 2: <title>
...
```

## Rules

- Update `current_step` whenever advancing to a new step.
- Set a step's status to `in-progress` when presenting its context to the user.
- Set a step's status to `completed` only when the user is satisfied with their implementation.
- Append observations from the collaborative check to the `Notes` field — keep them brief.
- When resuming a session, read this file first and offer to continue from the last `in-progress` or first `pending` step.
