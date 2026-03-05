# Code Steps Guide

Code steps execute Python in a secure sandbox. Use them when you need logic, transformation, or conditional behaviour between tool calls.

## Basic syntax

```yaml
steps:
  - id: process
    code: |
      # Your Python here.
      # Assign the step's output to the 'result' variable.
      data = context.inputs["data"]
      result = data.upper()
```

**`result =` is required.** The sandbox captures whatever you assign to `result` and makes it available to subsequent steps as `steps.process.output`. If you don't set `result`, the step returns `None`.

---

## Accessing context

The sandbox injects a `context` object with three attributes:

### `context.inputs`

Access workflow inputs by name:

```python
url   = context.inputs["url"]           # raises KeyError if missing
limit = context.inputs.get("limit", 10) # safe with default
```

### `context.steps`

Access output from any completed step:

```python
response = context.steps["fetch"].output        # full output object
body     = context.steps["fetch"].output["body"] # nested field
count    = context.steps["fetch"].output.get("count", 0)
```

### `context.tools.call()`

Call any MCP tool from within a code step:

```python
response = await context.tools.call("http_request", {
    "url": context.inputs["url"],
    "method": "GET"
})
result = response.get("body", "")
```

Tool calls from code are rate-limited (default: 10 per step). Use tool steps for single calls; code steps for conditional or iterative logic.

---

## Complete example

```yaml
steps:
  - id: fetch
    tool: http_request
    params:
      url: "{{ inputs.url }}"
      method: GET

  - id: process
    depends_on: [fetch]
    code: |
      import re
      from datetime import datetime

      # Access previous step output
      body = context.steps["fetch"].output.get("body", "")

      # Extract title
      match = re.search(r"<title[^>]*>(.*?)</title>", body, re.IGNORECASE)
      title = match.group(1).strip() if match else "Unknown"

      # Build result
      result = {
          "title": title,
          "fetched_at": datetime.now().isoformat(),
          "length": len(body),
      }
```

---

## Allowed imports

| Module | Use for |
|--------|---------|
| `json` | JSON encoding/decoding |
| `re` | Regular expressions |
| `datetime` | Date and time handling |
| `math` | Mathematical functions |
| `random` | Random number generation |
| `collections` | `defaultdict`, `Counter`, `deque` |
| `itertools` | Iteration utilities |
| `functools` | `reduce`, `partial`, `lru_cache` |
| `hashlib` | Hashing |
| `uuid` | UUID generation |
| `decimal` | Decimal arithmetic |
| `statistics` | `mean`, `median`, `stdev` |
| `operator` | Standard operators |
| `copy` | `copy`, `deepcopy` |
| `time` | `time.sleep`, timestamps |
| `typing` | Type hints |

Use imports at the top of the code block:

```python
import json
import re
from datetime import datetime

data = json.loads(context.inputs["json_data"])
result = {"processed": True, "at": datetime.now().isoformat()}
```

---

## Forbidden operations

The sandbox blocks anything that can escape the execution environment.

### Forbidden builtins

`eval`, `exec`, `compile`, `open`, `__import__`, `globals`, `locals`, `getattr`, `setattr`, `delattr`, `breakpoint`

### Forbidden imports

`os`, `sys`, `subprocess`, `socket`, `shutil`, `pathlib`, `urllib` (full module — `urllib.parse` is allowed), `requests`, `http`, `ctypes`, `pickle`

### Forbidden attribute access

Any dunder that enables class hierarchy traversal or code object inspection: `__class__`, `__bases__`, `__mro__`, `__subclasses__`, `__code__`, `__globals__`, `__builtins__`

---

## Error handling inside code

Use try/except to handle expected failures without failing the whole workflow:

```yaml
steps:
  - id: safe_parse
    code: |
      import json

      try:
          data = json.loads(context.inputs["json_data"])
          result = {"ok": True, "data": data}
      except json.JSONDecodeError as e:
          result = {"ok": False, "error": str(e)}
```

A downstream step can then branch on `steps.safe_parse.output["ok"]`.

---

## Timeout

Default timeout is 30 seconds. Override per step:

```yaml
steps:
  - id: slow_computation
    timeout: 120
    code: |
      # Up to 2 minutes
      result = expensive_calculation()
```

---

## Common patterns

### JSON processing

```python
import json

raw  = context.inputs["json_string"]
data = json.loads(raw)
result = json.dumps(data, indent=2)
```

### List transformation

```python
items  = context.steps["fetch"].output["items"]
result = [{"id": i, "value": x * 2} for i, x in enumerate(items)]
```

### Conditional logic

```python
value = context.inputs["threshold"]
if value > 100:
    result = "high"
elif value > 50:
    result = "medium"
else:
    result = "low"
```

### Aggregation

```python
from collections import defaultdict

rows   = context.steps["query"].output["rows"]
groups = defaultdict(list)
for row in rows:
    groups[row["category"]].append(row["value"])

result = {k: sum(v) for k, v in groups.items()}
```

### Calling a tool in a loop

```python
import json

urls    = context.inputs["urls"]  # list of strings
results = []

for url in urls[:5]:  # respect rate limits — max 10 calls/step
    resp = await context.tools.call("http_request", {"url": url, "method": "GET"})
    results.append({"url": url, "status": resp.get("status_code")})

result = results
```

---

## What the sandbox does NOT support

- Reading or writing files — use `fs_read` / `fs_write` tool steps instead
- Network access — use `http_request` tool steps instead
- Spawning processes
- Installing packages at runtime

All I/O goes through tool steps. Code steps handle transformation, logic, and aggregation.

---

## Next steps

- [Workflow Authoring Guide](workflow-authoring.md) — full YAML reference
- [Tool Integration Guide](tool-integration.md) — using MCP tools
- [Security Model](../concepts/security-model.md) — sandbox security in depth
- [Troubleshooting](troubleshooting.md) — common sandbox errors
