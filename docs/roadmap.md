# Roadmap

Ploston is actively developed. This page shows what's available now and what's coming.

---

## Current: OSS Core (Available Now)

The open-source release includes everything you need to run deterministic workflows:

### Data Plane ✅
| Feature | Description |
|---------|-------------|
| **MCP Frontend** | Native MCP protocol support |
| **Workflow Engine** | Sequential step execution with error handling |
| **Tool Invoker** | Call any MCP tool from workflows |
| **Python Sandbox** | Secure code execution with 7-layer security |
| **Template Engine** | Jinja2-based parameter templating |

### Control Plane ✅
| Feature | Description |
|---------|-------------|
| **Tool Registry** | Discover and manage MCP tools |
| **Workflow Registry** | Load and validate workflow definitions |
| **Config Loader** | YAML-based configuration |
| **MCP Client Manager** | Connect to multiple MCP servers |

### Developer Experience ✅
| Feature | Description |
|---------|-------------|
| **CLI Interface** | `ploston run`, `ploston validate`, `ploston list` commands |
| **Self-Configuration Tools** | Manage Ploston via MCP tools |
| **Structured Logging** | JSON-formatted execution logs |
| **Error Registry** | Consistent error codes and messages |

---

## Phase 2: Plugin Framework (In Progress)

**Status:** 🔜 Current focus
**Timeline:** Q1 2026 (January–March)

| Deliverable | Description |
|-------------|-------------|
| **Plugin System** | `PlostonPlugin` base class for extensibility |
| **Plugin Registry** | Discover and load plugins |
| **Hook System** | Pre/post execution hooks |
| **OSS Repo Public** | Apache 2.0 license, public GitHub |

---

## Phase 3: OSS Hardening (Coming Soon)

**Timeline:** Q1 2026

| Feature | Description |
|---------|-------------|
| **REST API** | HTTP endpoints for workflow/tool management |
| **HTTP Transport** | MCP over HTTP/SSE with TLS |
| **Telemetry Store** | SQLite persistence for execution history |
| **Prometheus Metrics** | Execution metrics export |
| **Authentication** | API key and token-based auth |

---

## Phase 4: Enterprise Foundation (Planned)

**Timeline:** Q2 2026

### Security & Governance
| Feature | Description |
|---------|-------------|
| **RBAC/ABAC** | Role and attribute-based access control |
| **Policy Engine** | Cedar/Rego-based policy enforcement |
| **SSO Integration** | SAML, OIDC identity providers |
| **Audit Logging** | Complete execution audit trail |

### Advanced Orchestration
| Feature | Description |
|---------|-------------|
| **Parallel Execution** | Run steps concurrently |
| **Compensation Steps** | Rollback on failure |
| **Human Approval** | Pause for human decision |
| **Async Execution** | Fire-and-forget workflows |
| **Execution Cancellation** | Cancel in-flight workflows |

### Operations
| Feature | Description |
|---------|-------------|
| **Multi-Tenancy** | Isolated execution environments |
| **Multi-Node Runtime** | Distributed execution |
| **Kubernetes Operator** | Native K8s deployment |

---

## Phase 5: Enterprise Intelligence (Future)

**Timeline:** H2 2026

### Learning & Optimization
| Feature | Description |
|---------|-------------|
| **Pattern Mining** | Detect common tool sequences |
| **Workflow Synthesis** | Auto-generate workflows from patterns |
| **Tool Selection Engine** | Recommend tools for tasks |
| **Cost Accounting** | Token and resource tracking |

### Enterprise Console
| Feature | Description |
|---------|-------------|
| **Visual Workflow Builder** | Drag-and-drop workflow creation |
| **Workflow Library** | Browse and search workflows |
| **Tool Catalog** | Explore available tools |
| **Execution Explorer** | Debug and analyze runs |
| **Pattern Review** | Review and approve synthesized workflows |

---

## Feature Summary

| Category | OSS | Enterprise |
|----------|-----|------------|
| **Core Runtime** | ✅ | ✅ |
| **MCP Integration** | ✅ | ✅ |
| **Python Sandbox** | ✅ | ✅ |
| **CLI** | ✅ | ✅ |
| **REST API** | Phase 3 | ✅ |
| **RBAC/ABAC** | — | Phase 4 |
| **Policy Engine** | — | Phase 4 |
| **Parallel Execution** | — | Phase 4 |
| **Human Approval** | — | Phase 4 |
| **Pattern Mining** | — | Phase 5 |
| **Workflow Synthesis** | — | Phase 5 |
| **Visual Builder** | — | Phase 5 |

---

## Design Partners

We're looking for design partners to shape Enterprise features.

**Ideal partners:**
- Running AI agents in production
- Need deterministic execution guarantees
- Have compliance/governance requirements
- Want to reduce agent token costs

**What you get:**
- Early access to Enterprise features
- Direct input on roadmap priorities
- Dedicated support channel
- Discounted pricing at GA

**Interested?** [Contact us](mailto:hello@ostanlabs.com) or open a GitHub discussion.

---

## Release Philosophy

1. **OSS-first** — Core execution is always open source
2. **Enterprise adds, doesn't subtract** — OSS features stay OSS
3. **Stability over speed** — We ship when it's ready
4. **Feedback-driven** — Roadmap adapts to user needs

---

## Stay Updated

- **GitHub Releases** — Watch the repo for release notifications
- **Discussions** — Join the conversation on GitHub Discussions
- **Twitter** — Follow [@ostanlabs](https://twitter.com/ostanlabs) for updates
