# Workflow Schema Reference

Complete YAML schema reference for AEL workflows.

## Top-Level Structure

```yaml
# Required
name: string          # Workflow identifier (alphanumeric, hyphens)
version: string       # Semantic version (e.g., "1.0", "2.1.3")

# Optional
description: string   # Human-readable description
packages: object      # Python package configuration
defaults: object      # Default step settings

# Schema
inputs: array         # Input definitions
steps: array          # Step definitions (required, at least one)
outputs: array        # Output definitions (optional)
output: string        # Single output expression (alternative to outputs)
```

## Metadata

### `name` (required)

Unique workflow identifier.

- Type: `string`
- Pattern: `^[a-zA-Z][a-zA-Z0-9-]*$`
- Example: `data-transform`, `hello-world`

### `version` (required)

Semantic version string.

- Type: `string`
- Example: `"1.0"`, `"2.1.3"`

### `description` (optional)

Human-readable description.

- Type: `string`
- Example: `"Transform and validate JSON data"`

## Packages Configuration

```yaml
packages:
  profile: string     # Package profile: minimal | standard | data_science
  additional: array   # Additional packages to install
```

### Profiles

| Profile | Packages |
|---------|----------|
| `minimal` | json, re, datetime, math |
| `standard` | minimal + collections, itertools, functools, hashlib, uuid |
| `data_science` | standard + numpy, pandas (if available) |

## Defaults

```yaml
defaults:
  timeout: integer    # Default step timeout (seconds)
  on_error: string    # Error handling: fail | continue | retry
  retry: object       # Retry configuration
```

### Retry Configuration

```yaml
defaults:
  retry:
    max_attempts: 3           # Maximum retry attempts
    initial_delay: 1.0        # Initial delay (seconds)
    max_delay: 30.0           # Maximum delay (seconds)
    backoff_multiplier: 2.0   # Exponential backoff multiplier
```

## Inputs

```yaml
inputs:
  input_name:
    type: string        # Type: string | integer | number | boolean | array | object
    required: boolean   # Required input (default: true)
    default: any        # Default value (makes input optional)
    description: string # Human-readable description
    
    # Validation (optional)
    enum: array         # Allowed values
    pattern: string     # Regex pattern (strings only)
    minimum: number     # Minimum value (numbers only)
    maximum: number     # Maximum value (numbers only)
```

### Input Types

| Type | JSON Type | Example |
|------|-----------|---------|
| `string` | string | `"hello"` |
| `integer` | number | `42` |
| `number` | number | `3.14` |
| `boolean` | boolean | `true` |
| `array` | array | `[1, 2, 3]` |
| `object` | object | `{"key": "value"}` |

### Input Examples

```yaml
inputs:
  # Required string
  name:
    type: string
    description: User name

  # Optional with default
  count:
    type: integer
    default: 10
    minimum: 1
    maximum: 100

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

## Steps

```yaml
steps:
  - id: string          # Step identifier (required)
    
    # Type (exactly one required)
    tool: string        # MCP tool name
    code: string        # Python code block
    
    # Tool parameters (tool steps only)
    params: object      # Tool parameters
    
    # Dependencies
    depends_on: array   # List of step IDs to wait for
    
    # Error handling
    timeout: integer    # Step timeout (seconds)
    on_error: string    # Error handling: fail | continue | retry
    retry: object       # Retry configuration
```

### Tool Step

```yaml
steps:
  - id: fetch
    tool: http_get
    params:
      url: "{{ inputs.url }}"
      headers:
        Authorization: "Bearer {{ inputs.token }}"
```

### Code Step

```yaml
steps:
  - id: process
    code: |
      import json
      data = json.loads('{{ inputs.data }}')
      result = {"processed": data}
```

### Dependencies

```yaml
steps:
  - id: step1
    code: |
      result = "first"

  - id: step2
    depends_on: [step1]
    code: |
      result = "second"

  - id: step3
    depends_on: [step1, step2]
    code: |
      result = "third"
```

## Outputs

### Single Output

```yaml
output: "{{ steps.final.output }}"
```

### Multiple Outputs

```yaml
outputs:
  - name: string        # Output name
    from_path: string   # Path to value (e.g., "steps.process.output.data")
    value: string       # Template expression (alternative to from_path)
    description: string # Human-readable description
```

### Output Examples

```yaml
outputs:
  - name: result
    from_path: steps.transform.output
    description: Transformed data

  - name: count
    value: "{{ steps.count.output }}"
    description: Number of items processed
```

## Template Expressions

Use Jinja2 templates to reference values:

| Expression | Description |
|------------|-------------|
| `{{ inputs.name }}` | Input value |
| `{{ steps.id.output }}` | Step output |
| `{{ steps.id.output.field }}` | Nested field |
| `{{ value \| tojson }}` | JSON encode |
| `{{ value \| default('x') }}` | Default value |

## Complete Example

```yaml
name: data-pipeline
version: "1.0"
description: Fetch, transform, and validate data

packages:
  profile: standard

defaults:
  timeout: 30
  on_error: fail

inputs:
  url:
    type: string
    description: API endpoint URL
  format:
    type: string
    enum: ["json", "csv"]
    default: "json"

steps:
  - id: fetch
    tool: http_get
    params:
      url: "{{ inputs.url }}"
    timeout: 60

  - id: transform
    depends_on: [fetch]
    code: |
      data = {{ steps.fetch.output }}
      result = [item for item in data if item.get("active")]

  - id: format
    depends_on: [transform]
    code: |
      import json
      data = {{ steps.transform.output }}
      result = json.dumps(data, indent=2)

output: "{{ steps.format.output }}"
```

