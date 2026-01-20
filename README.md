# AEL Documentation

[![Netlify Status](https://api.netlify.com/api/v1/badges/b57f9264-9998-489b-b8bf-61a2f41171ac/deploy-status)](https://app.netlify.com/projects/ostanlabs/deploys)

Public documentation for the [Agent Execution Layer (AEL)](https://github.com/ostanlabs/agent-execution-layer).

**Live Site:** [https://ostanlabs.netlify.app](https://ostanlabs.netlify.app)

---

## Quick Links

| Resource | URL |
|----------|-----|
| **Production Docs** | [ostanlabs.netlify.app](https://ostanlabs.netlify.app) |
| **Main Repo** | [github.com/ostanlabs/agent-execution-layer](https://github.com/ostanlabs/agent-execution-layer) |
| **Netlify Dashboard** | [app.netlify.com/projects/ostanlabs](https://app.netlify.com/projects/ostanlabs) |

---

## Deployment

Documentation is automatically deployed via **Netlify** on every push to `main`.

| Trigger | Action |
|---------|--------|
| Push to `main` | Production deploy to [ostanlabs.netlify.app](https://ostanlabs.netlify.app) |
| Pull Request | Deploy preview at `deploy-preview-{PR#}--ostanlabs.netlify.app` |

### Build Configuration

- **Build Command:** `pip install -r requirements-docs.txt && mkdocs build`
- **Publish Directory:** `site/`
- **Python Version:** 3.11

---


## Local Development

### Prerequisites

- Python 3.11+
- pip or uv

### Setup

```bash
# Clone the repo
git clone https://github.com/ostanlabs/ael_documentation.git
cd ael_documentation

# Install dependencies
pip install -r requirements-docs.txt

# Start local server
mkdocs serve
```

The site will be available at [http://localhost:8000](http://localhost:8000).

### Build

```bash
mkdocs build
```

Output is generated in the `site/` directory.

---

## Documentation Structure

```
docs/
├── index.md                    # Homepage
├── why-ael.md                  # Differentiation page
├── roadmap.md                  # Roadmap and phases
├── getting-started/
│   ├── installation.md
│   ├── quickstart.md
│   └── first-workflow.md
├── concepts/
│   ├── how-ael-works.md        # Mental model
│   ├── execution-model.md      # Step execution details
│   ├── workflows-as-tools.md   # MCP integration
│   └── security-model.md       # 7-layer security
├── guides/
│   ├── workflow-authoring.md
│   ├── code-steps.md
│   ├── tool-integration.md
│   └── troubleshooting.md
├── reference/
│   ├── cli-reference.md
│   ├── config-reference.md
│   ├── workflow-schema.md
│   └── error-codes.md
└── examples/
    ├── web-scraping.md
    ├── data-processing.md
    └── api-integration.md
```

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| **Static Site Generator** | [MkDocs](https://www.mkdocs.org/) |
| **Theme** | [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) |
| **Diagrams** | [Mermaid](https://mermaid.js.org/) via mkdocs-mermaid2-plugin |
| **Hosting** | [Netlify](https://www.netlify.com/) |

---

## Contributing

1. Create a branch from `main`
2. Make your changes
3. Open a PR — Netlify will create a deploy preview
4. Get review and merge

---

## License

Documentation content is licensed under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0).
