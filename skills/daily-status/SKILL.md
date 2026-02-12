---
name: daily-status
description: Generate a daily status report of my GitLab achievements from my last working day.
---

Generate a daily status report of my GitLab achievements from my last working day.

## Instructions

Follow these steps to generate the status report:

### Step 1: Determine Target Date

Calculate the "last working day":
- If today is Monday, use last Friday (3 days ago)
- Otherwise, use yesterday

Use bash to determine this:
```bash
# On macOS
DAY_OF_WEEK=$(date +%u)
if [ "$DAY_OF_WEEK" -eq 1 ]; then
  echo "TARGET: $(date -v-3d +%Y-%m-%d)"
else
  echo "TARGET: $(date -v-1d +%Y-%m-%d)"
fi
```

### Step 2: Get GitLab Username

```bash
glab api user | jq -r '.username'
```

### Step 3: Get Projects with Recent Activity

Query user events to find projects where you pushed commits on or after the target date:

```bash
# Get push events from target date onwards (replace TARGET_DATE with YYYY-MM-DD)
glab api "events?action=pushed&after=TARGET_DATE" | jq -r '.[] | select(.push_data.commit_count > 0) | .project_id' | sort -u
```

This returns only project IDs where you actually pushed code, avoiding the need to iterate through thousands of projects.

### Step 4: Fetch My Commits

For each project ID from Step 3, fetch the project path and commit details:

```bash
# Get project path for display
glab api "projects/PROJECT_ID" | jq -r '.path_with_namespace'

# Get commits for the target date (replace TARGET_DATE with YYYY-MM-DD)
glab api "projects/PROJECT_ID/repository/commits?since=TARGET_DATET00:00:00Z&until=TARGET_DATET23:59:59Z&author=USERNAME&per_page=50" | jq -r '.[] | "\(.short_id)|\(.title)|\(.created_at)"'
```

### Step 5: Handle No Commits Found

If no push events are found for the target date:
1. Query events without date filter to get recent activity: `glab api "events?action=pushed&per_page=50"`
2. Find the most recent push event date from the results
3. Use that date instead and note it in the output

### Step 6: Get Associated MRs

For each commit found, get the associated merge request URL:
```bash
glab api "projects/PROJECT_ID/repository/commits/COMMIT_SHA/merge_requests" | jq -r '.[0] | "\(.iid)|\(.web_url)"'
```

Group commits that belong to the same MR together.

### Step 7: Summarize Into Achievements

Transform the raw commits into achievement-focused bullet points:

**Summarization rules:**
- Group related commits (same MR, same feature) into ONE bullet point
- Focus on "what was delivered" not "what was done"
- Use past tense, achievement-oriented language
- Keep each bullet to 1-2 sentences max
- Extract the meaningful feature/fix from commit messages

**Example transformation:**
```
Input commits:
- "feat: add user validation endpoint"
- "fix: handle null case in validation"
- "test: add unit tests for validation"

Output:
- Implemented user input validation with error handling and test coverage
```

### Output Format

Output ONLY the final markdown status. Do not include the steps or debugging info.

Format:
```markdown
## Daily Status - [Day of Week], [Month Day, Year]

- **[project-name]** Achievement description ([MR !number](url))
- **[project-name]** Another achievement ([MR !number](url))
- **[project-name]** Third achievement (no MR)
```

Rules:
- Project name in bold brackets at the start
- MR link at end in parentheses (if available)
- If no MR, omit the link portion or note "(no MR)"
- Sort by project name for readability
- Use the short project name (last segment of path), not full path

If absolutely no commits found in the last 7 days, output:
```markdown
## Daily Status

No commits found in the last 7 days across your GitLab projects.
```
