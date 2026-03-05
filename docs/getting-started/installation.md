# Installation

This guide walks you through installing Ploston and getting the Control Plane running on your machine.

## Requirements

- **Python**: 3.12 or higher
- **Docker & Docker Compose**: v2.20+
- **OS**: macOS, Linux, or Windows (WSL2)
- **MCP client**: Claude Desktop, Cursor, or any MCP-compatible client

---

## Step 1: Install the CLI

```bash
pip install ploston-cli
```

Verify the install:

```bash
ploston --version
```

---

## Step 2: Deploy the Control Plane

```bash
ploston bootstrap
```

Bootstrap pulls the Ploston Docker images, generates `~/.ploston/docker-compose.yaml`, starts the Control Plane and supporting services (Redis, native-tools), and waits for everything to become healthy.

**What you'll see:**

```
Ploston Bootstrap
=================

Step 1: Prerequisites
---------------------
✓ Docker Engine 25.0.0 detected
✓ Docker Compose v2.24.0 detected
✓ Port 8082 available (Control Plane)
✓ Port 6379 available (Redis)

Step 2: Pull Images
-------------------
✓ Pulling ostanlabs/ploston:latest
✓ Pulling native-tools:latest
✓ Pulling redis:7-alpine

Step 3: Start Services
----------------------
✓ redis started (healthy)
✓ native-tools started
✓ ploston started

Step 4: Health Check
--------------------
✓ Control Plane healthy at http://localhost:8082

Step 5: Import Detection
------------------------
✓ Found: Claude Desktop (5 servers configured)
Import now? [Y/n]:
```

The first run takes about 60 seconds. Subsequent starts are instant.

### Deployment options

```bash
# Default: Docker Compose on localhost
ploston bootstrap

# Include Prometheus + Grafana + Loki observability stack
ploston bootstrap --with-observability

# Deploy to Kubernetes instead
ploston bootstrap --target k8s
```

---

## Step 3: Import your existing MCP tools

When Bootstrap completes, it hands off to `ploston init --import`. You can also run it manually at any time:

```bash
ploston init --import
```

This command:
1. Detects your existing MCP configurations from Claude Desktop and/or Cursor
2. Shows an interactive server picker — choose which servers to bring in
3. Detects secrets and generates `~/.ploston/.env`
4. Pushes the configuration to the Control Plane
5. Starts the local runner daemon
6. Optionally injects Ploston into your Claude Desktop config

**The interactive flow:**

```
Ploston Config Import
=====================

✓ Control Plane connected at http://localhost:8082

Scanning for MCP configurations...
✓ Found: Claude Desktop (5 servers configured)
✓ Found: Cursor (3 servers, 2 overlap)
6 unique servers found.

Select servers to import:
  [x] github         npx @modelcontextprotocol/server-github
  [x] filesystem     npx @modelcontextprotocol/server-filesystem
  [x] postgres       npx @modelcontextprotocol/server-postgres
  [ ] brave-search   npx @modelcontextprotocol/server-brave-search
                      ⚠ BRAVE_API_KEY not set in environment
  [x] custom-slack   node /Users/marc/tools/slack-mcp/index.js
  [x] docker         npx @modelcontextprotocol/server-docker

5 servers selected

✓ Runner 'local' started — 23 tools available

Inject Ploston into Claude Desktop config? [y/N]: y
✓ Backed up original config
✓ Added ploston MCP server entry

→ Restart Claude Desktop to pick up the new config.
```

---

## Step 4: Restart Claude Desktop

After init completes, restart Claude Desktop. Your tools now route through Ploston.

---

## Step 5: Verify everything is running

```bash
ploston runner status    # Check the local runner
ploston tools list       # See all available tools
```

You should see your imported tools listed, prefixed by runner name (e.g. `local__github__create_issue`).

---

## Managing the stack

```bash
# Check what's running
ploston bootstrap status

# View Control Plane logs
ploston bootstrap logs

# Stop the stack
ploston bootstrap down

# Start the stack again (fast, no re-pull)
ploston bootstrap up
```

---

## Configuration Modes

The Control Plane operates in two modes:

**Configuration mode** — Active when first deployed. Only config tools are available via MCP. This is what `bootstrap` starts in by default, so `init --import` can push the initial config.

**Running mode** — Active after `init --import` completes. All workflow and tool execution is available.

You generally don't need to manage this manually — the bootstrap + init flow handles the transition automatically.

---

## From Source (Development)

To run Ploston from source:

```bash
git clone https://github.com/ostanlabs/ploston.git
cd ploston

# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync

# Run the CLI
uv run ploston --help
```

---

## Troubleshooting

### Docker not found

```
Error: Docker Engine not detected
```

Install Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop/) and ensure it's running before calling `ploston bootstrap`.

### Port already in use

```
Error: Port 8082 is already in use
```

Another service is occupying port 8082 (Control Plane default). Either stop that service or check `~/.ploston/docker-compose.yaml` to change the port mapping, then re-run `ploston bootstrap`.

### CLI command not found after install

```bash
# macOS/Linux — add pip scripts to PATH
export PATH="$HOME/.local/bin:$PATH"

# Or install with pipx for isolated install
pipx install ploston-cli
```

### Control Plane unhealthy

```bash
# Check logs
ploston bootstrap logs

# Or directly with Docker
docker compose -f ~/.ploston/docker-compose.yaml logs ploston
```

---

## Next Steps

- [Quickstart](quickstart.md) — Write and run your first workflow
- [First Workflow Tutorial](first-workflow.md) — Step-by-step walkthrough
- [CLI Reference](../reference/cli-reference.md) — All available commands
