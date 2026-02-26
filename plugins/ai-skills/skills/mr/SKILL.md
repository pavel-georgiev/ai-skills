---
name: mr
description: Push branch, create GitLab merge request with auto-merge and delete source branch
---


Create a GitLab MR workflow. All git and glab commands MUST use `dangerouslyDisableSandbox: true` since they require network and SSH access.

**Important**: If any `git` or `glab` commands fail due to sandbox restrictions (e.g., SSH permission denied, network errors), retry the command with `dangerouslyDisableSandbox: true`.

1. Run `git status` to check the repo state
2. If the current branch is main or master, inform the user that they need to be on a feature branch and stop
3. Push the branch: `git push -u origin HEAD`
   - If push fails due to SSH/permission issues, retry with sandbox disabled
4. Check if MR already exists: `glab mr list --source-branch $(git branch --show-current)`
5. Detect an optional Jira ticket ID from the branch name:
   - Read branch name: `BRANCH_NAME=$(git branch --show-current)`
   - Extract first Jira-style token: `JIRA_ID=$(echo "$BRANCH_NAME" | grep -oE '[A-Z][A-Z0-9]+-[0-9]+' | head -n1)`
   - If `JIRA_ID` is present, prepend the MR title with `JIRA_ID: <MR title>`
   - If no Jira token is found, keep the current MR title format unchanged
6. Analyze the changes for the MR description:
   - Run `git log --oneline main..HEAD` to see what commits will be included
   - Run `git diff main...HEAD --stat` for a summary of changed files
7. Create or update the MR:
   - If no MR exists, create one using glab with a HEREDOC body
   - If an MR exists, update it
   - Always include `--remove-source-branch` on create

Example glab create command:
```
BRANCH_NAME="$(git branch --show-current)"
JIRA_ID="$(echo "$BRANCH_NAME" | grep -oE '[A-Z][A-Z0-9]+-[0-9]+' | head -n1)"
MR_BASE_TITLE="Title here"
MR_TITLE="$MR_BASE_TITLE"
if [ -n "$JIRA_ID" ]; then
  MR_TITLE="${JIRA_ID}: ${MR_BASE_TITLE}"
fi

glab mr create --target-branch main --remove-source-branch --title "$MR_TITLE" --description "$(cat <<EOF
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
BRANCH_NAME="$(git branch --show-current)"
JIRA_ID="$(echo "$BRANCH_NAME" | grep -oE '[A-Z][A-Z0-9]+-[0-9]+' | head -n1)"
MR_BASE_TITLE="New title"
MR_TITLE="$MR_BASE_TITLE"
if [ -n "$JIRA_ID" ]; then
  MR_TITLE="${JIRA_ID}: ${MR_BASE_TITLE}"
fi

glab mr update <MR_NUMBER> --title "$MR_TITLE" --description "$(cat <<EOF
## Summary
...description...
EOF
)"
```

8. Enable auto-merge on the MR using the GitLab API:
   - Extract the MR IID (number) from the create/list output
   - Call: `glab api --method PUT "projects/:fullpath/merge_requests/<MR_IID>" -f merge_when_pipeline_succeeds=true -f should_remove_source_branch=true`
   - Do NOT use `glab mr merge` — it attempts an immediate merge and returns 405 if the pipeline hasn't passed yet
   - If the API call fails (e.g., no pipeline configured, auto-merge not enabled for the project), inform the user but do not treat it as a fatal error
9. Return the MR URL to the user
10. If the environment variable `SLACK_WEBHOOK_URL_MR` is set, post a Slack notification:
    - Check: `echo "$SLACK_WEBHOOK_URL_MR"` — if empty, skip this step silently
    - Post to Slack using curl:
    ```
    MR_URL="<the MR URL from step 9>"
    curl -s -X POST "$SLACK_WEBHOOK_URL_MR" \
      -H 'Content-Type: application/json' \
      -d "$(cat <<EOF
    {
      "blocks": [
        {
          "type": "section",
          "text": {
            "type": "mrkdwn",
            "text": ":rocket: *Fresh MR just dropped!* :eyes:\n\n<${MR_URL}|${MR_TITLE}>\n\n:pray: Could someone give this a review? :sparkles:"
          }
        }
      ]
    }
    EOF
    )"
    ```
    - If the curl fails, inform the user but do not treat it as a fatal error

## Prerequisites (optional)

To enable Slack notifications when an MR is created or updated, set `SLACK_WEBHOOK_URL_MR` in your Claude settings:

```json
// ~/.claude/settings.json or .claude/settings.local.json
{
  "env": {
    "SLACK_WEBHOOK_URL_MR": "https://hooks.slack.com/services/T.../B.../xxx"
  }
}
```
