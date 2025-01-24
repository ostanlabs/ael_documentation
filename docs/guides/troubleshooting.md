# Troubleshooting Guide

Common issues and solutions when using Ploston.

## Installation Issues

### "Command not found: ploston"

**Problem:** After running `uv sync`, the `ploston` command is not found.

**Solution:** Use one of these methods to run the CLI:

```bash
# Option 1: Use uv run (recommended)
ploston --help

# Option 2: Activate virtual environment
source .venv/bin/activate
ploston --help

# Option 3: Direct path
.venv/bin/ploston --help
```

### "Python 3.12+ required"

**Problem:** Ploston requires Python 3.12 or higher.

**Solution:**

```bash
# Check your Python version
python --version

# Install Python 3.12 using pyenv
pyenv install 3.12
pyenv local 3.12

# Or use uv to manage Python
uv python install 3.12
```

### "uv: command not found"

**Problem:** uv package manager is not installed.

**Solution:**

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Add to PATH (if needed)
export PATH="$HOME/.local/bin:$PATH"

# Verify installation
uv --version
```

### "uv sync failed"

**Problem:** Dependencies failed to install.

**Solution:**

```bash
# Clear cache and retry
uv cache clean
uv sync

# If still failing, check Python version
uv python list
uv python install 3.12
uv sync
```

## Configuration Issues

### "No config found"

**Problem:** Ploston cannot find configuration file.

**Solution:**

Ploston searches for config in this order:
1. `AEL_CONFIG_PATH` environment variable
2. `./ael-config.yaml` (current directory)
3. `~/.ploston/config.yaml` (home directory)

Create a minimal config:

```bash
cat > ael-config.yaml << EOF
workflows:
  directory: "./workflows"
EOF

mkdir -p workflows
```

### "Configuration mode" instead of "Running mode"

**Problem:** Ploston starts in configuration mode when you expect running mode.

**Cause:** No valid config file found.

**Solution:**

```bash
# Check if config exists
ls -la ael-config.yaml

# Verify config is valid YAML
python -c "import yaml; yaml.safe_load(open('ael-config.yaml'))"

# Force running mode (will fail if no config)
ploston serve --mode running
```

### "Invalid configuration"

**Problem:** Config file has syntax or validation errors.

**Solution:**

```bash
# Check YAML syntax
python -c "import yaml; yaml.safe_load(open('ael-config.yaml'))"

# Validate with Ploston
ploston config show
```

Common config errors:
- Indentation issues (use spaces, not tabs)
- Missing required fields
- Invalid enum values

## Workflow Issues

### "Workflow validation failed"

**Problem:** Workflow YAML has errors.

**Solution:**

```bash
# Validate workflow
ploston validate workflows/my-workflow.yaml
```

Check for common issues:
- Missing `name` field
- Invalid step IDs (must be alphanumeric)
- Missing `code` or `tool` in steps
- Invalid Jinja2 syntax in templates

### "Tool not found"

**Problem:** Workflow references a tool that does not exist.

**Solution:**

```bash
# List available tools
ploston tools list

# Check if MCP server is configured
ploston config show --section tools

# Refresh tools from MCP servers
ploston tools refresh
```

### "Step timeout"

**Problem:** Step execution exceeded timeout.

**Solution:**

Increase timeout in workflow:

```yaml
steps:
  - id: slow_step
    tool: http_request
    params:
      url: "https://slow-api.example.com"
    timeout: 120  # seconds
```

Or in config:

```yaml
execution:
  default_timeout: 300
```

## MCP Server Issues

### "MCP server connection failed"

**Problem:** Cannot connect to MCP server.

**Solution:**

```bash
# Check if command exists
which npx  # for npm-based servers
which python  # for Python servers

# Test server manually
npx -y @modelcontextprotocol/server-filesystem /tmp

# Check environment variables
echo $GITHUB_TOKEN  # for GitHub server
```

### "MCP server timeout"

**Problem:** MCP server takes too long to respond.

**Solution:**

Check server logs and increase timeout:

```yaml
tools:
  mcp_servers:
    slow_server:
      command: "python"
      args: ["-m", "slow_server"]
      timeout: 60  # seconds
```

### "Tool schema mismatch"

**Problem:** Tool parameters do not match expected schema.

**Solution:**

```bash
# Check tool schema
ploston tools show tool_name

# Verify your parameters match the schema
```

## Runtime Issues

### "Sandbox execution error"

**Problem:** Python code in code step failed.

**Solution:**

Check for:
- Syntax errors in code
- Disallowed imports
- Timeout exceeded

```yaml
# Enable debug logging
logging:
  level: DEBUG
  components:
    sandbox: true
```

### "Memory limit exceeded"

**Problem:** Code step used too much memory.

**Solution:**

Increase memory limit in config:

```yaml
python_exec:
  max_memory: 1073741824  # 1GB
```

Or optimize your code to use less memory.

### "Import not allowed"

**Problem:** Code step tried to import a disallowed module.

**Solution:**

Add the import to allowed list:

```yaml
python_exec:
  allowed_imports:
    - json
    - re
    - datetime
    - math
    - your_module  # Add your module
```

## Docker Issues

### "Container will not start"

**Problem:** Docker container fails to start.

**Solution:**

```bash
# Check logs
docker compose logs ploston

# Verify config mount
docker compose config

# Rebuild image
docker compose build --no-cache
docker compose up -d
```

### "Volume mount issues"

**Problem:** Files not accessible in container.

**Solution:**

```bash
# Check volume mounts
docker compose config | grep volumes

# Verify file permissions
ls -la workflows/
chmod -R 755 workflows/
```

## Getting Help

If you are still stuck:

1. **Check logs:** Enable debug logging
   ```yaml
   logging:
     level: DEBUG
   ```

2. **Search issues:** [GitHub Issues](https://github.com/ostanlabs/ploston/issues)

3. **Ask for help:** [GitHub Discussions](https://github.com/ostanlabs/ploston/discussions)

4. **Report a bug:** Include:
   - Ploston version (`ploston version`)
   - Python version (`python --version`)
   - OS and version
   - Full error message
   - Minimal reproduction steps
