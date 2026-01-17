# CLI Reference

Complete reference for all AEL command-line interface commands.

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
ael serve [OPTIONS]
```

**Options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--no-watch` | false | Disable workflow file watching |
| `--mode MODE` | auto | Initial mode: `configuration` or `running` |
| `--transport TYPE` | stdio | Transport: `stdio` or `http` |
| `--port PORT` | 8080 | HTTP port (only with `--transport http`) |
| `--host HOST` | 0.0.0.0 | HTTP host (only with `--transport http`) |

**Examples:**

```bash
# Start with stdio transport (default)
ael serve

# Start with HTTP transport
ael serve --transport http --port 8080

# Start without file watching
ael serve --no-watch

# Start in configuration mode
ael serve --mode configuration
```

### `ael run`

Execute a workflow.

```bash
ael run WORKFLOW [OPTIONS]
```

**Arguments:**

| Argument | Description |
|----------|-------------|
| `WORKFLOW` | Path to workflow YAML file |

**Options:**

| Option | Description |
|--------|-------------|
| `-i, --input KEY=VALUE` | Input parameter (can be repeated) |
| `--input-file PATH` | JSON/YAML file with inputs |
| `-t, --timeout SECONDS` | Execution timeout |

**Examples:**

```bash
# Run with default inputs
ael run workflows/hello.yaml

# Run with input parameters
ael run workflows/hello.yaml -i name=Alice -i count=5

# Run with inputs from file
ael run workflows/process.yaml --input-file inputs.json

# Run with timeout
ael run workflows/long-task.yaml --timeout 300
```

### `ael validate`

Validate a workflow YAML file.

```bash
ael validate FILE [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--strict` | Treat warnings as errors |
| `--check-tools` | Verify tools exist (requires MCP connection) |

**Examples:**

```bash
# Basic validation
ael validate workflows/my-workflow.yaml

# Strict validation
ael validate workflows/my-workflow.yaml --strict

# Validate with tool checking
ael validate workflows/my-workflow.yaml --check-tools
```

### `ael version`

Show version information.

```bash
ael version
```

### `ael config show`

Show current configuration.

```bash
ael config show [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--section NAME` | Show specific section only |

**Examples:**

```bash
# Show all configuration
ael config show

# Show specific section
ael config show --section mcp
ael config show --section logging
```

### `ael workflows list`

List registered workflows.

```bash
ael workflows list
```

### `ael workflows show`

Show workflow details.

```bash
ael workflows show NAME
```

**Example:**

```bash
ael workflows show hello-world
```

### `ael tools list`

List available tools.

```bash
ael tools list [OPTIONS]
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
ael tools show NAME
```

### `ael tools refresh`

Refresh tool schemas from MCP servers.

```bash
ael tools refresh [OPTIONS]
```

**Options:**

| Option | Description |
|--------|-------------|
| `--server NAME` | Refresh specific server only |

### `ael api`

Start AEL REST API server.

```bash
ael api [OPTIONS]
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
ael api --port 8080

# Start with authentication
ael api --require-auth

# Start with rate limiting
ael api --rate-limit 100
```

