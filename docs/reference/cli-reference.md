# CLI Reference

Complete reference for all AEL command-line interface commands.

## Running the CLI

After installing AEL from source, use one of these methods:

```bash
# Recommended: Use uv run
uv run ael <command>

# Or activate venv first
source .venv/bin/activate
ael <command>

# Or use direct path
.venv/bin/ael <command>
```

## Global Options

These options apply to all commands:

| Option | Description |
|--------|-------------|
| `-c, --config PATH` | Path to configuration file |
| `-v, --verbose` | Increase verbosity (can be repeated: -vv) |
| `-q, --quiet` | Suppress output |
| `--json` | Output as JSON |
| `--help` | Show help message |

## Commands

### `ael serve`

Start AEL as an MCP server.

```bash
uv run ael serve [OPTIONS]
```

**Options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--no-watch` | false | Disable workflow file watching |
| `--mode MODE` | auto | Startup mode: `configuration` or `running` |
| `--transport TYPE` | stdio | Transport: `stdio` or `http` |
| `--port PORT` | 8080 | HTTP port (only with `--transport http`) |
| `--host HOST` | 0.0.0.0 | HTTP host (only with `--transport http`) |

**Examples:**

```bash
# Start with stdio transport (default, for AI agent integration)
uv run ael serve

# Start with HTTP transport (for REST access)
uv run ael serve --transport http --port 8080

# Start without file watching
uv run ael serve --no-watch

# Force configuration mode (for initial setup)
uv run ael serve --mode configuration

# Force running mode (fails if no config)
uv run ael serve --mode running
```

**Startup Output:**

When config is found (running mode):
```
[AEL] Config loaded from: ./ael-config.yaml
[AEL] Mode: running
[AEL] MCP servers: filesystem, github (2)
[AEL] Workflows: 5 registered
```

When no config found (configuration mode):
```
[AEL] No config found (searched: ./ael-config.yaml, ~/.ael/config.yaml)
[AEL] Mode: configuration
[AEL] Use config tools to set up AEL
```

### `ael run`

Execute a workflow.

```bash
uv run ael run WORKFLOW [OPTIONS]
```

**Arguments:**

| Argument | Description |
|----------|-------------|
| `WORKFLOW` | Path to workflow YAML file or workflow name |

**Options:**

| Option | Description |
|--------|-------------|
| `-i, --input KEY=VALUE` | Input parameter (can be repeated) |
| `--input-file PATH` | JSON/YAML file with inputs |
| `-t, --timeout SECONDS` | Execution timeout |

**Examples:**

```bash
# Run with default inputs
uv run ael run workflows/hello.yaml

# Run with input parameters
uv run ael run workflows/hello.yaml -i name=Alice -i count=5

# Run with inputs from file
uv run ael run workflows/process.yaml --input-file inputs.json

# Run with timeout
uv run ael run workflows/long-task.yaml --timeout 300
```

### `ael validate`

Validate a workflow YAML file.

```bash
uv run ael validate FILE [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--strict` | Treat warnings as errors |
| `--check-tools` | Verify tools exist (requires MCP connection) |

**Examples:**

```bash
# Basic validation
uv run ael validate workflows/my-workflow.yaml

# Strict validation
uv run ael validate workflows/my-workflow.yaml --strict
```

### `ael version`

Show version information.

```bash
uv run ael version
```

### `ael api`

Start AEL REST API server.

```bash
uv run ael api [OPTIONS]
```

**Options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--host HOST` | 0.0.0.0 | API host |
| `--port PORT` | 8080 | API port |
| `--prefix PREFIX` | /api/v1 | API path prefix |
| `--no-docs` | false | Disable OpenAPI docs |
| `--require-auth` | false | Require API key authentication |
| `--rate-limit N` | 0 | Requests per minute (0=disabled) |
| `--db PATH` | - | SQLite database path |

**Examples:**

```bash
# Start API server
uv run ael api --port 8080

# Start with authentication
uv run ael api --require-auth

# Start with rate limiting
uv run ael api --rate-limit 100

# With execution history storage
uv run ael api --db ./ael-executions.db
```

**API Endpoints:**

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/workflows` | GET | List workflows |
| `/api/v1/workflows/{name}` | GET | Get workflow details |
| `/api/v1/workflows/{name}/execute` | POST | Execute workflow |
| `/api/v1/tools` | GET | List tools |
| `/api/v1/health` | GET | Health check |
| `/docs` | GET | OpenAPI documentation |

### `ael config show`

Show current configuration.

```bash
uv run ael config show [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--section NAME` | Show specific section only |

**Examples:**

```bash
# Show all configuration
uv run ael config show

# Show specific section
uv run ael config show --section mcp
uv run ael config show --section logging

# Output as JSON
uv run ael --json config show
```

### `ael workflows list`

List registered workflows.

```bash
uv run ael workflows list
```

### `ael workflows show`

Show workflow details.

```bash
uv run ael workflows show NAME
```

**Example:**

```bash
uv run ael workflows show hello-world
```

### `ael tools list`

List available tools.

```bash
uv run ael tools list [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--source TYPE` | Filter by source: `mcp` or `system` |
| `--server NAME` | Filter by MCP server name |
| `--status STATUS` | Filter by status: `available` or `unavailable` |

### `ael tools show`

Show tool details.

```bash
uv run ael tools show NAME
```

### `ael tools refresh`

Refresh tool schemas from MCP servers.

```bash
uv run ael tools refresh [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--server NAME` | Refresh specific server only |

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error (validation failed, config not found, etc.) |
