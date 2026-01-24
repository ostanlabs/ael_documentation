# Installation

This guide covers installing Ploston on your system.

## Requirements

- **Python**: 3.12 or higher
- **OS**: macOS, Linux, or Windows

## Installation Methods

### From PyPI (Recommended)

Install the Ploston CLI from PyPI:

```bash
pip install ploston-cli
```

Verify the installation:

```bash
ploston version
```

### Using Docker

Run Ploston using the official Docker image:

```bash
# Pull and run the latest image
docker run -p 8082:8082 ostanlabs/ploston:latest

# Or use a specific version
docker run -p 8082:8082 ostanlabs/ploston:dev
```

For development with Docker Compose, create a `docker-compose.yml`:

```yaml
services:
  ploston:
    image: ostanlabs/ploston:latest
    ports:
      - "8082:8082"
    volumes:
      - ./workflows:/app/workflows
      - ./ploston-config.yaml:/app/ploston-config.yaml
    environment:
      - PLOSTON_LOG_LEVEL=INFO
```

Then run:

```bash
docker compose up -d
docker compose logs -f ploston
```

### From Source (Development)

For contributing or development:

```bash
# Clone the repository
git clone https://github.com/ostanlabs/ploston.git
cd ploston

# Install uv (fast Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync

# Run the CLI
uv run ploston --help
```

## Running the CLI

After installing from PyPI, the `ploston` command is available globally:

```bash
ploston --help
ploston version
ploston serve
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

### Command Not Found After pip install

Make sure your Python scripts directory is in your PATH:

```bash
# Linux/macOS
export PATH="$HOME/.local/bin:$PATH"

# Or use pipx for isolated installation
pipx install ploston-cli
```

### Docker Image Not Found

```bash
# Pull the latest image
docker pull ostanlabs/ploston:latest

# Or use the dev tag
docker pull ostanlabs/ploston:dev
```

## Next Steps

- [Quickstart Guide](quickstart.md) - Run your first workflow
- [First Workflow Tutorial](first-workflow.md) - Create a custom workflow
- [Configuration Reference](../reference/config-reference.md) - All config options
