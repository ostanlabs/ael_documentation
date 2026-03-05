<!--
  DESTINATION
  Repo:  ostanlabs/ploston  (public OSS repo, separate from the monorepo)
  Path:  README.md

  NOTE: This is the public-facing GitHub README for the ploston package repo.
  It is NOT the monorepo README (which lives at the repo root and is for internal dev use).
  When the public repo is created, copy this file to its root as README.md.
-->

# Ploston

> **LLMs plan. Ploston executes.**

Ploston is a deterministic execution layer for AI agents. Define multi-step tool workflows in YAML, expose them as MCP tools, and let your agent call them with a single invocation — no orchestration loops, no token explosion, no unpredictable behavior.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![PyPI](https://img.shields.io/pypi/v/ploston-cli)](https://pypi.org/project/ploston-cli/)
[![Docs](https://img.shields.io/badge/docs-docs.ploston.ai-brightgreen)](https://docs.ploston.ai)

---

## The Problem

When an AI agent orchestrates its own tool calls, every intermediate result flows back through the LLM:

```
User → LLM → Tool 1 → LLM → Tool 2 → LLM → Tool 3 → LLM
              800t          1500t          2200t
```

A 3-step task costs ~10,000 tokens. Run it twice and you may get different results. When something fails, you get a reasoning blob with no clear trace.

**This is why agent workflows that work in demos break in production.**

## The Fix

With Ploston, the LLM is called exactly twice — once to decide what to do, once to receive the result:

```
User → LLM → Ploston → Tool 1 → Tool 2 → Tool 3 → LLM
       200t    (deterministic execution, no LLM in the loop)   300t
```

The same 3-step task costs ~500 tokens. Same inputs always produce the same outputs. Every execution is fully traced.

| | Without Ploston | With Ploston |
|---|---|---|
| 3-step workflow | ~10,000 tokens | ~500 tokens |
| 5-step workflow | ~25,000 tokens | ~600 tokens |
| Behavior | Non-deterministic | Deterministic |
| Testable | ✗ | ✓ |
| Auditable | ✗ | ✓ |
| Self-hosted | — | ✓ |

---

## Quick Start

### Prerequisites

- Docker & Docker Compose
- Python 3.12+
- Claude Desktop or any MCP-compatible client

### Install

```bash
pip install ploston-cli
```

### Deploy the Control Plane

```bash
ploston bootstrap
```

This pulls the Ploston images, generates `~/.ploston/docker-compose.yaml`, starts the Control Plane and supporting services, and waits for everything to be healthy. Takes about 60 seconds on first run.

```
Ploston Bootstrap
=================
✓ Docker Engine detected
✓ Pulling ostanlabs/ploston:latest
✓ Pulling native-tools:latest
✓ Pulling redis:7-alpine
✓ Control Plane healthy at http://localhost:8082
✓ Found: Claude Desktop (5 servers configured)
Import now? [Y/n]:
```

### Import your existing MCP tools

```bash
ploston init --import
```

Ploston scans your Claude Desktop and Cursor configs, shows you an interactive picker for which MCP servers to bring in, generates a local `.env` with your secrets, and starts the local runner. When prompted, allow Ploston to inject itself into your Claude Desktop config — your existing tools still work, now routed through the execution layer.

### Restart Claude Desktop

Your tools are now available through Ploston. You'll see them as before.

### Write your first workflow

Create `summarize-page.yaml`:

```yaml
name: summarize-page
version: "1.0.0"
description: "Fetch a URL and return the first 2000 characters"

inputs:
  - name: url
    type: string
    description: "URL to fetch"

steps:
  - id: fetch
    tool: http_request
    params:
      url: "{{ inputs.url }}"
      method: GET

  - id: trim
    code: |
      content = context.steps['fetch'].output['body']
      return {"text": content[:2000], "length": len(content)}

outputs:
  summary:
    from: steps.trim.output
```

Validate and check it's registered:

```bash
ploston validate summarize-page.yaml
ploston workflows list
```

Claude can now call `w_summarize-page` as a single tool call.

---

## How It Works

```
┌──────────────┐    MCP (stdio)    ┌─────────────────────────────────┐
│ Claude / any │ ◄───────────────► │       Ploston Control Plane     │
│  MCP client  │                   │                                  │
└──────────────┘                   │  ┌────────────┐  ┌───────────┐  │
                                   │  │  Workflow  │  │   Tool    │  │
                                   │  │  Engine    │  │ Registry  │  │
                                   │  └─────┬──────┘  └─────┬─────┘  │
                                   └────────┼───────────────┼────────┘
                                            │               │
                              ┌─────────────▼───────────────▼──────┐
                              │          Local Runner               │
                              │  (executes your MCP servers)        │
                              │  filesystem, github, postgres, ...  │
                              └─────────────────────────────────────┘
```

**Control Plane** — Manages tool registration, workflow execution, routing, and telemetry. Runs as a Docker service on your machine.

**Local Runner** — A daemon that runs on your machine, connects to the Control Plane over WebSocket, and executes your local MCP servers. Your tools never leave your machine.

**Bridge** — Translates stdio ↔ HTTP+SSE so any MCP client can connect to the Control Plane.

**Workflows** — YAML files defining deterministic multi-step tool sequences. Once registered, they appear as MCP tools prefixed with `w_`.

---

## Workflow YAML — Quick Reference

```yaml
name: my-workflow          # Tool appears as w_my-workflow
version: "1.0.0"
description: "..."

inputs:
  - name: query
    type: string
    required: true
  - name: limit
    type: integer
    default: 10

steps:
  # Call any MCP tool
  - id: search
    tool: local__github__search_code
    params:
      query: "{{ inputs.query }}"

  # Run inline Python
  - id: process
    code: |
      results = context.steps['search'].output['items']
      return {"count": len(results), "top": results[:context.inputs['limit']]}

  # Conditional step
  - id: notify
    tool: slack_post
    when: "{{ steps.process.output.count > 0 }}"
    params:
      channel: "#alerts"
      text: "Found {{ steps.process.output.count }} results"

outputs:
  results:
    from: steps.process.output
```

**Step types:** `tool` (any MCP tool), `code` (sandboxed Python), `template` (Jinja2 transform)

**Context:** `{{ inputs.name }}`, `{{ steps.id.output }}`, `{{ env.VAR_NAME }}`

**Error handling:** `on_error: retry`, `on_error: skip`, `on_error: fail`

---

## CLI Reference

```bash
# Control Plane lifecycle
ploston bootstrap                        # Deploy CP stack (Docker Compose)
ploston bootstrap --with-observability   # Include Prometheus + Grafana + Loki
ploston bootstrap --target k8s           # Deploy to Kubernetes

# Initial setup
ploston init --import                    # Import MCP configs from Claude Desktop / Cursor

# Local runner
ploston runner start --daemon            # Start runner as background daemon
ploston runner status                    # Check runner health
ploston runner stop                      # Stop the runner
ploston runner logs --follow             # Stream runner logs

# Manage workflows
ploston workflows list                   # List all registered workflows
ploston workflows show <name>            # Show workflow details and steps
ploston validate <file.yaml>             # Validate a workflow file

# Inspect tools
ploston tools list                       # List all available tools
ploston tools list --server github       # Filter by MCP server
```

---

## What's Included (OSS)

**Execution Engine** — YAML workflows, sandboxed Python steps, Jinja2 templating, retry logic, structured traces

**MCP Integration** — MCP server (HTTP+SSE and stdio), tool registry with auto-discovery, workflow tools as `w_*` MCP tools

**Operations** — Docker Compose deployment, Prometheus metrics, OpenTelemetry traces, REST management API

**Developer Experience** — `ploston bootstrap` one-command setup, `ploston init --import` zero-config onboarding, plugin framework

**Enterprise (paid)** — RBAC/ABAC policies, parallel execution, immutable audit logs, multi-tenancy, Kubernetes operator, visual workflow builder, pattern mining + workflow synthesis

---

## Examples

**Fetch and summarize any URL**
```yaml
name: summarize-url
steps:
  - id: fetch
    tool: http_request
    params: { url: "{{ inputs.url }}", method: GET }
  - id: trim
    code: return {"content": context.steps['fetch'].output['body'][:3000]}
```

**Search GitHub and post a Slack digest**
```yaml
name: code-digest
steps:
  - id: search
    tool: local__github__search_code
    params: { query: "{{ inputs.query }}", repo: "{{ inputs.repo }}" }
  - id: format
    code: |
      items = context.steps['search'].output['items'][:5]
      return {"summary": "\n".join(f"- {i['path']}" for i in items)}
  - id: post
    tool: slack_post
    params: { channel: "{{ inputs.channel }}", text: "{{ steps.format.output.summary }}" }
```

---

## Deployment Options

**Local (Docker Compose)** — Default. Runs on your laptop.
```bash
ploston bootstrap
```

**Self-hosted (Kubernetes)** — For team or production deployments.
```bash
ploston bootstrap --target k8s
```

**With observability** — Adds Prometheus, Grafana, and Loki.
```bash
ploston bootstrap --with-observability
```

---

## Contributing

```bash
git clone https://github.com/ostanlabs/ploston
cd ploston
make dev    # Install dev dependencies
make test   # Run tests
make lint   # Run linters
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

---

## Documentation

- [Getting Started](https://docs.ploston.ai/getting-started/installation) — Install and run your first workflow
- [Workflow Reference](https://docs.ploston.ai/reference/workflow-schema) — Complete YAML schema
- [CLI Reference](https://docs.ploston.ai/reference/cli-reference) — All commands and flags
- [Why Ploston](https://docs.ploston.ai/why-ploston) — The case for deterministic execution

---

## License

Apache 2.0 — see [LICENSE](LICENSE)
