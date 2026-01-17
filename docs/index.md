# Agent Execution Layer (AEL) Documentation

Welcome to the Agent Execution Layer documentation. AEL is a workflow execution engine with MCP (Model Context Protocol) integration that enables AI agents to execute complex, multi-step workflows.

## Quick Links

- **[Installation](getting-started/installation.md)** - Get AEL installed on your system
- **[Quickstart](getting-started/quickstart.md)** - Run your first workflow in 5 minutes
- **[First Workflow](getting-started/first-workflow.md)** - Create your first custom workflow

## What is AEL?

AEL (Agent Execution Layer) is a workflow execution engine that:

- **Executes YAML-defined workflows** with tool and code steps
- **Integrates with MCP servers** to access external tools
- **Provides a CLI** for workflow management and execution
- **Exposes workflows as MCP tools** for AI agent consumption
- **Supports REST API** for programmatic access

## Documentation Sections

### Getting Started

| Guide | Description |
|-------|-------------|
| [Installation](getting-started/installation.md) | Install via pip, Docker, or from source |
| [Quickstart](getting-started/quickstart.md) | 5-minute introduction to AEL |
| [First Workflow](getting-started/first-workflow.md) | Step-by-step workflow tutorial |

### Guides

| Guide | Description |
|-------|-------------|
| [Workflow Authoring](guides/workflow-authoring.md) | Complete guide to writing workflows |
| [Code Steps](guides/code-steps.md) | Using Python code in workflows |
| [Tool Integration](guides/tool-integration.md) | Connecting MCP tools |
| [Troubleshooting](guides/troubleshooting.md) | Common issues and solutions |

### Reference

| Reference | Description |
|-----------|-------------|
| [CLI Reference](reference/cli-reference.md) | All CLI commands and options |
| [Workflow Schema](reference/workflow-schema.md) | Complete YAML schema reference |
| [Configuration](reference/config-reference.md) | Configuration file options |
| [Error Codes](reference/error-codes.md) | Error codes and resolutions |

### Examples

| Example | Description |
|---------|-------------|
| [Web Scraping](examples/web-scraping.md) | Extract data from websites |
| [Data Processing](examples/data-processing.md) | Transform and process data |
| [API Integration](examples/api-integration.md) | Integrate with external APIs |

## System Requirements

- **Python**: 3.12 or higher
- **OS**: macOS, Linux, Windows
- **Memory**: 512MB minimum, 1GB recommended

## Getting Help

- **GitHub Issues**: [Report bugs or request features](https://github.com/ostanlabs/agent-execution-layer/issues)
- **Discussions**: [Ask questions and share ideas](https://github.com/ostanlabs/agent-execution-layer/discussions)

## License

AEL is released under the [MIT License](https://github.com/ostanlabs/agent-execution-layer/blob/main/LICENSE).

