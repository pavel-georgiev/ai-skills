---
name: commit-and-mr
description: Commit staged changes, push branch, and create GitLab merge request
---


Commit and create a GitLab MR workflow:

**Important**: If any `git` or `glab` commands fail due to sandbox restrictions (e.g., SSH permission denied, network errors), retry the command with `dangerouslyDisableSandbox: true`.

1. Run `git status` to check the repo state
2. If there are no staged or unstaged changes, inform the user and stop
3. If there are unstaged or untracked files, you MUST use the AskUserQuestion tool to ask the user what to do. Present these options:
   - "Stage all" - run `git add -A` to stage everything
   - "Stage specific files" - then ask which files and run `git add <files>`
   - "Ignore" - proceed with only currently staged changes
   - "Abort" - stop the workflow entirely
4. If there are staged changes:
   - Run `git diff --staged` to understand the changes
   - Run `git log --oneline -5` to see recent commit style
   - If the current branch is main or master, create a new branch with a descriptive name
   - Create a commit with a descriptive message following conventional commit style
5. Push the branch: `git push -u origin HEAD`
   - If push fails due to SSH/permission issues, retry with sandbox disabled
6. Check if MR already exists: `glab mr list --source-branch $(git branch --show-current)`
7. Detect an optional Jira ticket ID from the branch name:
   - Read branch name: `BRANCH_NAME=$(git branch --show-current)`
   - Extract first Jira-style token: `JIRA_ID=$(echo "$BRANCH_NAME" | grep -oE '[A-Z][A-Z0-9]+-[0-9]+' | head -n1)`
   - If `JIRA_ID` is present, prepend the MR description with `JIRA_ID: <MR title>`
   - If no Jira token is found, keep the current MR description format unchanged
8. Create or update the MR using glab with a HEREDOC body that includes:
   - Optional Jira prefix line in the format `JIRA_ID: <MR title>` when Jira is detected
   - Summary section with bullet points describing the changes
   - Test plan section
   - AI Tool Assistance Usage Statement (always include all checkboxes selected)

Example glab create command:
```
BRANCH_NAME="$(git branch --show-current)"
JIRA_ID="$(echo "$BRANCH_NAME" | grep -oE '[A-Z][A-Z0-9]+-[0-9]+' | head -n1)"
MR_TITLE="Title here"
JIRA_PREFIX=""
if [ -n "$JIRA_ID" ]; then
  JIRA_PREFIX="${JIRA_ID}: ${MR_TITLE}

"
fi

glab mr create --target-branch main --title "$MR_TITLE" --description "$(cat <<EOF
${JIRA_PREFIX}## Summary
- Change 1
- Change 2

## Test plan
- [ ] Test item

---

# AI Tool Assistance Usage Statement
Please select applicable statement out of the following, regarding AI assistance usage.
- [x] AI assistance was used to draft parts of the implementation, that was subsequently modified and extended.
- [x] AI assistance was used in generating tests/documentation/comments for this change.
- [x] AI assistance was used for optimizing/troubleshooting/refactoring existing code in this change.
EOF
)"
```

Example glab update command (if MR exists):
```
BRANCH_NAME="$(git branch --show-current)"
JIRA_ID="$(echo "$BRANCH_NAME" | grep -oE '[A-Z][A-Z0-9]+-[0-9]+' | head -n1)"
MR_TITLE="New title"
JIRA_PREFIX=""
if [ -n "$JIRA_ID" ]; then
  JIRA_PREFIX="${JIRA_ID}: ${MR_TITLE}

"
fi

glab mr update <MR_NUMBER> --title "$MR_TITLE" --description "$(cat <<EOF
${JIRA_PREFIX}## Summary
...description...
EOF
)"
```

9. Return the MR URL to the user
