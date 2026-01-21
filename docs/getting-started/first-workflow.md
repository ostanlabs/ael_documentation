# First Workflow Tutorial

Learn to build a complete workflow with multiple steps, inputs, and outputs.

## What We'll Build

A workflow that:
1. Takes a list of numbers as input
2. Calculates statistics (sum, average, min, max)
3. Returns a formatted report

## Step 1: Create the Workflow File

Create `workflows/number-stats.yaml`:

```yaml
name: number-stats
version: "1.0"
description: Calculate statistics for a list of numbers

inputs:
  numbers:
    type: string
    description: Comma-separated list of numbers
    default: "1,2,3,4,5"

steps:
  - id: parse
    code: |
      raw = "{{ inputs.numbers }}"
      numbers = [float(n.strip()) for n in raw.split(",")]
      result = numbers

  - id: calculate
    depends_on: [parse]
    code: |
      numbers = {{ steps.parse.output }}
      result = {
          "count": len(numbers),
          "sum": sum(numbers),
          "average": sum(numbers) / len(numbers),
          "min": min(numbers),
          "max": max(numbers)
      }

  - id: format
    depends_on: [calculate]
    code: |
      stats = {{ steps.calculate.output }}
      result = f"""
      Number Statistics Report
      ========================
      Count:   {stats['count']}
      Sum:     {stats['sum']}
      Average: {stats['average']:.2f}
      Min:     {stats['min']}
      Max:     {stats['max']}
      """

output: "{{ steps.format.output }}"
```

## Step 2: Understand the Structure

### Metadata

```yaml
name: number-stats      # Unique identifier
version: "1.0"          # Semantic version
description: ...        # Human-readable description
```

### Inputs

```yaml
inputs:
  numbers:
    type: string        # Input type
    description: ...    # Help text
    default: "1,2,3,4,5"  # Default value
```

### Steps

Each step has:
- `id`: Unique identifier within the workflow
- `code`: Python code to execute
- `depends_on`: List of steps that must complete first

### Output

```yaml
output: "{{ steps.format.output }}"
```

References the output of the `format` step using Jinja2 templating.

## Step 3: Validate

```bash
ploston validate workflows/number-stats.yaml
```

## Step 4: Run

With default input:

```bash
ploston run workflows/number-stats.yaml
```

With custom input:

```bash
ploston run workflows/number-stats.yaml -i "numbers=10,20,30,40,50"
```

Expected output:
```
Number Statistics Report
========================
Count:   5
Sum:     150.0
Average: 30.00
Min:     10.0
Max:     50.0
```

## Key Concepts

### Step Dependencies

Steps execute in order based on `depends_on`:

```yaml
steps:
  - id: step1
    code: ...

  - id: step2
    depends_on: [step1]  # Waits for step1
    code: ...

  - id: step3
    depends_on: [step1, step2]  # Waits for both
    code: ...
```

### Accessing Step Outputs

Use Jinja2 templates to access previous step outputs:

```yaml
- id: use_previous
  code: |
    data = {{ steps.previous_step.output }}
```

### The `result` Variable

Each code step must set a `result` variable. This becomes the step's output:

```yaml
- id: example
  code: |
    result = {"key": "value"}  # This is the step output
```

## Next Steps

- [Workflow Authoring Guide](../guides/workflow-authoring.md) - Complete workflow reference
- [Code Steps Guide](../guides/code-steps.md) - Python code step details
- [Examples](../examples/web-scraping.md) - More workflow examples

