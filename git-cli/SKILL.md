---
name: git-cli
description: >-
  Advanced git workflows for orphan branches, clean history publishing,
  multi-remote setups, squash commits, and secret remediation. Use when
  working with git branches, remotes, rebasing, or publishing repos with
  clean history.
---

# Git CLI

## Orphan Branch (Clean History)

Create a branch with no parent commits — useful for publishing a subset of work as a new repo with no prior history.

```bash
# Create orphan branch from current working tree
git checkout --orphan new-branch-name

# Stage all files and commit
git add -A
git commit -m "Initial commit: description"
```

**Important**: After `git checkout --orphan`, all files are staged. If you need to exclude files, `git rm --cached` them before committing, or ensure `.gitignore` is set up first.

### Push orphan branch as main to a new remote

```bash
git remote add v2 git@github.com:owner/new-repo.git
git push v2 new-branch-name:main
```

Subsequent pushes use the same mapping:

```bash
git push v2 new-branch-name:main
```

## Multi-Remote Workflows

### Add a second remote

```bash
git remote add v2 git@github.com:owner/other-repo.git
git remote -v   # verify both remotes
```

### Push different branches to different remotes

```bash
# Push feature branch to origin
git push origin feature-branch

# Push clean branch to v2 as main
git push v2 clean-branch:main
```

### Fetch from a specific remote

```bash
git fetch v2
git log v2/main --oneline -5
```

## Squashing History

### Squash all commits into one (soft reset)

```bash
# On the branch you want to squash
git reset --soft $(git rev-list --max-parents=0 HEAD)
git commit -m "Squashed: single clean commit"
```

### Interactive rebase to squash N commits

```bash
# Squash last 5 commits
git rebase -i HEAD~5
# In editor: change "pick" to "squash" (or "s") for all but the first
```

**Note**: Never use `-i` (interactive) in automated/agent contexts — it requires editor interaction. Use `--soft` reset approach instead.

### Squash merge from feature branch

```bash
git checkout main
git merge --squash feature-branch
git commit -m "feat: merged feature-branch as single commit"
```

## Secret Remediation

### Find secrets in current files

```bash
# Search for common patterns
grep -rn "API_KEY\|SECRET\|PASSWORD\|TOKEN" --include="*.ts" --include="*.env*" --include="*.json"

# Search for hardcoded hex strings (potential keys)
grep -rn '[a-f0-9]\{32,\}' --include="*.ts" --include="*.js"
```

### Find secrets in git history

```bash
# Search all commits for a string
git log -p --all -S "client_secret"
git log -p --all -S "d19b3650"  # partial key

# Search specific file history
git log -p -- .env.example
```

### Remove secrets from history

If secrets were committed, the orphan branch approach is safest — it creates entirely new history:

```bash
git checkout --orphan clean-main
# Edit files to remove secrets
git add -A
git commit -m "Clean initial commit"
git push new-remote clean-main:main --force
```

For targeted removal from existing history (use with caution):

```bash
# Remove a file from all history
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch path/to/secret-file' \
  --prune-empty -- --all

# Or use git-filter-repo (preferred, faster)
pip install git-filter-repo
git filter-repo --path path/to/secret-file --invert-paths
```

### Prevention checklist

- [ ] `.gitignore` includes `.env`, `*.pem`, `credentials.json`, `data/`
- [ ] `.env.example` has placeholder values, not real keys
- [ ] Config files use `process.env.VAR ?? ''` not hardcoded fallbacks
- [ ] Review `git diff --cached` before every commit

## Commit Best Practices

### Message format

```bash
git commit -m "$(cat <<'EOF'
feat(auth): add GitHub OAuth login

- New oauth.ts module with arctic library
- DB migration for oauth_provider/oauth_id columns
- AuthGate shows provider buttons dynamically
EOF
)"
```

Use heredoc for multi-line messages to avoid shell escaping issues.

### Conventional commit prefixes

| Prefix | Use |
|--------|-----|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `refactor` | Code restructure, no behavior change |
| `chore` | Build, CI, dependencies |
| `test` | Adding or fixing tests |

## Useful Commands

### Status and diff

```bash
git status --short           # compact status
git diff --stat              # summary of changes
git diff --cached            # staged changes only
git log --oneline -10        # recent history
```

### Stash

```bash
git stash                    # stash working changes
git stash pop                # restore
git stash list               # see all stashes
```

### Cherry-pick

```bash
git cherry-pick abc1234      # apply a specific commit
git cherry-pick abc1234 --no-commit  # stage without committing
```

### Tags

```bash
git tag v1.0.0
git push origin v1.0.0
git tag -d v1.0.0            # delete local tag
git push origin :v1.0.0      # delete remote tag
```

## Edge Cases Encountered

### Detached HEAD after orphan operations

If you end up in detached HEAD state:

```bash
git checkout -b recovery-branch   # create branch from current state
```

### Push rejected (non-fast-forward)

When pushing an orphan branch to a remote that already has history:

```bash
# Only if you're certain (replaces all remote history)
git push v2 clean-branch:main --force
```

### `.gitignore` not taking effect for tracked files

```bash
# Remove from tracking without deleting the file
git rm --cached path/to/file
# Then commit — the file stays on disk but is no longer tracked
```
