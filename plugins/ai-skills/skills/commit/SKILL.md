---
name: commit
description: Review current git changes and create a single commit
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*), Bash(git diff:*), Bash(git branch:*), Bash(git log:*)
---

## Context

- Current git status: !`git status`
- Current git diff (staged and unstaged changes): !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Your task

Based on the above changes, create a single git commit.

Use the current diff as the source of truth for the commit message and stage only the files that belong in this commit.

**Critical permission rule**: Each git command MUST be its own separate Bash tool call so that permission patterns like `Bash(git add:*)` and `Bash(git commit:*)` match correctly. NEVER bundle multiple commands into a single Bash call using `&&`, `;`, variable assignments before the command, or shell scripts.

1. Review the current changes from the Context section above
2. Stage the files that belong in a single coherent commit
3. Create one commit with an appropriate message
4. Do not push, open a PR, or use any non-git tools
5. Do not send any other text or messages besides these tool calls
