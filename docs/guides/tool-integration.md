# Tool Integration Guide

Guide to integrating MCP tools with AEL workflows.

## Overview

AEL connects to MCP (Model Context Protocol) servers to access tools. Tools can:

- Read/write files
- Make HTTP requests
- Query databases
- Execute system commands
- And more...

## Configuring MCP Servers

### Stdio Transport (Local)

For locally-spawned MCP servers:

```yaml
# ael-config.yaml
tools:
  mcp_servers:
    filesystem:
      command: npx
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
      timeout: 30
```

### HTTP Transport (Remote)

For remote MCP servers:

```yaml
tools:
  mcp_servers:
    remote-tools:
      url: "http://mcp-server.example.com:8080/mcp"
      transport: http
      timeout: 30
```

### Environment Variables

Pass environment variables to MCP servers:

```yaml
tools:
  mcp_servers:
    api-tools:
      command: npx
      args: ["-y", "@example/mcp-server-api"]
      env:
        API_KEY: "${MY_API_KEY}"
        API_URL: "https://api.example.com"
```

## Discovering Tools

### List Available Tools

```bash
ael tools list
```

Output:
```
Available Tools:
  filesystem/read_file     - Read file contents
  filesystem/write_file    - Write file contents
  filesystem/list_directory - List directory contents
  fetch/fetch              - Fetch URL content
```

### View Tool Details

```bash
ael tools show filesystem/read_file
```

Output:
```
Tool: filesystem/read_file
Description: Read the contents of a file

Parameters:
  path (string, required): Path to the file to read

Example:
  params:
    path: "/tmp/data.txt"
```

### Refresh Tools

After adding new MCP servers:

```bash
ael tools refresh
```

## Using Tools in Workflows

### Basic Tool Step

```yaml
steps:
  - id: read_config
    tool: filesystem/read_file
    params:
      path: "/etc/app/config.json"
```

### With Dynamic Parameters

```yaml
steps:
  - id: fetch_data
    tool: fetch/fetch
    params:
      url: "{{ inputs.api_url }}/data"
      headers:
        Authorization: "Bearer {{ inputs.token }}"
```

### With Error Handling

```yaml
steps:
  - id: write_result
    tool: filesystem/write_file
    params:
      path: "{{ inputs.output_path }}"
      content: "{{ steps.process.output }}"
    on_error: retry
    retry:
      max_attempts: 3
      initial_delay: 1.0
```

## Calling Tools from Code

Use `tools.call()` in code steps:

```yaml
steps:
  - id: dynamic_fetch
    code: |
      urls = {{ inputs.urls }}
      results = []
      
      for url in urls:
          response = await tools.call("fetch/fetch", {
              "url": url
          })
          results.append(response)
      
      result = results
```

### Tool Call Limits

Default: 10 calls per code step. Configure in `ael-config.yaml`:

```yaml
python_exec:
  max_tool_calls: 50
```

## Common MCP Servers

### Filesystem

```yaml
tools:
  mcp_servers:
    filesystem:
      command: npx
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"]
```

Tools: `read_file`, `write_file`, `list_directory`, `create_directory`

### Fetch (HTTP)

```yaml
tools:
  mcp_servers:
    fetch:
      command: npx
      args: ["-y", "@anthropic/mcp-server-fetch"]
```

Tools: `fetch` (GET/POST requests)

### SQLite

```yaml
tools:
  mcp_servers:
    sqlite:
      command: npx
      args: ["-y", "@anthropic/mcp-server-sqlite", "/path/to/db.sqlite"]
```

Tools: `query`, `execute`

## Troubleshooting

### Tool Not Found

```
Error: TOOL_UNAVAILABLE - Tool 'unknown_tool' is unavailable
```

**Solutions:**
1. Check tool name: `ael tools list`
2. Verify MCP server is configured
3. Refresh tools: `ael tools refresh`

### Connection Failed

```
Error: MCP_CONNECTION_FAILED - Failed to connect to MCP server
```

**Solutions:**
1. Check MCP server command is correct
2. Verify required packages are installed
3. Check server logs for errors
4. Test server manually: `npx -y @package/server`

### Tool Timeout

```
Error: TOOL_TIMEOUT - Tool 'slow_tool' timed out after 30s
```

**Solutions:**
1. Increase timeout in config or step
2. Check if tool is stuck
3. Verify network connectivity

## Best Practices

1. **Use specific tool names** - Include server prefix: `filesystem/read_file`
2. **Set appropriate timeouts** - Different tools need different timeouts
3. **Handle errors** - Use `on_error` and `retry` for unreliable tools
4. **Limit tool calls** - Don't call tools in tight loops
5. **Validate parameters** - Check inputs before calling tools

## Related

- [Configuration Reference](../reference/config-reference.md)
- [Error Codes Reference](../reference/error-codes.md)
- [Troubleshooting Guide](troubleshooting.md)

