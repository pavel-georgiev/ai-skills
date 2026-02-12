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
7. Create or update the MR using glab with a HEREDOC body that includes:
   - Summary section with bullet points describing the changes
   - Test plan section
   - AI Tool Assistance Usage Statement (always include all checkboxes selected)

Example glab create command:
```
glab mr create --target-branch main --title "Title here" --description "$(cat <<'EOF'
## Summary
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
glab mr update <MR_NUMBER> --title "New title" --description "$(cat <<'EOF'
...description...
EOF
)"
```

8. Return the MR URL to the user
