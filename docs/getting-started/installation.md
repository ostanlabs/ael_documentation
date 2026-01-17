# Installation

This guide covers installing AEL on your system.

## Requirements

- **Python**: 3.12 or higher
- **pip**: Latest version recommended
- **OS**: macOS, Linux, or Windows

## Installation Methods

### Using pip (Recommended)

```bash
pip install ael
```

Verify the installation:

```bash
ael version
```

Expected output:
```
AEL version 0.1.0
```

### Using Docker

Pull the official Docker image:

```bash
docker pull ghcr.io/ostanlabs/ael:latest
```

Run AEL in a container:

```bash
docker run -it --rm \
  -v $(pwd)/workflows:/app/workflows \
  -v $(pwd)/ael-config.yaml:/app/ael-config.yaml \
  ghcr.io/ostanlabs/ael:latest serve
```

### From Source

Clone the repository:

```bash
git clone https://github.com/ostanlabs/agent-execution-layer.git
cd agent-execution-layer
```

Install with uv (recommended for development):

```bash
# Install uv if not already installed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies and create virtual environment
uv sync

# Verify installation
uv run ael version
```

Or with pip:

```bash
pip install -e .
ael version
```

## Post-Installation Setup

### 1. Create Configuration File

Create `ael-config.yaml` in your project directory:

```yaml
mcp:
  servers: {}

workflows:
  directory: "./workflows"

logging:
  level: INFO
```

### 2. Create Workflows Directory

```bash
mkdir -p workflows
```

### 3. Verify Setup

```bash
ael config show
```

This should display your configuration without errors.

## Optional: MCP Server Setup

To use external tools, configure MCP servers in your config:

```yaml
mcp:
  servers:
    filesystem:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
```

## Troubleshooting

### Python Version Error

If you see "Python 3.12+ required":

```bash
# Check your Python version
python --version

# Use pyenv to install Python 3.12
pyenv install 3.12
pyenv local 3.12
```

### Permission Denied

On Linux/macOS, you may need to use `pip install --user`:

```bash
pip install --user ael
```

Or use a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
pip install ael
```

### Command Not Found

Ensure the pip scripts directory is in your PATH:

```bash
# Linux/macOS
export PATH="$HOME/.local/bin:$PATH"

# Windows (PowerShell)
$env:PATH += ";$env:APPDATA\Python\Python312\Scripts"
```

## Next Steps

- [Quickstart Guide](quickstart.md) - Run your first workflow
- [First Workflow Tutorial](first-workflow.md) - Create a custom workflow
- [Configuration Reference](../reference/config-reference.md) - All config options

