# Configuration Reference

Complete reference for Ploston configuration options.

---

## How configuration works

Ploston's configuration is managed in one of two ways depending on how you set it up:

**Via `ploston init --import` (recommended)** — Configuration is pushed to the Control Plane through the MCP config tools and stored in Redis. The compose file at `~/.ploston/docker-compose.yaml` and secrets at `~/.ploston/.env` are managed automatically. You don't edit YAML files by hand.

**Via config file (advanced/self-hosted)** — If you're running the Control Plane manually (from source or a custom deployment), you can pass a YAML config file. The CP searches for it in this order:

1. `--config <path>` CLI flag
2. `AEL_CONFIG_PATH` environment variable
3. `./ael-config.yaml` in the current directory
4. `~/.ploston/config.yaml`

---

## Configuration modes

The Control Plane operates in one of two modes:

### Configuration mode

Active when no valid config has been committed yet. Only the config MCP tools are available — no workflows, no tool execution. This is the state the CP starts in after `ploston bootstrap`, before `ploston init --import` completes.

| Tool | Description |
|------|-------------|
| `config_get` | Read the current staged configuration |
| `config_set` | Stage a configuration change |
| `config_validate` | Validate the staged configuration |
| `config_done` | Commit config and switch to running mode |
| `config_location` | Get or set the config file location |

### Running mode

Active after `ploston init --import` completes. All tools, workflows, and execution features are available. This is the normal operating state.

Force a specific mode (useful for debugging):

```bash
ploston serve --mode configuration
ploston serve --mode running   # fails if no valid config exists
```

---

## Complete configuration schema

```yaml
# ═══════════════════════════════════════════════════════════
# PLOSTON CONFIGURATION FILE
# ═══════════════════════════════════════════════════════════

# Server identity
server:
  name: "ploston"
  version: "0.1.0"

# MCP server connections (proxied through the local runner)
mcp:
  servers:
    filesystem:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
      env: {}

    github:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-github"]
      env:
        GITHUB_TOKEN: "${GITHUB_TOKEN}"

# Workflow settings
workflows:
  directory: "./workflows"
  hot_reload: true          # auto-reload when YAML files change

# Execution settings
execution:
  max_concurrent: 10        # max parallel workflow executions
  default_timeout: 300      # seconds

  retry:
    max_attempts: 3
    backoff_multiplier: 2.0

# Python sandbox settings
python_exec:
  timeout: 30               # seconds per code step
  max_memory: 536870912     # 512 MB

  allowed_imports:          # extend the default whitelist
    - json
    - re
    - datetime
    - math
    - collections
    - itertools
    - functools

# Logging
logging:
  level: INFO               # DEBUG | INFO | WARNING | ERROR
  format: text              # text | json

  components:
    workflow: true
    step: true
    tool: true
    sandbox: true

  options:
    show_params: false       # log tool parameters
    show_results: false      # log step outputs
    truncate_at: 1000

# Security
security:
  allowed_hosts: []         # whitelist for outbound HTTP (empty = allow all)
  blocked_hosts: []

# Telemetry
telemetry:
  enabled: false
  endpoint: ""
```

---

## Section reference

### `server`

| Field | Default | Description |
|-------|---------|-------------|
| `name` | `"ploston"` | MCP server name reported to clients |
| `version` | `"0.1.0"` | Server version |

### `mcp`

Defines MCP servers the runner will connect to. Each entry is a server name mapped to a launch command.

```yaml
mcp:
  servers:
    my-server:
      command: "python"        # executable
      args: ["-m", "my_mcp"]  # arguments
      env:
        API_KEY: "${MY_KEY}"  # environment variables
```

| Field | Required | Description |
|-------|----------|-------------|
| `command` | Yes | Executable to run |
| `args` | No | Command arguments |
| `env` | No | Environment variables — use `${VAR}` for substitution |

### `workflows`

| Field | Default | Description |
|-------|---------|-------------|
| `directory` | `"./workflows"` | Path to workflow YAML files |
| `hot_reload` | `true` | Reload workflows when files change |

### `execution`

| Field | Default | Description |
|-------|---------|-------------|
| `max_concurrent` | `10` | Max concurrent workflow executions |
| `default_timeout` | `300` | Default step timeout (seconds) |
| `retry.max_attempts` | `3` | Default max retry attempts |
| `retry.backoff_multiplier` | `2.0` | Exponential backoff multiplier |

### `python_exec`

| Field | Default | Description |
|-------|---------|-------------|
| `timeout` | `30` | Code step execution timeout (seconds) |
| `max_memory` | `536870912` | Memory limit (bytes) — 512 MB default |
| `allowed_imports` | See sandbox defaults | Extend the module whitelist |

### `logging`

| Field | Default | Description |
|-------|---------|-------------|
| `level` | `"INFO"` | Log level |
| `format` | `"text"` | `text` for development, `json` for production |
| `components.*` | `true` | Toggle logging per component |
| `options.show_params` | `false` | Include tool parameters in logs |
| `options.show_results` | `false` | Include step outputs in logs |
| `options.truncate_at` | `1000` | Max characters for any logged value |

---

## Environment variable substitution

Use `${VAR}` in config values. Ploston substitutes at startup:

```yaml
mcp:
  servers:
    github:
      env:
        GITHUB_TOKEN: "${GITHUB_TOKEN}"           # error if unset
        API_KEY: "${API_KEY:-default-value}"       # use default if unset
        DB_URL: "${DB_URL:?DB_URL must be set}"   # custom error message
```

| Syntax | Behaviour |
|--------|-----------|
| `${VAR}` | Required — error at startup if unset |
| `${VAR:-default}` | Optional — use default if unset |
| `${VAR:?message}` | Required — print message if unset |

---

## Example configs

### Minimal (bootstrap-managed, file not required)

When using `ploston bootstrap` + `ploston init --import`, no config file is needed. Config lives in Redis.

### Development (from source)

```yaml
workflows:
  directory: "./workflows"
  hot_reload: true

logging:
  level: DEBUG
  options:
    show_params: true
    show_results: true
```

### Production (self-hosted)

```yaml
workflows:
  directory: "/app/workflows"
  hot_reload: false

execution:
  max_concurrent: 50
  default_timeout: 600

logging:
  level: WARNING
  format: json

security:
  allowed_hosts:
    - "api.mycompany.com"
    - "internal.service.local"
```
