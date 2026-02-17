---
name: spec
description: Build a spec from a prompt or file, save to ./specs/
---


# Spec Builder

## Step 1: Process Input

Check if `$ARGUMENTS` is a file path by running `test -f "$ARGUMENTS"`:
- If it's a file: Read its contents using the Read tool as the task description
- If it's text: Use `$ARGUMENTS` directly as the task description

## Step 2: Explore Codebase

Before asking questions, explore the codebase to understand:
- How similar features are currently implemented
- Existing patterns, abstractions, and conventions
- Relevant files and modules that will be affected
- Current architecture and data flow

Use Glob, Grep, and Read tools to investigate. Document your findings.

## Step 3: Interview (Required)

Interview the user before writing the spec.

- Use a structured question tool when available:
  - Codex Plan mode: `request_user_input`
  - Claude Code: `AskUserQuestion`
- In regular/default mode (or if no question tool is available), ask questions directly in chat and wait for answers.
- Do not skip the interview just because a specific tool is unavailable.
- For new users or first-pass requests, ask at least 2 questions unless requirements are already explicit and complete.

Focus on unresolved items from codebase exploration:
- **Architecture**: How should this integrate with existing systems?
- **Mechanism**: What's the core mechanism/approach for this change?
- **Scope**: What's in scope vs out of scope?
- **Constraints**: Performance, security, compatibility requirements?

Only ask questions you couldn't answer from codebase exploration. Be concise and specific.
If you proceed with assumptions, list them explicitly in the spec.

## Step 4: Write Spec

1. Create the `./specs/` directory if it doesn't exist: `mkdir -p ./specs`
2. Generate a short kebab-case name summarizing the spec

The spec MUST include these sections:

### Required Sections

- **Overview**: 2-3 sentence summary
- **Goals**: Numbered list of objectives
- **Non-Goals**: What's explicitly out of scope
- **Architecture**: How this fits into the system, data flow, component interactions
- **Implementation Tasks**: Ordered checklist of implementation steps (see format below)
- **Files to Modify**: List of files that need changes
- **Testing**: How to verify the implementation

### Task List Format

Each task has a title and description separated by ` - `. The title is used as the matching key for task tracking.

```markdown
## Implementation Tasks

- [ ] Add authentication middleware - Create Express middleware that validates JWT tokens and extracts user context
- [ ] Create login endpoint - POST /api/auth/login accepting email/password, returning JWT token
- [ ] Add session storage - Store sessions in Redis with 24h TTL, handle refresh tokens
```

3. Write the completed spec to `./specs/{name}.md`

## Step 5: Next Steps

After writing the spec:
1. Output the spec path and task summary (count and titles)
2. Ask: "Would you like me to start implementing? I'll update the spec checkboxes as I progress."

If yes, hand off to the implementation flow by working through tasks in order, updating checkboxes in the spec file as tasks are completed.
