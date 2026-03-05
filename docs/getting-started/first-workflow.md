# Your First Workflow

Build a real multi-step workflow that fetches a web page, extracts its title, and saves a report to disk. Along the way you'll learn the core YAML syntax and see Ploston's actual value: the agent makes one tool call, Ploston handles everything else.

**Prerequisites:** [Installation](installation.md) complete — Control Plane running, runner started, Claude Desktop restarted.

---

## What we're building

A workflow called `page-report` that:

1. Fetches a URL via `http_request`
2. Extracts the page title with a Python code step
3. Writes a one-line report file via `fs_write`
4. Returns a summary to the agent

Claude calls it as a single MCP tool: `w_page-report`.

---

## Step 1: Create the workflow file

Create `page-report.yaml` anywhere on your machine:

```yaml
name: page-report
version: "1.0.0"
description: "Fetch a URL, extract its title, and save a report file"

inputs:
  - name: url
    type: string
    description: "URL to fetch"
  - name: output_path
    type: string
    default: "report.txt"
    description: "File path to write the report"

steps:
  # Step 1: fetch the page
  - id: fetch
    tool: http_request
    params:
      url: "{{ inputs.url }}"
      method: GET
    timeout: 30
    on_error: fail

  # Step 2: extract the title from the HTML
  - id: extract
    depends_on: [fetch]
    code: |
      import re

      # context.steps gives access to previous step outputs
      body = context.steps["fetch"].output.get("body", "")

      match = re.search(r"<title[^>]*>(.*?)</title>", body, re.IGNORECASE | re.DOTALL)
      title = match.group(1).strip() if match else "No title found"
      url   = context.inputs["url"]

      result = {"title": title, "url": url, "body_length": len(body)}

  # Step 3: write the report file
  - id: save
    depends_on: [extract]
    tool: fs_write
    params:
      path: "{{ inputs.output_path }}"
      content: "Title: {{ steps.extract.output.title }}\nURL:   {{ inputs.url }}\n"
      format: text

outputs:
  - name: title
    from: steps.extract.output.title
  - name: saved_to
    from: steps.save.output.path
  - name: body_length
    from: steps.extract.output.body_length
```

---

## Step 2: Validate it

```bash
ploston validate page-report.yaml
```

Expected output:

```
✓ Workflow 'page-report' is valid
  Steps:   3  (fetch → extract → save)
  Inputs:  url (required), output_path (default: "report.txt")
  Outputs: title, saved_to, body_length
```

---

## Step 3: Run it from the CLI

```bash
ploston run page-report.yaml -i url=https://example.com
```

Expected output:

```
✓ Executed: page-report
Duration: 820ms  |  Execution ID: exec-7f3a2b1c

Outputs:
  title       → "Example Domain"
  saved_to    → "report.txt"
  body_length → 1256

Steps:
  fetch    ✓  780ms   tool=http_request
  extract  ✓   28ms   code (8 lines)
  save     ✓   12ms   tool=fs_write
```

Verify the file was written:

```bash
cat report.txt
# Title: Example Domain
# URL:   https://example.com
```

---

## Step 4: Register it with the Control Plane

Copy the file to your configured workflows directory and verify it's registered:

```bash
cp page-report.yaml ~/my-workflows/page-report.yaml

ploston workflows list
```

You should see `page-report` in the list. Claude Desktop can now call it as `w_page-report`.

---

## Step 5: Call it from Claude Desktop

Open Claude Desktop and ask:

> "Fetch https://news.ycombinator.com and save a report to hn-report.txt"

Claude will call `w_page-report` — a single MCP tool invocation. Ploston runs all three steps deterministically and returns the result. No intermediate tokens, no LLM orchestration loop.

```
Claude Desktop
    │
    │  tools/call  w_page-report
    │  { "url": "https://news.ycombinator.com",
    │    "output_path": "hn-report.txt" }
    ▼
Ploston Control Plane
    │
    ├── [fetch]    http_request  → 200 OK, 42KB HTML
    ├── [extract]  code step     → title: "Hacker News"
    └── [save]     fs_write      → hn-report.txt
    │
    ▼
{ "title": "Hacker News", "saved_to": "hn-report.txt", "body_length": 42891 }
```

One LLM decision. Three deterministic steps. Full trace.

---

## Understanding the pieces

### `tool` steps call MCP tools directly

```yaml
- id: fetch
  tool: http_request        # real tool name from native-tools
  params:
    url: "{{ inputs.url }}" # Jinja2 template — renders at execution time
    method: GET
```

Tool names come from wherever your MCP servers are registered. Run `ploston tools list` to see everything available.

### `code` steps run sandboxed Python

```yaml
- id: extract
  code: |
    import re

    # Access previous step output
    body = context.steps["fetch"].output.get("body", "")

    # Access workflow inputs
    url = context.inputs["url"]

    # The result variable is what this step returns
    result = {"title": "...", "url": url}
```

The `context` object gives you:

| Expression | What you get |
|---|---|
| `context.inputs["name"]` | Workflow input value |
| `context.steps["id"].output` | Previous step's output (any type) |
| `await context.tools.call("tool_name", {...})` | Call any MCP tool from code |

**`result =` is required.** Whatever you assign to `result` becomes the step's output. If you don't set it, the step returns `None`.

### `depends_on` controls execution order

```yaml
- id: save
  depends_on: [extract]  # waits for extract to complete
```

Steps without `depends_on` run in YAML order. Add multiple dependencies to fan-in:

```yaml
- id: combine
  depends_on: [step_a, step_b]  # waits for both
```

### Templates reference earlier steps

```yaml
params:
  content: "Title: {{ steps.extract.output.title }}\n"
```

Template expressions are rendered at runtime using the actual step output. They work in `params`, `when` conditions, and `output` definitions.

---

## Calling a tool from a code step

If you need to call a tool conditionally or in a loop, you can do it from code:

```yaml
- id: fetch_and_check
  code: |
    import json

    # Call an MCP tool from within code
    response = await context.tools.call("http_request", {
        "url": context.inputs["url"],
        "method": "GET"
    })

    body = response.get("body", "")
    status = response.get("status_code", 0)

    result = {
        "ok": status == 200,
        "length": len(body),
        "preview": body[:200]
    }
```

Tool calls from code count against the per-step limit (default: 10). Use tool steps for single calls; use code steps when you need conditional or iterative logic.

---

## What to build next

- **[Workflow Authoring Guide](../guides/workflow-authoring.md)** — complete YAML reference: parallel steps, retries, conditionals, outputs
- **[Code Steps Guide](../guides/code-steps.md)** — sandbox security model, allowed imports, error patterns
- **[Tool Integration Guide](../guides/tool-integration.md)** — all native tools, connecting external MCP servers
- **[Examples](../examples/web-scraping.md)** — ready-to-run workflows you can adapt
