---
name: gh-cli
description: >-
  GitHub CLI (gh) workflows for repo management, OAuth app creation, API
  queries, and release automation. Includes browser MCP fallback for operations
  with no API endpoint (e.g. creating OAuth Apps). Use when working with GitHub
  repos, issues, PRs, OAuth apps, or the gh command.
---

# GitHub CLI (gh)

## Prerequisites

```bash
# Verify auth status and scopes
gh auth status
# Required scopes: repo, read:org (add admin:public_key for SSH)
```

## Repository Operations

### Create and clone a new repo

```bash
gh repo create owner/repo-name --public --description "description" --clone
```

Flags: `--private`, `--public`, `--internal`. Add `--clone` to clone immediately.

### Create from an existing local directory

```bash
cd my-project
git init && git add -A && git commit -m "Initial commit"
gh repo create owner/repo-name --public --source=. --push
```

### Fork and clone

```bash
gh repo clone owner/repo -- --depth 1
gh repo fork owner/repo --clone
```

## Pull Requests

### Create PR with body from heredoc

```bash
gh pr create --title "the title" --body "$(cat <<'EOF'
## Summary
- Change 1
- Change 2

## Test plan
- [ ] Tested locally
EOF
)"
```

### List, view, merge

```bash
gh pr list --state open
gh pr view 42
gh pr merge 42 --squash --delete-branch
```

### Review PR comments

```bash
gh api repos/owner/repo/pulls/42/comments
```

## API Access

### Authenticated REST calls

```bash
# GET
gh api /user
gh api repos/owner/repo/releases/latest

# POST with JSON body
gh api repos/owner/repo/issues --method POST \
  -f title="Bug report" -f body="Details here"

# Paginated listing
gh api repos/owner/repo/issues --paginate
```

### GraphQL

```bash
gh api graphql -f query='{ viewer { login } }'
```

## OAuth App Creation

**There is no `gh` CLI command or REST API to create OAuth Apps.** This can only be done through the GitHub web UI or browser automation.

### Browser MCP approach (Cursor)

When Cursor's browser MCP is available, automate the form:

```
1. browser_navigate -> https://github.com/settings/applications/new
2. browser_lock
3. browser_snapshot (get element refs)
4. browser_fill ref for "Application name"
5. browser_fill ref for "Homepage URL"
6. browser_fill ref for "Authorization callback URL"
7. browser_click ref for "Register application"
8. Wait 3s, browser_snapshot to capture Client ID from page
9. For "Generate a new client secret":
   - This is an <input type="submit" readonly> not a <button>
   - browser_click WILL FAIL with "stale element reference"
   - Workaround: browser_unlock, ask user to click, then browser_snapshot
10. Capture secret from page snapshot text
```

### Known browser MCP edge cases

| Issue | Cause | Workaround |
|-------|-------|------------|
| `stale element reference: expected button but found (input)` | GitHub uses `<input type="submit">` for CSRF-protected forms | Unlock browser, have user click, then snapshot |
| Login page on navigate | Not authenticated in embedded browser | Unlock, let user sign in, then continue |
| Page unchanged after click | Form submission still loading | `sleep 3` then `browser_snapshot` |

### Manual fallback

If no browser MCP is available, guide the user:

1. Go to https://github.com/settings/applications/new
2. Fill in: name, homepage URL, callback URL
3. Click "Register application"
4. Copy Client ID
5. Click "Generate a new client secret" and copy it
6. Provide the values back

## Releases

```bash
# Create a release from a tag
gh release create v1.0.0 --title "v1.0.0" --generate-notes

# Upload assets
gh release upload v1.0.0 ./dist/app.tar.gz

# List releases
gh release list
```

## Secrets (for GitHub Actions)

```bash
# Set a repo secret
gh secret set SECRET_NAME --body "value"

# Set from file
gh secret set SECRET_NAME < secret.txt

# List secrets
gh secret list
```

## Common Patterns

### Check if repo exists before creating

```bash
if gh repo view owner/repo &>/dev/null; then
  echo "Repo already exists"
else
  gh repo create owner/repo --public
fi
```

### Get authenticated user info

```bash
gh api /user --jq '.login'
```

### Search across repos

```bash
gh search repos "keyword" --owner=myuser --json name,url
gh search code "CONFIG_KEY" --repo owner/repo
```
