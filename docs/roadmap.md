# Roadmap

Ploston is actively developed. This page reflects the current state of the project.

---

## OSS Core — Available Now ✅

Everything in this section is shipped and working.

### Setup & Deployment
| Feature | Description |
|---------|-------------|
| **`ploston bootstrap`** | One-command Docker Compose deployment — pulls images, starts CP, Redis, native-tools |
| **`ploston init --import`** | Scans Claude Desktop / Cursor configs, interactive import, starts local runner |
| **Local Runner** | Daemon that connects your local MCP servers to the Control Plane over WebSocket |
| **Docker Compose** | Production-ready compose file with health checks and restart policies |
| **Kubernetes** | Helm charts + manifests for K8s deployment (`ploston bootstrap --target k8s`) |

### Execution Engine
| Feature | Description |
|---------|-------------|
| **Workflow Engine** | Sequential step execution with full dependency resolution |
| **MCP Frontend** | MCP server over stdio and HTTP+SSE — Claude Desktop, Cursor, any MCP client |
| **Tool Invoker** | Calls any MCP tool from a workflow step |
| **Python Sandbox** | Secure sandboxed code execution with 7-layer security model |
| **Jinja2 Templates** | Parameter templating across tool steps, conditions, and outputs |
| **Retry Logic** | Per-step configurable retry with exponential backoff |
| **Error Handling** | `on_error: fail / skip / retry` per step |

### Control Plane
| Feature | Description |
|---------|-------------|
| **Tool Registry** | Discovers and manages all MCP tools from connected runners |
| **Workflow Registry** | Loads, validates, and hot-reloads workflow YAML files |
| **REST API** | Full HTTP API for workflows, tools, executions, runners, config |
| **HTTP Transport** | MCP over HTTP+SSE with TLS support |
| **Config Tools** | MCP-native configuration management (`config_get`, `config_set`, `config_done`) |
| **Runner Management** | Multi-runner support with WebSocket connections and persistent registration |

### Observability
| Feature | Description |
|---------|-------------|
| **Prometheus Metrics** | Execution metrics, tool call counts, token savings estimates |
| **OpenTelemetry Traces** | Distributed tracing for workflow and step execution |
| **Structured Logging** | JSON-formatted logs via structlog |
| **Telemetry Store** | SQLite-backed execution history with configurable retention |
| **Grafana Dashboards** | Pre-built dashboards for execution metrics and system health |

### Developer Experience
| Feature | Description |
|---------|-------------|
| **CLI** | `ploston run`, `ploston validate`, `ploston workflows list/show`, `ploston tools list` |
| **Plugin Framework** | `PlostonPlugin` base class with pre/post execution hooks |
| **Plugin Registry** | Auto-discovery and loading of plugins |
| **Error Registry** | Consistent error codes across all failure modes |
| **Native Tools** | Built-in MCP server: filesystem, HTTP, data transforms, Kafka, Firecrawl, ML |

### Security
| Feature | Description |
|---------|-------------|
| **API Key Auth** | Token-based authentication for the REST API and MCP frontend |
| **7-Layer Sandbox** | Import restrictions, builtin restrictions, tool whitelist, rate limiting, parameter validation, timeout, recursion prevention |

---

## Phase 4: Enterprise Foundation — Planned Q2 2026

### Security & Governance
| Feature | Description |
|---------|-------------|
| **RBAC / ABAC** | Role and attribute-based access control for workflows and tools |
| **Policy Engine** | Cedar/Rego-based policy enforcement at the execution layer |
| **SSO Integration** | SAML and OIDC identity provider support |
| **Immutable Audit Logs** | Tamper-proof, compliance-ready execution audit trail |

### Advanced Orchestration
| Feature | Description |
|---------|-------------|
| **Parallel Execution** | Run independent steps concurrently |
| **Compensation Steps** | Automatic rollback on workflow failure |
| **Human Approval** | Pause workflow execution for human decision |
| **Async Execution** | Fire-and-forget workflows with webhook callbacks |
| **Execution Cancellation** | Cancel in-flight workflow executions |

### Operations
| Feature | Description |
|---------|-------------|
| **Multi-Tenancy** | Fully isolated execution environments per tenant |
| **Multi-Node Runtime** | Distributed execution across multiple runner nodes |
| **Kubernetes Operator** | Native K8s CRD-based workflow management |

---

## Phase 5: Enterprise Intelligence — H2 2026

### Learning & Optimization
| Feature | Description |
|---------|-------------|
| **Pattern Mining** | Detect repeated tool sequences from agent session telemetry |
| **Workflow Synthesis** | Auto-generate YAML workflows from observed patterns |
| **Token Cost Accounting** | Per-workflow token savings tracking and ROI reporting |
| **Tool Selection Engine** | Suggest the right tool or workflow for a given task |

### Enterprise Console
| Feature | Description |
|---------|-------------|
| **Visual Workflow Builder** | Drag-and-drop YAML workflow creation |
| **Workflow Library** | Browse, search, and clone workflows across your organization |
| **Tool Catalog** | Explore all registered tools with schemas and usage stats |
| **Execution Explorer** | Inspect, replay, and debug any execution |
| **Pattern Review** | Review LLM-synthesized workflows before publishing |

---

## OSS vs Enterprise at a glance

| Capability | OSS | Enterprise |
|------------|-----|------------|
| Workflow Engine | ✅ | ✅ |
| MCP Integration | ✅ | ✅ |
| Python Sandbox | ✅ | ✅ |
| CLI + REST API | ✅ | ✅ |
| Docker + K8s deploy | ✅ | ✅ |
| Prometheus + Grafana | ✅ | ✅ |
| Plugin Framework | ✅ | ✅ |
| RBAC / ABAC | — | Phase 4 |
| Policy Engine | — | Phase 4 |
| Parallel Execution | — | Phase 4 |
| Human Approval | — | Phase 4 |
| Multi-Tenancy | — | Phase 4 |
| Pattern Mining | — | Phase 5 |
| Workflow Synthesis | — | Phase 5 |
| Visual Builder | — | Phase 5 |

---

## Design Partners

We're looking for design partners to shape Enterprise features.

**Ideal partners:** teams running AI agents in production who need deterministic execution guarantees, compliance/governance requirements, or want to reduce agent token costs at scale.

**What you get:** early access to Enterprise features, direct input on roadmap priorities, dedicated support channel, discounted pricing at GA.

[Contact us](mailto:hello@ostanlabs.com) or open a GitHub discussion.

---

## Release Philosophy

- **OSS-first** — core execution is always open source
- **Enterprise adds, never subtracts** — OSS features stay OSS
- **Stability over speed** — ship when it's ready
- **Feedback-driven** — roadmap adapts to what users actually need

## Stay Updated

- **GitHub Releases** — watch the repo for release notifications
- **Discussions** — [github.com/ostanlabs/ploston/discussions](https://github.com/ostanlabs/ploston/discussions)
