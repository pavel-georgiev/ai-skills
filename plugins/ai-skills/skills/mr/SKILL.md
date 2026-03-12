---
name: mr
description: Push branch, create GitLab merge request with auto-merge and delete source branch
allowed-tools: Bash(git status:*), Bash(git push:*), Bash(git log:*), Bash(git diff:*), Bash(git rev-parse:*), Bash(git branch:*), Bash(git config:*), Bash(glab mr:*), Bash(glab api:*), Bash(glab repo:*)
---

## Context

- Current git status: !`git status`
- Current branch: !`git branch --show-current`
- Recent commits from main: !`git log --oneline main..HEAD`
- Changed files summary: !`git diff main...HEAD --stat`
- HEAD SHA: !`git rev-parse HEAD`

## Your task

Create a GitLab MR for the above changes.

**Critical permission rule**: Each `git` and `glab` command MUST be its own separate Bash tool call so that permission patterns like `Bash(git push:*)` and `Bash(glab mr create:*)` match correctly. NEVER bundle multiple commands into a single Bash call using `&&`, `;`, variable assignments before the command, or shell scripts. If you need values from a previous command, capture them from the tool output and interpolate them into the next separate Bash call.

**Sandbox rule**: Only use `dangerouslyDisableSandbox: true` for `git push` (requires SSH access). All other commands should run inside the sandbox.

1. If the current branch is main or master, inform the user and stop
2. Push the branch with `dangerouslyDisableSandbox: true`: `git push -u origin HEAD`
   - If diverged (e.g. after squash/rebase), use `git push -u origin HEAD --force-with-lease`
3. Check if MR already exists: `glab mr list --source-branch <branch>`
4. Extract a Jira ticket ID from the branch name (match pattern `[A-Z][A-Z0-9]+-[0-9]+`). If found, prepend MR title: `JIRA_ID: <title>`
5. Analyze the commits and diff from the Context section above to draft an MR title and description
6. Create or update the MR — the command MUST start with `glab` (no variable assignments before it):
   - If no MR exists: use `glab mr create --target-branch main --remove-source-branch --title "<JIRA_ID: >Title" --description "<multi-line description>"`
   - If MR exists: use `glab mr update <MR_NUMBER> --title "<JIRA_ID: >Title" --description "<multi-line description>"`
   - Inline the Jira ID directly in the title string — do NOT use shell variables
   - Pass the description as a single quoted argument on the `glab` command itself, with embedded newlines inside the quoted string
   - Do NOT use `$(...)`, HEREDOCs, pipes, temp files, or helper commands for the description; they break permission-prefix matching and can trigger avoidable approval prompts
   - If the description needs a literal double quote, escape it as `\"`
   - Description must include this footer:

```
---

# AI Tool Assistance Usage Statement
Please select applicable statement out of the following, regarding AI assistance usage.
- [x] AI assistance was used to draft parts of the implementation, that was subsequently modified and extended.
- [x] AI assistance was used in generating tests/documentation/comments for this change.
- [x] AI assistance was used for optimizing/troubleshooting/refactoring existing code in this change.
```

7. Enable auto-merge using the GitLab GraphQL API. Run these as **separate** Bash calls:
   - Get the project path: `git remote get-url origin` — then extract the path by stripping the `.*:` prefix and `.git` suffix
   - Call the GraphQL mutation (inline all values — no shell variables):
     ```
     glab api graphql -f query="mutation { mergeRequestAccept(input: { projectPath: \"<PROJECT_PATH>\", iid: \"<MR_IID>\", sha: \"<SHA>\", strategy: MERGE_WHEN_CHECKS_PASS, shouldRemoveSourceBranch: true }) { mergeRequest { state autoMergeEnabled } errors } }"
     ```
   - Do NOT use `glab mr merge` — it attempts an immediate merge and returns 405 if the pipeline hasn't passed yet
   - If the API call fails, inform the user but do not treat it as a fatal error

8. Return the MR URL to the user

## Optimization notes

- Steps 2-3 can be run as parallel Bash calls (push + MR list check are independent)
- The Context section above already provides git status, branch, commits, diff, and SHA — do NOT re-run these commands
