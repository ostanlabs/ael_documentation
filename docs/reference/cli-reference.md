# CLI Reference

Complete reference for all Ploston command-line interface commands.

## Running the CLI

After installing Ploston CLI (`pip install ploston-cli`), the `ploston` command is available:

```bash
ploston <command>
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

### `ploston serve`

Start Ploston as an MCP server.

```bash
ploston serve [OPTIONS]
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
ploston serve

# Start with HTTP transport (for REST access)
ploston serve --transport http --port 8080

# Start without file watching
ploston serve --no-watch

# Force configuration mode (for initial setup)
ploston serve --mode configuration

# Force running mode (fails if no config)
ploston serve --mode running
```

**Startup Output:**

When config is found (running mode):
```
[Ploston] Config loaded from: ./ploston-config.yaml
[Ploston] Mode: running
[Ploston] MCP servers: filesystem, github (2)
[Ploston] Workflows: 5 registered
```

When no config found (configuration mode):
```
[Ploston] No config found (searched: ./ploston-config.yaml, ~/.ploston/config.yaml)
[Ploston] Mode: configuration
[Ploston] Use config tools to set up Ploston
```

### `ploston run`

Execute a workflow.

```bash
ploston run WORKFLOW [OPTIONS]
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
ploston run workflows/hello.yaml

# Run with input parameters
ploston run workflows/hello.yaml -i name=Alice -i count=5

# Run with inputs from file
ploston run workflows/process.yaml --input-file inputs.json

# Run with timeout
ploston run workflows/long-task.yaml --timeout 300
```

### `ploston validate`

Validate a workflow YAML file.

```bash
ploston validate FILE [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--strict` | Treat warnings as errors |
| `--check-tools` | Verify tools exist (requires MCP connection) |

**Examples:**

```bash
# Basic validation
ploston validate workflows/my-workflow.yaml

# Strict validation
ploston validate workflows/my-workflow.yaml --strict
```

### `ploston version`

Show version information.

```bash
ploston version
```

### `ploston api`

Start Ploston REST API server.

```bash
ploston api [OPTIONS]
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
ploston api --port 8080

# Start with authentication
ploston api --require-auth

# Start with rate limiting
ploston api --rate-limit 100

# With execution history storage
ploston api --db ./ploston-executions.db
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

### `ploston config show`

Show current configuration.

```bash
ploston config show [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--section NAME` | Show specific section only |

**Examples:**

```bash
# Show all configuration
ploston config show

# Show specific section
ploston config show --section mcp
ploston config show --section logging

# Output as JSON
ploston --json config show
```

### `ploston workflows list`

List registered workflows.

```bash
ploston workflows list
```

### `ploston workflows show`

Show workflow details.

```bash
ploston workflows show NAME
```

**Example:**

```bash
ploston workflows show hello-world
```

### `ploston tools list`

List available tools.

```bash
ploston tools list [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--source TYPE` | Filter by source: `mcp` or `system` |
| `--server NAME` | Filter by MCP server name |
| `--status STATUS` | Filter by status: `available` or `unavailable` |

### `ploston tools show`

Show tool details.

```bash
ploston tools show NAME
```

### `ploston tools refresh`

Refresh tool schemas from MCP servers.

```bash
ploston tools refresh [OPTIONS]
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
