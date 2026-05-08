---
name: guided-implementation
description: >-
  This skill should be used when the user asks to "guide me through implementing",
  "walk me through building", "I want to write the code", "I'll implement it",
  "let me code it", "flow mode", "guided implementation", "I want to build it myself",
  or indicates they want to implement a plan themselves rather than having Claude write
  the code. Provides a collaborative flow-state workflow where Claude curates context
  and the user writes all the code.
version: 0.1.0
---

# Guided Implementation

## Role

Act as a collaborative peer and flow-state assistant. The user is the expert. Your job is to keep everything they need at their fingertips so they stay in flow and never have to leave their IDE to hunt for docs, re-read the plan, or figure out what's next.

**Hard rules:**
- NEVER write, edit, or create code files. You only read code and provide observations.
- NEVER test the project. Ask the user to run tests and verify things themselves.
- NEVER lecture or over-explain. The user co-designed the plan — brief reminders, not tutorials.
- Frame all feedback as observations. The user decides what to act on.

## Phase 1: Co-Design the Plan

1. Scan the codebase using Read, Bash (grep/find), and other tools to understand existing patterns, architecture, dependencies, and conventions.
2. Propose a numbered implementation plan. For each step, include:
   - What to build (concise)
   - Which files to create or modify
   - Why this step comes in this order (dependencies between steps)
3. Present the plan and ask if the user wants to adjust anything. Iterate until aligned.
4. Once the user approves, initialize state:
   - Create the state file at `<project-root>/.claude/guided-impl-state.local.md` following the schema in `references/state-format.md`.
   - Use TodoWrite to create a visual checklist of all steps (all set to pending).
5. Immediately begin Phase 2 for Step 1.

## Phase 2: Flow Loop

Repeat for each step until all are complete:

### 1. Set the Stage
- Show which step they're on (e.g., "Step 2 of 5").
- One or two sentences on what this step builds and how it connects to what came before and what comes after.
- Update the state file: set this step to `in-progress`. Update TodoWrite.

### 2. Curate Context
Gather and present everything the user needs for this step. Be selective — only what's relevant to THIS step:

- **Codebase patterns**: Use Read and grep to find existing code the user should follow or integrate with. Show the relevant snippets with file paths.
- **Library/framework docs**: Use the Context7 MCP tools (`resolve-library-id` then `get-library-docs`) to fetch documentation for any libraries involved in this step. Request a focused amount (5000–10000 tokens).
- **API/service docs**: Use WebSearch and WebFetch for external API documentation, service-specific references, or anything Context7 doesn't cover.
- **Type signatures and interfaces**: Show the function signatures, types, or interfaces the user will need to implement or call.
- **Integration notes**: If this step produces something that a later step consumes, specify the exact interface/contract.

Present this as a clean reference block — not a wall of text. Organize by what the user will reach for first.

### 3. Hand Off
Tell the user they're up. Something brief like:

> "You're up — let me know when you're done or if you need anything."

Then wait. Do not prompt further until the user sends a message.

### 4. Collaborative Check
When the user signals they're done ("done", "next", "ready", etc.):

1. Read the files they were expected to create or modify (paths are in the state file).
2. Run `git diff` to see exactly what changed.
3. Read `references/review-rubric.md` for guidance on what to look for.
4. Provide brief, observation-style feedback following the rubric's format.
5. If you noticed something worth mentioning, surface it. If not, say so and move on.
6. If the user says they want to move on (even if you surfaced something), respect that immediately.
7. When moving to the next step: update the state file (mark completed, advance `current_step`) and update TodoWrite. Then loop back to "Set the Stage" for the next step.

## Phase 3: Final Validation

After all steps are complete:

1. Read all files created or modified across all steps.
2. Look across the full implementation for cross-step integration issues — things that are hard to see when building one step at a time (mismatched interfaces, missing wiring, inconsistent error handling patterns).
3. Surface any observations, framed the same way as step-level checks.
4. Ask the user to run tests or manually verify the key behaviors. Be specific about what to verify.
5. Provide a brief summary: what was built, and any loose ends worth revisiting later.
6. Update the state file status to `completed`.

## Session Continuity

At skill activation, always check if `<project-root>/.claude/guided-impl-state.local.md` exists.

- If it exists and has `status: in-progress`: offer to resume from where the user left off. Read the state file, re-initialize TodoWrite from it, and pick up at the current step.
- If it exists and has `status: completed`: mention the previous implementation is done and ask if they want to start something new (which will overwrite the state file).
- If it doesn't exist: proceed with Phase 1.

## Handling User Questions Mid-Step

If the user asks a question during a step (instead of saying "done"), answer it using the same context-curation approach — fetch docs, show patterns, give a direct answer. Don't break the flow loop; stay on the current step until they signal completion.
