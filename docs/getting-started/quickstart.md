# Quickstart

Get Ploston running in 5 minutes.

## Prerequisites

- Python 3.12 or higher
- Terminal access

## Step 1: Install Ploston CLI

```bash
pip install ploston-cli
```

## Step 2: Create a Project Directory

```bash
mkdir my-ploston-project
cd my-ploston-project
```

Create `ael-config.yaml`:

```yaml
workflows:
  directory: "./workflows"

logging:
  level: INFO
```

## Step 4: Create Your First Workflow

Create the workflows directory and `workflows/hello.yaml`:

```bash
mkdir workflows
```

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

## Step 5: Validate the Workflow

```bash
ploston validate workflows/hello.yaml
```

Expected output:
```
✓ Workflow 'hello' is valid
```

## Step 6: Run the Workflow

```bash
ploston run workflows/hello.yaml
```

Expected output:
```
Hello, World!
```

Run with custom input:

```bash
ploston run workflows/hello.yaml -i name=Alice
```

Expected output:
```
Hello, Alice!
```

## Step 7: Start as MCP Server (Optional)

Start Ploston as an MCP server for AI agent integration:

```bash
# Start with stdio transport (for AI agent integration)
ploston serve

# Or start with HTTP transport (for REST access)
ploston serve --transport http --port 8080
```

The workflow is now available as an MCP tool named `workflow:hello`.

## What's Next?

You've successfully:

- ✅ Created a Ploston project
- ✅ Written a workflow
- ✅ Validated and executed it
- ✅ Started Ploston as an MCP server

### Continue Learning

- [First Workflow Tutorial](first-workflow.md) - Build a more complex workflow
- [Workflow Authoring Guide](../guides/workflow-authoring.md) - Deep dive into workflows
- [CLI Reference](../reference/cli-reference.md) - All available commands

### Try More Examples

- [Web Scraping Example](../examples/web-scraping.md)
- [Data Processing Example](../examples/data-processing.md)
- [API Integration Example](../examples/api-integration.md)
