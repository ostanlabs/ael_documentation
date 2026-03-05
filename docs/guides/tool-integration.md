# Tool Integration Guide

How to use MCP tools in Ploston workflows.

---

## Two ways to call a tool

### Tool steps (recommended for single calls)

```yaml
steps:
  - id: fetch
    tool: http_request
    params:
      url: "{{ inputs.url }}"
      method: GET
```

Clean, declarative, retry/timeout/error handling all configurable.

### Tool calls from code steps (for conditional or iterative use)

```python
# Inside a code step
response = await context.tools.call("http_request", {
    "url": context.inputs["url"],
    "method": "GET"
})
result = response
```

Use this when you need a loop, a conditional, or want to branch based on a tool's output before deciding whether to call another.

---

## Native tools (always available)

Ploston includes a native-tools server that starts with the Control Plane. These tools are available in every workflow without any additional configuration.

### Filesystem

| Tool | Description |
|------|-------------|
| `fs_read` | Read a file — returns `{"content": "..."}` |
| `fs_write` | Write a file — returns `{"path": "...", "bytes_written": N}` |
| `fs_list` | List directory contents |
| `fs_delete` | Delete a file or directory |

```yaml
- id: read_config
  tool: fs_read
  params:
    path: "config/settings.json"
    format: json            # "text" (default) | "json" | "yaml"
```

```yaml
- id: save_output
  tool: fs_write
  params:
    path: "results/output.txt"
    content: "{{ steps.process.output.text }}"
    format: text
    overwrite: true
    create_dirs: true       # create parent directories if needed
```

### Network

| Tool | Description |
|------|-------------|
| `http_request` | HTTP request with retry — returns `{"status_code", "body", "headers"}` |
| `network_ping` | Ping a host |
| `network_dns_lookup` | DNS lookup |
| `network_port_check` | Check if a port is open |

```yaml
- id: api_call
  tool: http_request
  params:
    url: "https://api.example.com/data"
    method: POST
    headers:
      Authorization: "Bearer {{ inputs.token }}"
      Content-Type: "application/json"
    data:
      key: "value"
    timeout: 30
```

### Data transformation

| Tool | Description |
|------|-------------|
| `data_validate` | Validate data against a JSON Schema |
| `data_json_to_csv` | JSON array → CSV string |
| `data_csv_to_json` | CSV string → JSON array |
| `data_json_to_xml` | JSON → XML |
| `data_xml_to_json` | XML → JSON |

### Extraction

| Tool | Description |
|------|-------------|
| `extract_text` | Extract plain text from HTML, PDF, or other sources |
| `extract_structured` | Extract structured data using regex or CSS patterns |
| `extract_file_metadata` | Get file metadata (size, mime type, modification time) |

### Kafka

Requires `KAFKA_BOOTSTRAP_SERVERS` to be configured.

| Tool | Description |
|------|-------------|
| `kafka_publish` | Publish a message to a topic |
| `kafka_consume` | Consume messages from a topic |
| `kafka_list_topics` | List all topics |
| `kafka_create_topic` | Create a new topic |
| `kafka_health` | Check Kafka cluster health |

### Firecrawl

Requires `FIRECRAWL_BASE_URL` (and optionally `FIRECRAWL_API_KEY`) to be configured.

| Tool | Description |
|------|-------------|
| `firecrawl_search` | Web search |
| `firecrawl_map` | Map all URLs on a website |
| `firecrawl_extract` | Extract structured data from URLs |

### ML (Ollama)

Requires a running Ollama instance (default: `host.docker.internal:11434`).

| Tool | Description |
|------|-------------|
| `ml_embed_text` | Generate text embeddings |
| `ml_text_similarity` | Cosine or Euclidean similarity between two texts |
| `ml_classify_text` | Classify text into predefined categories |
| `ml_analyze_sentiment` | Sentiment analysis (lexicon-based, no Ollama needed) |

---

## External MCP servers

Any MCP server you've imported via `ploston init --import` is available as a tool. Tool names follow the pattern:

```
local__<server-name>__<tool-name>
```

For example, if you imported the GitHub MCP server:

```yaml
- id: create_issue
  tool: local__github__create_issue
  params:
    owner: "{{ inputs.org }}"
    repo: "{{ inputs.repo }}"
    title: "{{ inputs.title }}"
    body: "{{ steps.format.output.body }}"
```

Run `ploston tools list` to see all available tools and their exact names.

---

## Listing and inspecting tools

```bash
# List all tools
ploston tools list

# Filter by server
ploston tools list --server native-tools
ploston tools list --server github

# Inspect a tool's input schema
ploston tools show http_request
ploston tools show local__github__create_issue
```

Example `tools list` output:

```
NAME                              SERVER        STATUS
─────────────────────────────────────────────────────
fs_read                           native-tools  available
fs_write                          native-tools  available
http_request                      native-tools  available
kafka_publish                     native-tools  available
local__github__create_issue       github        available
local__github__search_code        github        available
w_page-report                     workflows     available
w_summarize-url                   workflows     available

Total: 8 tools
```

---

## Tool step options

```yaml
steps:
  - id: unreliable_api
    tool: http_request
    params:
      url: "https://flaky-api.example.com/data"

    timeout: 60             # override default timeout (seconds)

    on_error: retry         # fail (default) | skip | retry

    retry:
      max_attempts: 3
      initial_delay: 1.0    # seconds before first retry
      max_delay: 30.0       # cap on delay
      backoff_multiplier: 2.0
```

### Error handling options

| `on_error` | Behaviour |
|------------|-----------|
| `fail` (default) | Step fails → workflow stops |
| `skip` | Step failure is ignored, workflow continues with `null` output |
| `retry` | Retry up to `max_attempts` times with backoff, then fail |

---

## Calling tools from code steps

For conditional or iterative tool use:

```yaml
steps:
  - id: process_list
    code: |
      urls  = context.inputs["urls"]   # list from workflow input
      found = []

      for url in urls[:10]:            # loop — use a tool step for single calls
          response = await context.tools.call("http_request", {
              "url": url,
              "method": "GET",
              "timeout": 10
          })
          if response.get("status_code") == 200:
              found.append(url)

      result = {"reachable": found, "count": len(found)}
```

**Limit:** 10 tool calls per code step by default. Configure in `python_exec.max_tool_calls`.

---

## Adding a custom MCP server

Create a FastMCP server:

```python
# my_tools/server.py
from fastmcp import FastMCP

mcp = FastMCP("my-tools")

@mcp.tool()
def lookup_customer(customer_id: str) -> dict:
    """Look up a customer record."""
    return {"id": customer_id, "name": "Alice", "tier": "premium"}

if __name__ == "__main__":
    mcp.run()
```

Import it into Ploston:

```bash
ploston init --import --add-server "my-tools:python -m my_tools.server"
```

Or add it via `ploston init --import` on the next run — it will appear as a new server to select.

---

## Best practices

**Use tool steps for single calls** — they're easier to read, test, and configure with retry/timeout.

**Use code steps for logic** — conditional tool calls, loops over lists, fan-in patterns.

**Always set timeouts on external calls** — network tools have no default timeout at the MCP level; set one at the step level.

**Handle partial failures** — if a tool is non-critical, use `on_error: skip` and check `steps.id.output` for `null` downstream.

**Log tool calls in development** — set `logging.options.show_params: true` in your config to see parameters in logs.
