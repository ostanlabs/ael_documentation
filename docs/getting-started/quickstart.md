# Quickstart

Get AEL running in 5 minutes.

## Prerequisites

- AEL installed from source ([Installation Guide](installation.md))
- Terminal access

## Step 1: Navigate to Project

```bash
cd agent-execution-layer
```

## Step 2: Create Configuration

Create `ael-config.yaml`:

```yaml
workflows:
  directory: "./workflows"

logging:
  level: INFO
```

## Step 3: Create Your First Workflow

Create `workflows/hello.yaml`:

```yaml
name: hello
version: "1.0"
description: A simple hello world workflow

inputs:
  name:
    type: string
    default: "World"

steps:
  - id: greet
    code: |
      name = "{{ inputs.name }}"
      result = f"Hello, {name}!"

output: "{{ steps.greet.output }}"
```

## Step 4: Validate the Workflow

```bash
uv run ael validate workflows/hello.yaml
```

Expected output:
```
✓ Workflow 'hello' is valid
```

## Step 5: Run the Workflow

```bash
uv run ael run workflows/hello.yaml
```

Expected output:
```
Hello, World!
```

Run with custom input:

```bash
uv run ael run workflows/hello.yaml -i name=Alice
```

Expected output:
```
Hello, Alice!
```

## Step 6: Start as MCP Server (Optional)

Start AEL as an MCP server for AI agent integration:

```bash
# Start with stdio transport (for AI agent integration)
uv run ael serve

# Or start with HTTP transport (for REST access)
uv run ael serve --transport http --port 8080
```

The workflow is now available as an MCP tool named `workflow:hello`.

## What's Next?

You've successfully:

- ✅ Created an AEL project
- ✅ Written a workflow
- ✅ Validated and executed it
- ✅ Started AEL as an MCP server

### Continue Learning

- [First Workflow Tutorial](first-workflow.md) - Build a more complex workflow
- [Workflow Authoring Guide](../guides/workflow-authoring.md) - Deep dive into workflows
- [CLI Reference](../reference/cli-reference.md) - All available commands

### Try More Examples

- [Web Scraping Example](../examples/web-scraping.md)
- [Data Processing Example](../examples/data-processing.md)
- [API Integration Example](../examples/api-integration.md)
