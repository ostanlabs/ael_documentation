# Workflow Authoring Guide

Complete guide to writing AEL workflows.

## Workflow Structure

Every workflow has this structure:

```yaml
# Metadata
name: my-workflow
version: "1.0"
description: What this workflow does

# Optional: Package configuration
packages:
  profile: standard
  additional: []

# Optional: Default settings
defaults:
  timeout: 30
  on_error: fail

# Inputs
inputs:
  input_name:
    type: string
    required: true
    default: "value"
    description: Input description

# Steps
steps:
  - id: step1
    code: |
      result = "output"

  - id: step2
    tool: tool_name
    params:
      key: "{{ steps.step1.output }}"

# Output
output: "{{ steps.step2.output }}"
```

## Metadata

### Required Fields

| Field | Description |
|-------|-------------|
| `name` | Unique workflow identifier (alphanumeric, hyphens) |
| `version` | Semantic version string (e.g., "1.0", "2.1.3") |

### Optional Fields

| Field | Description |
|-------|-------------|
| `description` | Human-readable description |
| `packages` | Python package configuration |
| `defaults` | Default step settings |

## Inputs

Define workflow inputs with validation:

```yaml
inputs:
  # Simple string input
  name:
    type: string
    default: "World"

  # Required input (no default)
  api_key:
    type: string
    required: true
    description: API key for authentication

  # Number with constraints
  count:
    type: integer
    minimum: 1
    maximum: 100
    default: 10

  # Enum (restricted values)
  format:
    type: string
    enum: ["json", "csv", "xml"]
    default: "json"

  # Pattern validation
  email:
    type: string
    pattern: "^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\\.[a-zA-Z0-9-.]+$"
```

### Input Types

| Type | Description |
|------|-------------|
| `string` | Text value |
| `integer` | Whole number |
| `number` | Decimal number |
| `boolean` | true/false |
| `array` | List of values |
| `object` | Key-value pairs |

## Steps

Steps are the building blocks of workflows. Each step is either a **code step** or a **tool step**.

### Code Steps

Execute Python code:

```yaml
steps:
  - id: process
    code: |
      data = "{{ inputs.data }}"
      result = data.upper()
```

See [Code Steps Guide](code-steps.md) for details.

### Tool Steps

Call MCP tools:

```yaml
steps:
  - id: fetch
    tool: http_get
    params:
      url: "{{ inputs.url }}"
      headers:
        Authorization: "Bearer {{ inputs.token }}"
```

### Step Dependencies

Control execution order with `depends_on`:

```yaml
steps:
  - id: step1
    code: |
      result = "first"

  - id: step2
    depends_on: [step1]  # Waits for step1
    code: |
      previous = "{{ steps.step1.output }}"
      result = f"after {previous}"

  - id: step3
    depends_on: [step1, step2]  # Waits for both
    code: |
      result = "last"
```

Without `depends_on`, steps execute in YAML order.

### Step Options

```yaml
steps:
  - id: risky_step
    tool: external_api
    params:
      data: "{{ inputs.data }}"
    
    # Timeout (seconds)
    timeout: 60
    
    # Error handling: fail | continue | retry
    on_error: retry
    
    # Retry configuration
    retry:
      max_attempts: 3
      initial_delay: 1.0
      max_delay: 30.0
      backoff_multiplier: 2.0
```

## Templates

Use Jinja2 templates to reference values:

### Accessing Inputs

```yaml
code: |
  name = "{{ inputs.name }}"
```

### Accessing Step Outputs

```yaml
code: |
  data = {{ steps.previous_step.output }}
```

### Template Filters

```yaml
code: |
  # Convert to JSON string
  json_str = '{{ steps.data.output | tojson }}'
  
  # Default value
  value = "{{ inputs.optional | default('fallback') }}"
```

## Output

Define workflow output:

```yaml
# Single output
output: "{{ steps.final.output }}"

# Multiple outputs
outputs:
  - name: result
    from_path: steps.process.output.data
  - name: count
    value: "{{ steps.count.output }}"
```

## Complete Example

```yaml
name: data-transform
version: "1.0"
description: Transform and validate data

inputs:
  data:
    type: string
    description: JSON data to transform
  format:
    type: string
    enum: ["json", "csv"]
    default: "json"

defaults:
  timeout: 30
  on_error: fail

steps:
  - id: parse
    code: |
      import json
      data = json.loads('{{ inputs.data }}')
      result = data

  - id: transform
    depends_on: [parse]
    code: |
      data = {{ steps.parse.output }}
      result = {k.upper(): v for k, v in data.items()}

  - id: format
    depends_on: [transform]
    code: |
      import json
      data = {{ steps.transform.output }}
      format_type = "{{ inputs.format }}"
      if format_type == "json":
          result = json.dumps(data, indent=2)
      else:
          result = ",".join(f"{k}={v}" for k, v in data.items())

output: "{{ steps.format.output }}"
```

## Next Steps

- [Code Steps Guide](code-steps.md) - Python code step details
- [Workflow Schema Reference](../reference/workflow-schema.md) - Complete schema
- [Examples](../examples/web-scraping.md) - More workflow examples

