# Installation

This guide covers installing Ploston on your system.

## Requirements

- **Python**: 3.12 or higher
- **Git**: For cloning the repository
- **OS**: macOS, Linux, or Windows

## Installation Methods

### From Source (Recommended)

Clone the repository and install dependencies:

```bash
# Clone the repository
git clone https://github.com/ostanlabs/agent-execution-layer.git
cd agent-execution-layer

# Install uv (fast Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync
```

### Using Docker

Build and run Ploston using Docker Compose:

```bash
# Clone the repository
git clone https://github.com/ostanlabs/agent-execution-layer.git
cd agent-execution-layer

# Build and start services
docker compose up -d

# View logs
docker compose logs -f ploston
```

Or build the image manually:

```bash
docker build -t ploston:latest .
docker run -p 8080:8080 ploston:latest
```

## Running the CLI

After installation from source, you have several ways to run the Ploston CLI:

### Option 1: Using uv run (Recommended)

```bash
# No activation needed - uv handles the virtual environment
ploston --help
ploston version
ploston serve
```

### Option 2: Activate Virtual Environment

```bash
# Activate the virtual environment first
source .venv/bin/activate  # Linux/macOS
# or
.venv\Scripts\activate     # Windows

# Then run ploston directly
ploston --help
ploston version
ploston serve
```

### Option 3: Direct Path

```bash
# Run directly from the virtual environment
.venv/bin/ploston --help       # Linux/macOS
.venv\Scripts\ploston.exe      # Windows
```

## Configuration Modes

Ploston operates in two modes:

### Running Mode

When Ploston finds a valid configuration file, it starts in **running mode** with full functionality:

```bash
ploston serve
# Output:
# [Ploston] Config loaded from: ./ploston-config.yaml
# [Ploston] Mode: running
# [Ploston] Workflows: 5 registered
```

### Configuration Mode

When no configuration file exists, Ploston starts in **configuration mode**:

```bash
ploston serve
# Output:
# [Ploston] No config found (searched: ./ploston-config.yaml, ~/.ploston/config.yaml)
# [Ploston] Mode: configuration
# [Ploston] Use config tools to set up Ploston
```

In configuration mode, Ploston exposes special MCP tools for setup:

| Tool | Description |
|------|-------------|
| `config_get` | Read current configuration |
| `config_set` | Stage configuration changes |
| `config_validate` | Validate staged configuration |
| `config_done` | Apply config and switch to running mode |

You can also force a specific mode:

```bash
# Force configuration mode (even if config exists)
ploston serve --mode configuration

# Force running mode (fails if no config)
ploston serve --mode running
```

## Creating a Configuration File

Create `ploston-config.yaml` in your project directory:

```yaml
workflows:
  directory: "./workflows"

logging:
  level: INFO
```

Create the workflows directory:

```bash
mkdir -p workflows
```

Verify the setup:

```bash
ploston config show
```

## Optional: MCP Server Setup

To use external tools, configure MCP servers:

```yaml
tools:
  mcp_servers:
    filesystem:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
```

## Troubleshooting

### Python Version Error

```bash
# Check your Python version
python --version

# Use pyenv to install Python 3.12
pyenv install 3.12
pyenv local 3.12
```

### uv Not Found

```bash
# Reinstall uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Add to PATH (if needed)
export PATH="$HOME/.local/bin:$PATH"
```

### Command Not Found After uv sync

Make sure you're using one of the CLI invocation methods above. The `ploston` command is only available:
- When using `ploston`
- After activating the virtual environment
- Via direct path `.venv/bin/ploston`

## Next Steps

- [Quickstart Guide](quickstart.md) - Run your first workflow
- [First Workflow Tutorial](first-workflow.md) - Create a custom workflow
- [Configuration Reference](../reference/config-reference.md) - All config options
