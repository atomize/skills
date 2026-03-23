# skills

Reusable [Agent Skills](https://agentskills.io) for CLI workflows. Works with Cursor, Claude Code, Codex, GitHub Copilot, and [40+ other agents](https://github.com/vercel-labs/skills#supported-agents).

## Install

```bash
npx skills add atomize/skills
```

### Install specific skills

```bash
npx skills add atomize/skills --skill gh-cli
npx skills add atomize/skills --skill render-cli
npx skills add atomize/skills --skill git-cli
```

### Install globally (available across all projects)

```bash
npx skills add atomize/skills -g
```

### Install for a specific agent

```bash
npx skills add atomize/skills -a cursor
npx skills add atomize/skills -a claude-code
npx skills add atomize/skills -a codex
```

### List available skills before installing

```bash
npx skills add atomize/skills --list
```

## Available Skills

| Skill | Description |
|-------|-------------|
| **gh-cli** | GitHub CLI — repo management, OAuth app creation (with browser automation fallback), API queries, PRs, releases, secrets |
| **render-cli** | Render CLI & API — blueprint deployment, env var management via REST API, deploy monitoring, monorepo build order, troubleshooting |
| **git-cli** | Git — orphan branches, clean history publishing, multi-remote workflows, squash strategies, secret remediation |

## What are Agent Skills?

Agent Skills are an open standard for extending AI coding agents with domain-specific knowledge. Each skill is a `SKILL.md` file with YAML frontmatter that agents auto-discover and apply when relevant.

These skills encode real-world edge cases and workarounds discovered through production use — things like:

- GitHub's OAuth App creation has no API (requires browser automation or manual UI)
- Render's CLI has no `env set` command (must use REST API with specific endpoint patterns)
- `git rebase -i` can't be used in agent/CI contexts (use `--soft` reset instead)

## Structure

```
skills/
├── gh-cli/
│   └── SKILL.md
├── render-cli/
│   └── SKILL.md
└── git-cli/
    └── SKILL.md
```

Skills follow the [Agent Skills spec](https://agentskills.io). Each `SKILL.md` has:

- **Frontmatter**: `name` and `description` (used for auto-discovery)
- **Body**: Concise instructions, commands, edge cases, and workarounds

## Manual Installation

If you prefer not to use `npx skills`, copy or symlink skill directories directly:

```bash
# Symlink to Cursor global skills
ln -s /path/to/skills/skills/gh-cli ~/.cursor/skills/gh-cli

# Or copy to a project
cp -r /path/to/skills/skills/gh-cli .cursor/skills/gh-cli
```

## License

MIT
