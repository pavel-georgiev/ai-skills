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

Create a GitLab MR for the above changes. All git and glab commands MUST use `dangerouslyDisableSandbox: true` since they require network and SSH access.

1. If the current branch is main or master, inform the user and stop
2. Push the branch: `git push -u origin HEAD`
3. Check if MR already exists: `glab mr list --source-branch <branch>`
4. Detect an optional Jira ticket ID from the branch name:
   - Extract first Jira-style token: `echo "$BRANCH" | grep -oE '[A-Z][A-Z0-9]+-[0-9]+' | head -n1`
   - If found, prepend MR title: `JIRA_ID: <title>`
5. Analyze the commits and diff from the Context section above to draft an MR title and description
6. Create or update the MR:
   - If no MR exists: `glab mr create --target-branch main --remove-source-branch --title "..." --description "..."`
   - If MR exists: `glab mr update <MR_NUMBER> --title "..." --description "..."`
   - Description must use a HEREDOC and include this footer:

```
---

# AI Tool Assistance Usage Statement
Please select applicable statement out of the following, regarding AI assistance usage.
- [x] AI assistance was used to draft parts of the implementation, that was subsequently modified and extended.
- [x] AI assistance was used in generating tests/documentation/comments for this change.
- [x] AI assistance was used for optimizing/troubleshooting/refactoring existing code in this change.
```

7. Enable auto-merge using the GitLab GraphQL API in a single command:

```bash
PROJECT_PATH=$(git config --get remote.origin.url | sed 's|.*://[^/]*/||;s|.*:||;s|\.git$||')
glab api graphql -f query="
mutation {
  mergeRequestAccept(input: {
    projectPath: \"${PROJECT_PATH}\",
    iid: \"${MR_IID}\",
    sha: \"${SHA}\",
    strategy: MERGE_WHEN_CHECKS_PASS,
    shouldRemoveSourceBranch: true
  }) {
    mergeRequest { state autoMergeEnabled }
    errors
  }
}"
```

   - Do NOT use `glab mr merge` — it attempts an immediate merge and returns 405 if the pipeline hasn't passed yet
   - If the API call fails, inform the user but do not treat it as a fatal error

8. Return the MR URL to the user

## Optimization notes

- Steps 2-3 can be run as parallel Bash calls (push + MR list check are independent)
- The Context section above already provides git status, branch, commits, diff, and SHA — do NOT re-run these commands
- Combine project path extraction + GraphQL into a single Bash call (step 7)
