# Quickstart

Get Ploston running and call your first workflow in under 10 minutes.

If you haven't installed Ploston yet, start with the [Installation guide](installation.md).

---

## Prerequisites

- `ploston bootstrap` completed (Control Plane running)
- `ploston init --import` completed (local runner started, tools imported)
- Claude Desktop restarted

Verify everything is up:

```bash
ploston runner status
ploston tools list
```

---

## Step 1: Create a workflow file

Create a file called `hello.yaml` anywhere on your machine:

```yaml
name: hello
version: "1.0.0"
description: "A simple greeting workflow"

inputs:
  - name: name
    type: string
    default: "World"

steps:
  - id: greet
    code: |
      name = context.inputs['name']
      return {"message": f"Hello, {name}! This ran deterministically inside Ploston."}

outputs:
  result:
    from: steps.greet.output
```

---

## Step 2: Validate it

```bash
ploston validate hello.yaml
```

Expected output:

```
✓ Workflow 'hello' is valid
  Steps: 1
  Inputs: name (string, default: "World")
  Outputs: result
```

---

## Step 3: Run it from the CLI

```bash
ploston run hello.yaml
```

Expected output:

```
✓ Executed: hello
Result: {"message": "Hello, World! This ran deterministically inside Ploston."}
Duration: 42ms
Execution ID: exec-a1b2c3d4
```

Run with a custom input:

```bash
ploston run hello.yaml -i name=Alice
```

---

## Step 4: Register it with the Control Plane

To make the workflow available as an MCP tool that Claude can call, copy it into your workflows directory and register it:

```bash
# Copy to wherever your workflows dir is configured
cp hello.yaml ~/my-workflows/hello.yaml
```

Then check it's registered:

```bash
ploston workflows list
```

You should see `hello` in the list. Claude Desktop can now call it as `w_hello`.

---

## Step 5: Call it from Claude Desktop

Open Claude Desktop and try:

> "Call the hello workflow with my name"

Claude will invoke `w_hello` — a single MCP tool call. Ploston executes it deterministically and returns the result. No orchestration loops, no intermediate tokens.

---

## What just happened

```
Claude Desktop
    │
    │  1 MCP call: w_hello(name="Alice")
    ▼
Ploston Control Plane
    │
    │  Executes: step greet (sandboxed Python)
    │  Duration: 42ms
    │  Trace: exec-a1b2c3d4
    ▼
Result returned to Claude Desktop
```

The LLM made one decision ("call hello workflow"), Ploston handled the execution. That's the pattern — at any scale.

---

## Next steps

- **[First Workflow Tutorial](first-workflow.md)** — Build a real multi-step workflow with tool calls
- **[Workflow Authoring Guide](../guides/workflow-authoring.md)** — Complete YAML reference
- **[CLI Reference](../reference/cli-reference.md)** — All CLI commands
- **[Examples](../examples/web-scraping.md)** — Ready-to-use workflows
