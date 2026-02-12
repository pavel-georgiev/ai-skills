---
name: review-mr
description: Review a GitLab MR, fix issues, and create a fix MR targeting the feature branch
---


Review a GitLab merge request for code issues. If issues are found, create fixes and open a new MR targeting the feature branch, then comment on the original MR with a link.

Create a todo list before starting to track progress.

## Step 1: MR Selection

Determine which MR to review:

1. First, try to get the MR for the current branch:
   ```bash
   glab mr view -F json
   ```

2. If no MR exists for current branch, list open MRs and use AskUserQuestion to let the user select:
   ```bash
   glab mr list
   ```

3. Extract and store:
   - MR number (iid)
   - Source branch name
   - Target branch name
   - MR title and description
   - MR web URL

## Step 2: Triage Check

Launch a haiku agent to check if review should proceed. Skip if ANY of these are true:
- The MR is closed or merged
- The MR is a draft
- The MR does not need code review (e.g., automated MR, trivial change that is obviously correct)
- You have already submitted a code review comment on this MR (check existing comments)

Note: Still review Claude/AI-generated MRs.

If any condition is true, inform the user and stop.

## Step 3: Gather Context

1. Launch a haiku agent to find all relevant CLAUDE.md files:
   - The root CLAUDE.md file, if it exists
   - Any CLAUDE.md files in directories containing files modified by the MR

2. Get the MR diff:
   ```bash
   glab mr diff <mr-number>
   ```

3. Get MR details for context (title, description)

## Step 4: Parallel Code Review

Launch 4 agents IN PARALLEL (single message, multiple Task tool calls) to independently review the changes. Each agent should return a list of issues with:
- File path and line number(s)
- Description of the issue
- Reason it was flagged (e.g., "CLAUDE.md adherence", "bug", "security")
- Suggested fix

**Agents 1 + 2: CLAUDE.md compliance sonnet agents**
Audit changes for CLAUDE.md compliance in parallel. When evaluating compliance, only consider CLAUDE.md files that share a file path with the file or its parents.

**Agent 3: Opus bug agent**
Scan for obvious bugs. Focus only on the diff itself without reading extra context. Flag only significant bugs; ignore nitpicks and likely false positives. Do not flag issues that cannot be validated without context outside of the git diff.

**Agent 4: Opus bug agent**
Look for problems in the introduced code: security issues, incorrect logic, etc. Only look for issues within the changed code.

**CRITICAL: We only want HIGH SIGNAL issues:**
- Objective bugs that will cause incorrect behavior at runtime
- Clear, unambiguous CLAUDE.md violations where you can quote the exact rule being broken
- Security vulnerabilities

**We do NOT want:**
- Subjective concerns or "suggestions"
- Style preferences not explicitly required by CLAUDE.md
- Potential issues that "might" be problems
- Anything requiring interpretation or judgment calls
- Pre-existing issues
- Issues a linter will catch
- General code quality concerns unless explicitly required in CLAUDE.md
- Issues mentioned in CLAUDE.md but explicitly silenced in code (e.g., lint ignore comments)
- Pedantic nitpicks that a senior engineer would not flag

If you are not certain an issue is real, do not flag it. False positives erode trust and waste reviewer time.

Provide each agent with the MR title and description for context about the author's intent.

## Step 5: Issue Validation

For each issue found in Step 4, launch parallel subagents to validate:
- Use Opus agents for bugs and logic issues
- Use Sonnet agents for CLAUDE.md violations

Each validation agent receives:
- MR title and description
- The issue description
- Access to read the relevant files

The agent's job is to confirm with high confidence that the issue is real. For example:
- If "variable is not defined" was flagged, verify it's actually undefined
- If a CLAUDE.md violation was flagged, verify the rule applies to this file and is actually violated

## Step 6: Filter and Collect Issues

Filter out any issues not validated in Step 5. This gives the final list of high-signal issues.

If no validated issues remain, skip to Step 8 (comment with "no issues found").

## Step 7: Create Fixes

If validated issues exist:

