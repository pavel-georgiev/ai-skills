---
name: implement-spec
description: Implement a spec by working through tasks sequentially
---


# Spec Implementer

## Step 1: Load Spec and Tasks

1. Read the spec file at `$ARGUMENTS` using the Read tool
2. If no argument provided, use Glob to find specs: `./specs/*.md` and ask user to select one
3. Call TaskList to get current tasks
4. If no tasks exist for this spec, offer to create them:
   - "No tasks found. Would you like me to create tasks from the spec?"
   - If yes, parse implementation tasks and create with TaskCreate

## Step 2: Find Next Task

1. From TaskList, find tasks that are:
   - Status: `pending`
   - Not blocked (blockedBy list is empty or all blockers are completed)
2. If no pending unblocked tasks:
   - If all tasks completed: "All tasks complete! Spec implementation finished."
   - If tasks are blocked: Show which tasks are blocked and by what
3. Select the first available task

## Step 3: Start Task

1. Output: "Starting: {task subject}"
2. Call TaskUpdate to set status to `in_progress`
3. Call TaskGet to retrieve full task description

## Step 4: Implement

1. Implement the task according to its description
2. Follow the spec's Architecture and Testing sections for guidance
3. If blocked or unclear:
   - Use AskUserQuestion to get clarification
   - Do NOT mark task as completed if implementation is partial

## Step 5: Complete Task

After successful implementation:

1. Call TaskUpdate to set status to `completed`
2. Update the spec file: change `- [ ] {task subject}` to `- [x] {task subject}`
3. Output: "Completed: {task subject}"

## Step 6: Continue or Stop

Ask: "Continue to next task?"

Options:
- **Yes**: Go back to Step 2
- **No**: Output progress summary and exit
- **Skip to specific task**: Let user name a task to work on next (will mark it in_progress regardless of blockers, with warning)

## Progress Summary

When stopping or completing all tasks, output:
- Tasks completed this session
- Tasks remaining
- Any blocked tasks and their blockers
- Link to spec file
