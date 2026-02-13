# AI skills

Packaged skills exported from local Cursor and Claude sources for Codex installation.

## Included

### Cursor-origin skills
- `create-subagent`

### Claude command-derived skills
- `commit-and-mr`
- `daily-status`
- `implement-spec`
- `review-mr`
- `review-spec`
- `spec`
- `tdd-fix`
- `verify`

## Install in Codex from GitHub

Install one:

```bash
python /Users/p3l/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo <owner>/<repo> \
  --path skills/spec
```

Install multiple:

```bash
python /Users/p3l/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo <owner>/<repo> \
  --path skills/spec skills/review-spec skills/implement-spec
```

Restart Codex after install.
