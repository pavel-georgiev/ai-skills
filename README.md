# AI skills

Claude Code plugin with custom workflow skills for commits, specs, MRs, code review, and daily status.

## Included skills

- `commit` - Review current changes and create a single git commit
- `mr` - Push branch, create GitLab MR with auto-merge and delete source branch
- `daily-status` - Generate daily status report from GitLab achievements
- `implement-spec` - Implement a spec by working through tasks sequentially
- `review-mr` - Review a GitLab MR, fix issues, and create a fix MR
- `review-spec` - Review a spec file for completeness and consistency
- `spec` - Build a spec from a prompt or file
- `tdd-fix` - Test-driven bug fixing
- `verify` - Verify spec implementation completeness
- `create-subagent` - Create custom subagents for Cursor

## Install in Claude Code

1. Add the marketplace:

```bash
claude plugin marketplace add pavel-georgiev/ai-skills
```

2. Install the plugin:

```bash
claude plugin install ai-skills@ai-skills
```

3. To update to the latest version:

```bash
claude plugin update ai-skills@ai-skills
```
