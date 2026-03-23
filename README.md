# Skills

Reusable [Cursor Agent Skills](https://docs.cursor.com/context/skills) for CLI workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [gh-cli](gh-cli/SKILL.md) | GitHub CLI — repo management, OAuth app creation (with browser MCP fallback), API queries, releases, secrets |
| [render-cli](render-cli/SKILL.md) | Render CLI & API — blueprint deployment, env var management, deploy monitoring, troubleshooting |
| [git-cli](git-cli/SKILL.md) | Git — orphan branches, clean history publishing, multi-remote workflows, secret remediation |

## Usage

### As personal skills (available in all projects)

```bash
# Symlink into your personal skills directory
ln -s /path/to/skills/gh-cli ~/.cursor/skills/gh-cli
ln -s /path/to/skills/render-cli ~/.cursor/skills/render-cli
ln -s /path/to/skills/git-cli ~/.cursor/skills/git-cli
```

### As project skills (shared with repo collaborators)

```bash
# Copy into your project
cp -r /path/to/skills/gh-cli .cursor/skills/gh-cli
```

## Structure

Each skill follows the Cursor SKILL.md format:

```
skill-name/
└── SKILL.md    # Frontmatter (name + description) + instructions
```

The `description` field in frontmatter determines when the agent auto-applies the skill. Each skill is self-contained in a single `SKILL.md` under 500 lines.
