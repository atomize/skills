---
name: render-cli
description: >-
  Render CLI and API workflows for deploying services, managing environment
  variables, blueprints, and troubleshooting failed deployments. Includes
  direct REST API patterns for operations the CLI cannot perform. Use when
  deploying to Render, setting env vars, debugging deploys, or working with
  render.yaml blueprints.
---

# Render CLI & API

## Prerequisites

```bash
# Check CLI is installed and authenticated
render --version
# API key is stored in ~/.render/cli.yaml
grep 'key:' ~/.render/cli.yaml | awk '{print $2}'
```

## Blueprint (render.yaml)

### Structure

```yaml
services:
  - type: web          # web, worker, private, background, cron
    name: my-service
    runtime: docker     # docker, node, python, go, rust, etc.
    dockerfilePath: docker/Dockerfile
    dockerCommand: ./entrypoint.sh
    region: oregon
    plan: starter       # free, starter, standard, pro, etc.
    envVars:
      - key: MY_VAR
        value: "static-value"
      - key: MY_SECRET
        sync: false           # declare but don't set — must be set manually
      - fromGroup: my-secrets # reference an env var group
    healthCheckPath: /

envVarGroups:
  - name: my-secrets
    envVars:
      - key: API_KEY
        sync: false
```

### Validate a blueprint

```bash
render blueprint validate render.yaml
```

### Deploy a blueprint

**The CLI cannot create new blueprints.** Use the Dashboard:
1. Go to https://dashboard.render.com/select-repo?type=blueprint
2. Select the repo
3. Click "Apply"

After initial creation, updates to `render.yaml` auto-apply on push if auto-deploy is enabled.

## Service Management

### List services

```bash
render services list --output json
```

Parse with Python for scripting:

```bash
render services list --output json | python3 -c "
import sys, json
for s in json.load(sys.stdin):
    svc = s.get('service', s)
    print(f\"{svc.get('name')}: {svc.get('id')}\")"
```

### Restart a service

```bash
render services restart srv-xxxx --confirm
```

## Environment Variables

### The CLI has no `env set` command

Use the Render REST API directly:

```bash
RENDER_API_KEY=$(grep 'key:' ~/.render/cli.yaml | awk '{print $2}')
SERVICE_ID="srv-xxxx"

# Set a service-level env var
curl -s -X PUT "https://api.render.com/v1/services/${SERVICE_ID}/env-vars/VAR_NAME" \
  -H "Authorization: Bearer ${RENDER_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"value":"the-value"}'
```

### Env group variables

```bash
# Find env group IDs
curl -s "https://api.render.com/v1/env-groups" \
  -H "Authorization: Bearer ${RENDER_API_KEY}" | python3 -c "
import sys, json
for g in json.load(sys.stdin):
    eg = g.get('envGroup', g)
    print(f\"{eg.get('name')}: {eg.get('id')}\")"

# Set a variable in an env group (individual PUT, NOT batch)
ENV_GROUP_ID="evg-xxxx"
curl -s -X PUT "https://api.render.com/v1/env-groups/${ENV_GROUP_ID}/env-vars/VAR_NAME" \
  -H "Authorization: Bearer ${RENDER_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"value":"the-value"}'
```

### API endpoint edge cases

| Endpoint | Works? | Notes |
|----------|--------|-------|
| `PUT /v1/services/{id}/env-vars/{key}` | Yes | Set one service env var |
| `PUT /v1/env-groups/{id}/env-vars/{key}` | Yes | Set one env group var |
| `PUT /v1/env-groups/{id}/env-vars` (batch) | **404** | Does not exist despite intuition |
| `PATCH /v1/services/{id}/env-vars` | **No** | Not a valid endpoint |

### Special characters in values

Shell interpolation breaks values containing `_`, `=`, `/`, `+`. Use a script:

```bash
node -e "
const val = process.env.MY_TOKEN;  // read from .env or direct
fetch('https://api.render.com/v1/services/srv-xxxx/env-vars/TOKEN', {
  method: 'PUT',
  headers: {
    'Authorization': 'Bearer ' + process.env.RENDER_API_KEY,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ value: val }),
}).then(r => r.json()).then(console.log);
"
```

### Env var changes don't auto-redeploy

After setting service-level env vars, trigger a deploy manually:

```bash
render deploys create srv-xxxx --confirm
```

## Deploys

### Trigger a deploy

```bash
render deploys create srv-xxxx --confirm
```

### Monitor deploy status

```bash
render deploys list srv-xxxx --output json | python3 -c "
import sys, json
data = json.load(sys.stdin)
dep = data[0].get('deploy', data[0])
print(f\"{dep.get('id')}: {dep.get('status')}\")"
```

Status progression: `created` -> `build_in_progress` -> `update_in_progress` -> `live`

### View logs

```bash
render logs --service-id srv-xxxx --tail 50
```

**Note**: The `--service-id` flag works for `logs` but NOT for `deploys list` (which takes a positional argument).

## Troubleshooting Failed Deploys

### Build failures

1. Check build logs: `render logs --service-id srv-xxxx --tail 100`
2. Common causes:
   - **Missing dependency**: Package not in `package.json` / `requirements.txt`
   - **TypeScript module not found**: Build order wrong in Dockerfile for monorepos
   - **Docker build context**: Files outside context not accessible

### Monorepo build order (pnpm workspaces + Docker)

Build packages in dependency order:

```dockerfile
# Build in dependency order: shared first, then packages that depend on it
RUN pnpm --filter @scope/shared build \
 && pnpm --filter @scope/lib-a build \
 && pnpm --filter @scope/server build \
 && pnpm --filter @scope/frontend build
```

### Runtime failures

1. Check runtime logs after deploy goes `live`
2. Common causes:
   - **Missing env var**: `sync: false` vars not set — use API to set them
   - **Port binding**: Render expects the service to bind to `PORT` env var (default 10000)
   - **Health check failing**: Ensure `healthCheckPath` returns 200

### Service won't start after env var update

Env var changes to service-level vars require a manual redeploy:

```bash
render deploys create srv-xxxx --confirm
```

## Common Patterns

### Full deploy workflow

```bash
# 1. Push code
git push origin main

# 2. Set any new env vars
curl -s -X PUT ".../env-vars/NEW_VAR" -d '{"value":"..."}'

# 3. Trigger deploy if env-only change
render deploys create srv-xxxx --confirm

# 4. Monitor
watch -n 5 'render deploys list srv-xxxx --output json | python3 -c "
import sys, json; d=json.load(sys.stdin)[0].get(\"deploy\",json.load(sys.stdin)[0])
print(d.get(\"status\"))"'
```

### Verify a web service is live

```bash
curl -sI https://my-service.onrender.com/ | head -5
```