1. Use AskUserQuestion to ask how fixes should be submitted:
   - **Option A: New MR** - Create a separate fix MR targeting the source branch (creates review-fixes/<mr-number> branch)
   - **Option B: Push to existing MR** - Push fixes directly to the source branch of the original MR

2. Checkout the appropriate branch:

   **If New MR (Option A):**
   ```bash
   git fetch origin <source-branch>
   git checkout -b review-fixes/<mr-number> origin/<source-branch>
   ```

   **If Push to existing MR (Option B):**
   ```bash
   git fetch origin <source-branch>
   git checkout <source-branch>
   git pull origin <source-branch>
   ```

3. For each validated issue:
   - Use Edit tool to apply the fix
   - Keep fixes minimal and focused

4. Stage and commit all fixes:
   ```bash
   git add -A
   git commit -m "fix: address code review issues from MR !<mr-number>

   - <brief description of fix 1>
   - <brief description of fix 2>
   ...

   Generated with Claude Code"
   ```

5. Push the changes:

   **If New MR (Option A):**
   ```bash
   git push -u origin review-fixes/<mr-number>
   ```

   **If Push to existing MR (Option B):**
   ```bash
   git push origin <source-branch>
   ```

6. **If New MR (Option A) only** - Create fix MR targeting the SOURCE branch:
   ```bash
   glab mr create --source-branch review-fixes/<mr-number> --target-branch <source-branch> --title "fix: code review fixes for !<mr-number>" --description "$(cat <<'EOF'
   ## Summary

   Automated fixes for code review issues found in !<mr-number>.

   **Issues Fixed:**
   - <issue 1 brief description>
   - <issue 2 brief description>

   ## Details

   <For each issue: description, file, line, what was fixed>

   ---
   Generated with Claude Code
   EOF
   )"
   ```

7. Store the fix MR URL (if created) for the comment.

## Step 8: Comment on Original MR

Post a comment on the original MR using:
```bash
glab mr comment <mr-number> -m "$(cat <<'EOF'
<comment content>
EOF
)"
```

### If issues were found and fixed (New MR):

```markdown
## Auto Code Review

Found X issues and created fixes.

**Issues Fixed:**

1. <brief description> (CLAUDE.md says: "<exact quote>")

   [<file>#L<start>-L<end>](<gitlab-link-to-code>)

2. <brief description> (bug: <explanation>)

   [<file>#L<start>-L<end>](<gitlab-link-to-code>)

**Fix MR:** !<fix-mr-number> (<fix-mr-url>)

Please review and merge the fix MR into your branch.
```

### If issues were found and fixed (Push to existing MR):

```markdown
## Auto Code Review

Found X issues and pushed fixes to this branch.

**Issues Fixed:**

1. <brief description> (CLAUDE.md says: "<exact quote>")

   [<file>#L<start>-L<end>](<gitlab-link-to-code>)

2. <brief description> (bug: <explanation>)

   [<file>#L<start>-L<end>](<gitlab-link-to-code>)

**Commit:** <commit-sha>

Fixes have been pushed directly to the source branch.
```

### If issues were found but could not be automatically fixed:

```markdown
## Auto Code Review

Found X issues requiring manual attention:

1. <brief description>

   [<file>#L<start>-L<end>](<gitlab-link-to-code>)

   **Suggested fix:** <suggestion>
```

### If no issues were found:

```markdown
## Auto Code Review

No issues found. Checked for bugs and CLAUDE.md compliance.
```

## GitLab Link Format

When linking to code, use this format:
```
https://gitlab.com/<namespace>/<project>/-/blob/<full-sha>/<file-path>#L<start>-<end>
```

Requirements:
- Use full git SHA (not abbreviated)
- Get SHA with: `git rev-parse HEAD` or from the MR's source branch
- Line range format: `#L<start>-<end>`
- Provide at least 1 line of context before and after the issue

To get the GitLab project path:
```bash
glab repo view -F json | jq -r '.full_path'
```

## Final Step

Return the original MR URL and (if created) the fix MR URL to the user.
