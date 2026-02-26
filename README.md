# AI skills

Claude Code plugin with custom workflow skills.

## Included skills

- `mr` - Push branch, create GitLab MR with auto-merge and delete source branch
- `daily-status` - Generate daily status report from GitLab achievements
- `implement-spec` - Implement a spec by working through tasks sequentially
- `review-mr` - Review a GitLab MR, fix issues, and create a fix MR
- `review-spec` - Review a spec file for completeness and consistency
- `spec` - Build a spec from a prompt or file
- `tdd-fix` - Test-driven bug fixing
- `verify` - Verify spec implementation completeness
- `create-subagent` - Create custom subagents for Cursor

## Install in Claude Code from GitHub

Add this repo as a custom marketplace in `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": [
    {
      "name": "ai-skills",
      "source": {
        "source": "github",
        "repo": "<owner>/ai-skills"
      }
    }
  ]
}
```

Then install the plugin:
```
/plugin install ai-skills@ai-skills
```

Updates from the repo will be picked up automatically.
