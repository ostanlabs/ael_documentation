# Troubleshooting Guide

Solutions to common AEL issues.

## Installation Issues

### Command Not Found

```
zsh: command not found: ael
```

**Solutions:**
1. Ensure AEL is installed: `pip install ael`
2. Check PATH includes pip bin directory
3. Try: `python -m ael --version`

### Import Errors

```
ModuleNotFoundError: No module named 'ael'
```

**Solutions:**
1. Verify installation: `pip show ael`
2. Check Python version (requires 3.11+)
3. Reinstall: `pip install --force-reinstall ael`

## Configuration Issues

### Config File Not Found

```
Error: Configuration file not found
```

**Solutions:**
1. Create config: `ael config show > ael-config.yaml`
2. Specify path: `ael --config /path/to/config.yaml`
3. Check current directory for `ael-config.yaml`

### Invalid Configuration

```
Error: CONFIG_INVALID - Invalid configuration
```

**Solutions:**
1. Validate YAML syntax
2. Check required fields
3. View current config: `ael config show`
4. See [Configuration Reference](../reference/config-reference.md)

## Workflow Issues

### Workflow Not Found

```
Error: WORKFLOW_NOT_FOUND - Workflow 'my-workflow' not found
```

**Solutions:**
1. List workflows: `ael workflows list`
2. Check workflows directory in config
3. Verify file exists and has `.yaml` extension
4. Check workflow `name` field matches

### Validation Errors

```
Error: Workflow validation failed
```

**Solutions:**
1. Validate workflow: `ael validate workflow.yaml`
2. Check required fields: `name`, `version`, `steps`
3. Verify step IDs are unique
4. Check `depends_on` references valid steps

### Circular Dependency

```
Error: CIRCULAR_DEPENDENCY - Circular dependency detected
```

**Solutions:**
1. Review `depends_on` relationships
2. Draw dependency graph
3. Remove circular references
4. Use sequential steps without `depends_on`

## Tool Issues

### Tool Unavailable

```
Error: TOOL_UNAVAILABLE - Tool 'tool_name' is unavailable
```

**Solutions:**
1. List tools: `ael tools list`
2. Check MCP server is configured
3. Refresh tools: `ael tools refresh`
4. Verify MCP server is running

### MCP Connection Failed

```
Error: MCP_CONNECTION_FAILED - Failed to connect to MCP server
```

**Solutions:**
1. Check server command: `npx -y @package/server`
2. Verify package is installed
3. Check environment variables
4. Review server logs

### Tool Timeout

```
Error: TOOL_TIMEOUT - Tool timed out after 30s
```

**Solutions:**
1. Increase step timeout:
   ```yaml
   steps:
     - id: slow_step
       tool: slow_tool
       timeout: 120
   ```
2. Check tool is not stuck
3. Verify network connectivity

## Code Step Issues

### Syntax Error

```
Error: CODE_SYNTAX - Syntax error in code block
```

**Solutions:**
1. Check Python syntax
2. Verify indentation (use spaces, not tabs)
3. Test code locally first
4. Check for unclosed brackets/quotes

### Security Violation

```
Error: CODE_SECURITY - Security violation in code block
```

**Solutions:**
1. Remove forbidden imports (os, sys, subprocess)
2. Don't use eval(), exec(), open()
3. See [Code Steps Guide](code-steps.md) for allowed imports

### Import Not Allowed

```
Error: Import 'requests' not allowed
```

**Solutions:**
1. Use allowed imports only (json, re, datetime, etc.)
2. Use tool steps for HTTP requests
3. See [Code Steps Guide](code-steps.md)

### Result Not Set

```
Error: Code step did not set 'result' variable
```

**Solutions:**
1. Ensure code sets `result`:
   ```yaml
   code: |
     data = process()
     result = data  # Required!
   ```

## Execution Issues

### Workflow Timeout

```
Error: WORKFLOW_TIMEOUT - Workflow timed out after 300s
```

**Solutions:**
1. Increase workflow timeout in config
2. Optimize slow steps
3. Check for stuck tools
4. Break into smaller workflows

### Template Error

```
Error: TEMPLATE_ERROR - Template rendering failed
```

**Solutions:**
1. Check Jinja2 syntax: `{{ variable }}`
2. Verify variable names exist
3. Check for typos in step/input names
4. Use `| default('value')` for optional values

## Server Issues

### Port Already in Use

```
Error: Address already in use (port 8080)
```

**Solutions:**
1. Use different port: `ael serve --port 8081`
2. Find process: `lsof -i :8080`
3. Kill process: `kill -9 <PID>`

### API Authentication Failed

```
Error: 401 Unauthorized
```

**Solutions:**
1. Check API key is configured
2. Verify key in request header
3. Check security config in `ael-config.yaml`

## Getting Help

### Debug Logging

Enable debug logs:

```bash
ael --log-level DEBUG run workflow.yaml
```

Or in config:

```yaml
logging:
  level: DEBUG
```

### Check Version

```bash
ael version
```

### Report Issues

1. Gather logs with DEBUG level
2. Include workflow YAML (redact secrets)
3. Include error message
4. Open issue on GitHub

## Related

- [Error Codes Reference](../reference/error-codes.md)
- [Configuration Reference](../reference/config-reference.md)
- [Tool Integration Guide](tool-integration.md)

