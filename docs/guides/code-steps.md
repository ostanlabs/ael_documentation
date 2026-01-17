# Code Steps Guide

Complete guide to using Python code in AEL workflows.

## Overview

Code steps execute Python code in a secure sandbox environment. They're useful for:

- Data transformation and processing
- Conditional logic
- Calculations and aggregations
- Formatting output

## Basic Syntax

```yaml
steps:
  - id: my_step
    code: |
      # Your Python code here
      data = "{{ inputs.data }}"
      result = data.upper()
```

**Important:** Every code step must set a `result` variable. This becomes the step's output.

## Accessing Data

### Inputs

Access workflow inputs using Jinja2 templates:

```yaml
steps:
  - id: process
    code: |
      name = "{{ inputs.name }}"
      count = {{ inputs.count }}
      result = f"Hello {name}, count is {count}"
```

### Previous Step Outputs

Access outputs from previous steps:

```yaml
steps:
  - id: step1
    code: |
      result = {"items": [1, 2, 3]}

  - id: step2
    depends_on: [step1]
    code: |
      data = {{ steps.step1.output }}
      result = sum(data["items"])
```

## Allowed Imports

The sandbox allows these standard library modules:

| Module | Description |
|--------|-------------|
| `json` | JSON encoding/decoding |
| `re` | Regular expressions |
| `datetime` | Date and time handling |
| `math` | Mathematical functions |
| `random` | Random number generation |
| `typing` | Type hints |
| `collections` | Container datatypes |
| `itertools` | Iterator functions |
| `functools` | Higher-order functions |
| `hashlib` | Secure hashes |
| `uuid` | UUID generation |
| `decimal` | Decimal arithmetic |
| `statistics` | Statistical functions |
| `operator` | Standard operators |
| `copy` | Shallow/deep copy |
| `time` | Time access |

### Using Imports

```yaml
steps:
  - id: process
    code: |
      import json
      import re
      from datetime import datetime
      
      data = json.loads('{{ inputs.json_data }}')
      timestamp = datetime.now().isoformat()
      result = {"data": data, "processed_at": timestamp}
```

## Forbidden Operations

For security, these are **not allowed**:

### Forbidden Builtins

| Builtin | Reason |
|---------|--------|
| `eval()` | Arbitrary code execution |
| `exec()` | Arbitrary code execution |
| `compile()` | Code compilation |
| `open()` | File system access |
| `__import__()` | Dynamic imports |
| `globals()` | Global namespace access |
| `locals()` | Local namespace access |
| `getattr()` | Attribute access |
| `setattr()` | Attribute modification |
| `delattr()` | Attribute deletion |
| `breakpoint()` | Debugger access |

### Forbidden Imports

These modules are **blocked**:

- `os` - Operating system access
- `sys` - System access
- `subprocess` - Process execution
- `socket` - Network access
- `requests` - HTTP requests (use tool steps instead)
- `urllib` - URL handling
- `shutil` - File operations
- `pathlib` - Path operations

## Calling Tools from Code

Use the `tools.call()` method to call MCP tools:

```yaml
steps:
  - id: fetch_data
    code: |
      # Call an MCP tool
      response = await tools.call("http_get", {
          "url": "{{ inputs.api_url }}"
      })
      result = response
```

### Tool Call Limits

- Default: 10 tool calls per code step
- Configurable via `python_exec.max_tool_calls`

## Error Handling

Handle errors within code steps:

```yaml
steps:
  - id: safe_parse
    code: |
      import json
      
      try:
          data = json.loads('{{ inputs.json_data }}')
          result = {"success": True, "data": data}
      except json.JSONDecodeError as e:
          result = {"success": False, "error": str(e)}
```

## Timeout

Code steps have a default timeout of 30 seconds. Override per step:

```yaml
steps:
  - id: long_running
    timeout: 120  # 2 minutes
    code: |
      # Long-running computation
      result = complex_calculation()
```

## Best Practices

### 1. Keep Code Simple

```yaml
# Good: Simple, focused code
steps:
  - id: transform
    code: |
      data = {{ steps.fetch.output }}
      result = [item["name"] for item in data]
```

### 2. Use Multiple Steps

Break complex logic into multiple steps:

```yaml
steps:
  - id: parse
    code: |
      import json
      result = json.loads('{{ inputs.data }}')

  - id: filter
    depends_on: [parse]
    code: |
      data = {{ steps.parse.output }}
      result = [x for x in data if x["active"]]

  - id: format
    depends_on: [filter]
    code: |
      items = {{ steps.filter.output }}
      result = "\n".join(str(x) for x in items)
```

### 3. Validate Input

```yaml
steps:
  - id: validate
    code: |
      value = {{ inputs.count }}
      if not isinstance(value, int) or value < 0:
          raise ValueError("count must be a positive integer")
      result = value
```

## Common Patterns

### JSON Processing

```yaml
- id: process_json
  code: |
    import json
    data = json.loads('{{ inputs.json_string }}')
    result = json.dumps(data, indent=2)
```

### List Transformation

```yaml
- id: transform_list
  code: |
    items = {{ steps.fetch.output }}
    result = [{"id": i, "value": x * 2} for i, x in enumerate(items)]
```

### Conditional Logic

```yaml
- id: decide
  code: |
    value = {{ inputs.threshold }}
    if value > 100:
        result = "high"
    elif value > 50:
        result = "medium"
    else:
        result = "low"
```

## Next Steps

- [Workflow Authoring Guide](workflow-authoring.md) - Complete workflow reference
- [Tool Integration Guide](tool-integration.md) - Using MCP tools
- [Troubleshooting](troubleshooting.md) - Common issues

