# Tool Integration Guide

This guide explains how to integrate and use tools in Ploston workflows.

## Tool Types

Ploston supports three types of tools:

| Type | Description | Example |
|------|-------------|---------|
| **System** | Built-in tools | `python_exec` |
| **MCP** | Tools from MCP servers | `fs_read`, `kafka_publish` |
| **HTTP** | REST API endpoints | Custom API wrappers |

## Built-in Tools

### python_exec

Execute Python code in a secure sandbox.

```yaml
steps:
  - id: calculate
    tool: python_exec
    params:
      code: |
        import math
        result = math.sqrt(16)
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Python code to execute |
| `timeout` | int | No | Execution timeout (seconds) |

## MCP Server Tools

MCP (Model Context Protocol) servers provide tools via a standardized protocol.

### Configuring MCP Servers

Add MCP servers to your `ploston-config.yaml`:

```yaml
tools:
  mcp_servers:
    # Filesystem tools
    filesystem:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
    
    # GitHub tools
    github:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-github"]
      env:
        GITHUB_TOKEN: "${GITHUB_TOKEN}"
    
    # Custom MCP server
    native_tools:
      command: "python"
      args: ["-m", "native_tools.server"]
      env:
        WORKSPACE_DIR: "${WORKSPACE_DIR:-.}"
```

### Available MCP Servers

Popular MCP servers from the community:

| Server | Package | Tools |
|--------|---------|-------|
| Filesystem | `@modelcontextprotocol/server-filesystem` | File read/write/list |
| GitHub | `@modelcontextprotocol/server-github` | Repo, issues, PRs |
| Fetch | `@modelcontextprotocol/server-fetch` | HTTP requests |
| Postgres | `@modelcontextprotocol/server-postgres` | Database queries |

### Native Tools Server

Ploston includes a native tools MCP server with these tools:

| Category | Tools |
|----------|-------|
| Filesystem | `fs_read`, `fs_write`, `fs_list`, `fs_delete` |
| Network | `http_request`, `network_ping`, `network_dns_lookup` |
| Kafka | `kafka_publish`, `kafka_list_topics`, `kafka_consume` |
| Firecrawl | `firecrawl_search`, `firecrawl_map`, `firecrawl_extract` |
| Data | `data_validate`, `data_json_to_csv`, `data_csv_to_json` |
| Extraction | `extract_text`, `extract_structured` |
| ML | `ml_embed_text`, `ml_text_similarity` |

## Using Tools in Workflows

### Tool Steps

Call a tool directly:

```yaml
steps:
  - id: read_file
    tool: fs_read
    params:
      path: "data/input.json"
```

### Tool Calls from Code Steps

Call tools from Python code:

```yaml
steps:
  - id: process
    code: |
      # Read file using tool
      content = call_tool("fs_read", {"path": "data/input.json"})
      
      # Process the content
      data = json.loads(content["content"])
      result = {"count": len(data)}
```

### Conditional Tool Calls

```yaml
steps:
  - id: check
    code: |
      result = {"needs_fetch": True}
  
  - id: fetch_data
    tool: http_request
    params:
      url: "https://api.example.com/data"
    when: "{{ steps.check.output.needs_fetch }}"
```

## Listing Available Tools

Use the CLI to see available tools:

```bash
# List all tools
ploston tools list

# Filter by source
ploston tools list --source mcp
ploston tools list --source system

# Show tool details
ploston tools show fs_read
```

Example output:

```
NAME              SOURCE   SERVER        STATUS      DESCRIPTION
─────────────────────────────────────────────────────────────────
fs_read           mcp      native_tools  available   Read file content
fs_write          mcp      native_tools  available   Write file content
kafka_publish     mcp      native_tools  available   Publish to Kafka
python_exec       system   -             available   Execute Python code

Total: 4 tools (4 available)
```

## Tool Input/Output

### Input Schema

Tools define their input parameters via JSON Schema:

```bash
ploston tools show fs_read
```

```yaml
Tool: fs_read
Description: Read file content from workspace

Input Schema:
  type: object
  properties:
    path:
      type: string
      description: File path relative to workspace
    encoding:
      type: string
      default: utf-8
  required:
    - path
```

### Output Handling

Tool outputs are available in subsequent steps:

```yaml
steps:
  - id: read
    tool: fs_read
    params:
      path: "config.json"
  
  - id: process
    code: |
      # Access tool output
      file_content = "{{ steps.read.output.content }}"
      result = json.loads(file_content)
```

## Error Handling

### Tool Errors

Handle tool failures gracefully:

```yaml
steps:
  - id: fetch
    tool: http_request
    params:
      url: "https://api.example.com/data"
    on_error: continue  # Don't fail workflow
  
  - id: handle_error
    code: |
      if "{{ steps.fetch.error }}":
        result = {"status": "failed", "fallback": True}
      else:
        result = "{{ steps.fetch.output }}"
```

### Retry Configuration

Configure retries in workflow:

```yaml
steps:
  - id: unreliable_call
    tool: http_request
    params:
      url: "https://flaky-api.example.com"
    retry:
      max_attempts: 3
      backoff: exponential
```

## Creating Custom MCP Servers

Create your own MCP server using FastMCP:

```python
# my_tools/server.py
from fastmcp import FastMCP

mcp = FastMCP("my-tools")

@mcp.tool()
def my_custom_tool(param1: str, param2: int = 10) -> dict:
    """My custom tool description."""
    return {"result": f"{param1} x {param2}"}

if __name__ == "__main__":
    mcp.run()
```

Configure in `ploston-config.yaml`:

```yaml
tools:
  mcp_servers:
    my_tools:
      command: "python"
      args: ["-m", "my_tools.server"]
```

## Best Practices

1. **Use descriptive tool names** - Makes workflows readable
2. **Handle errors** - Use `on_error` and fallback logic
3. **Set timeouts** - Prevent hanging on slow tools
4. **Validate inputs** - Check parameters before tool calls
5. **Log tool calls** - Enable tool logging for debugging

```yaml
logging:
  components:
    tool: true
  options:
    show_params: true
    show_results: true
```
