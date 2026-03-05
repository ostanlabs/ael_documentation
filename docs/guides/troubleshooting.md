# Troubleshooting

Common issues and fixes, organised by where in the setup flow they appear.

---

## Bootstrap issues

### Docker not found or not running

```
Error: Docker Engine not detected.
Please ensure Docker Desktop is running.
```

Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) and make sure it's started before running `ploston bootstrap`.

### Port already in use

```
Error: Port 8082 is already in use (Control Plane).
```

Another process is on 8082. Either stop it, or override the port:

```bash
# Edit ~/.ploston/.env
AEL_PORT=8083

# Then regenerate and restart
ploston bootstrap --regenerate
ploston bootstrap up
```

### Images fail to pull

```
Error: Failed to pull ostanlabs/ploston:latest
```

Check your internet connection and Docker Hub access:

```bash
docker pull ostanlabs/ploston:latest   # test directly
```

If you're behind a corporate proxy, configure Docker's proxy settings in Docker Desktop → Settings → Resources → Proxies.

### Control Plane starts but stays unhealthy

```bash
ploston bootstrap logs --service ploston
```

Common causes and fixes:

| Log message | Fix |
|-------------|-----|
| `Redis connection refused` | Wait 10s — Redis may still be starting. Run `ploston bootstrap up` again. |
| `Config not found` | Run `ploston init --import` to push initial config. |
| `Address already in use` | Another process on the CP port — see "Port already in use" above. |

---

## `init --import` issues

### No MCP configs found

```
No MCP configurations detected.
```

Ploston scans Claude Desktop and Cursor config files. If you haven't configured either, create a minimal Claude Desktop config first, or manually specify an MCP server:

```bash
ploston init --import --add-server "filesystem:npx -y @modelcontextprotocol/server-filesystem /tmp"
```

### Secret not found in environment

```
⚠  GITHUB_TOKEN not set — github server will be imported without token
```

The importer detected a secret in your existing config that isn't in your environment. Either set the variable before running init:

```bash
export GITHUB_TOKEN=ghp_...
ploston init --import
```

Or add it to `~/.ploston/.env` after the fact and restart the runner:

```bash
echo "GITHUB_TOKEN=ghp_..." >> ~/.ploston/.env
ploston runner restart
```

### Runner fails to start

```
Error: Runner failed to connect to Control Plane at http://localhost:8082
```

Confirm the CP is healthy first:

```bash
ploston bootstrap status
curl http://localhost:8082/health
```

If the CP is healthy but the runner still can't connect:

```bash
ploston runner logs          # check runner output
ploston runner restart
```

---

## Runner issues

### Runner shows "disconnected"

```bash
ploston runner status
# → Status: disconnected
```

The runner lost its WebSocket connection to the Control Plane. Restart it:

```bash
ploston runner restart
```

If it keeps disconnecting, check for CP errors:

```bash
ploston bootstrap logs --service ploston | grep ERROR
```

### Tools missing after runner restart

Tool discovery happens on runner connect. After a restart, wait ~5 seconds, then check:

```bash
ploston tools list
```

If tools are still missing, the MCP server for that tool may have failed to start:

```bash
ploston runner logs --follow     # watch for MCP server startup errors
```

---

## Workflow issues

### Validation fails

```bash
ploston validate my-workflow.yaml
# Error: ...
```

Common errors:

| Error | Fix |
|-------|-----|
| `Missing required field: name` | Add `name:` to the top of the YAML |
| `Invalid step id` | Step IDs must be alphanumeric + hyphens, no spaces |
| `Syntax error at line N` | YAML indentation issue — use spaces, not tabs |
| `Unknown step type` | Step must have either `tool:` or `code:` but not both |
| `Circular dependency` | A step's `depends_on` creates a loop — review the chain |

### Tool not found at runtime

```
Error: TOOL_NOT_FOUND — tool 'my_tool' is not registered
```

```bash
ploston tools list             # see what's registered
ploston runner status          # confirm runner is connected
```

The tool name in your workflow must exactly match the registered name, including the `local__<server>__<tool>` prefix for runner tools.

### Step timeout

```
Error: CODE_TIMEOUT — execution timeout after 30s
```

Increase the timeout on the slow step:

```yaml
steps:
  - id: slow_step
    timeout: 120        # seconds
    code: |
      ...
```

Or globally in config:

```yaml
execution:
  default_timeout: 300
```

### Code step: `result` is None

The sandbox captured `None` because `result` was never assigned. Every code step must set the `result` variable:

```python
# ✅ correct
data = context.inputs["query"]
result = {"processed": data.upper()}

# ❌ wrong — result is never set
data = context.inputs["query"]
output = data.upper()           # 'output' is ignored
```

### Code step: import not allowed

```
SecurityError: Import 'requests' not allowed.
```

Blocked modules cannot be imported in code steps — use a `tool` step for network requests instead:

```yaml
# ❌ Don't do this in a code step
import requests
response = requests.get(url)

# ✅ Do this — use a tool step
- id: fetch
  tool: http_request
  params:
    url: "{{ inputs.url }}"
    method: GET
```

If you need a module that's blocked and you believe it should be allowed (e.g., `yaml`, `pydantic`), add it to `python_exec.allowed_imports` in your config.

### Code step: variable not found in context

```
KeyError: 'some_step'
```

The step ID in `context.steps["some_step"]` doesn't exist or hasn't run yet. Check:
- The referenced step ID matches exactly (case-sensitive)
- The code step has `depends_on: [some_step]` so it runs after

---

## MCP / Claude Desktop issues

### Workflow tool not visible in Claude Desktop

After registering a new workflow:

1. Confirm it's registered: `ploston workflows list`
2. Restart Claude Desktop — MCP tool lists are loaded at startup
3. If still missing, check the CP logs: `ploston bootstrap logs --service ploston`

### Claude Desktop shows Ploston as disconnected

Check that the Ploston entry in Claude Desktop's config points to the right address:

```json
{
  "mcpServers": {
    "ploston": {
      "command": "ploston",
      "args": ["serve"]
    }
  }
}
```

Then restart Claude Desktop.

---

## Getting more information

### Enable debug logging

Add to `~/.ploston/.env`:

```
LOG_LEVEL=DEBUG
```

Then restart: `ploston bootstrap down && ploston bootstrap up`

### View all logs in one stream

```bash
ploston bootstrap logs --follow
```

### Check service health directly

```bash
curl -s http://localhost:8082/health | python3 -m json.tool
curl -s http://localhost:8081/health | python3 -m json.tool
```

### File a bug report

Include:
- `ploston version` output
- `ploston bootstrap status` output
- Relevant log lines from `ploston bootstrap logs`
- Workflow YAML (if the issue is workflow-related)
- OS and Docker version

[Open an issue on GitHub →](https://github.com/ostanlabs/ploston/issues)
