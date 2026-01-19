# Why AEL?

AEL exists because **agents shouldn't orchestrate their own execution**.

This page explains why—and how AEL compares to alternatives.

---

## vs. Letting the Agent Orchestrate

The most common question: *"Why not just let Claude/GPT handle the tool calls directly?"*

### The Token Cost Problem

Every tool call in an agent loop costs tokens:

```
Step 1: Agent receives user request         (~100 tokens)
Step 2: Agent reasons about what to do      (~200 tokens)
Step 3: Agent calls tool #1                 (~50 tokens)
Step 4: Agent receives result #1            (~500 tokens)
Step 5: Agent reasons about result          (~200 tokens)
Step 6: Agent calls tool #2                 (~50 tokens)
Step 7: Agent receives result #2            (~500 tokens)
... and so on
```

A 5-step task easily costs 2,000-5,000 tokens. With AEL:

```
Step 1: Agent receives user request         (~100 tokens)
Step 2: Agent calls workflow                (~50 tokens)
Step 3: Agent receives final result         (~200 tokens)

Total: ~350 tokens
```

**Token reduction: 75-90% on multi-step tasks.**

### The Reliability Problem

LLMs are probabilistic. Given the same input, they might:
- Take a different execution path
- Format tool parameters differently
- Hallucinate an extra step
- Forget context from earlier in the chain

This is fine for creative tasks. It's unacceptable for automation.

AEL workflows are deterministic. Same inputs → same execution path → same outputs. You can write tests. You can make guarantees.

### The Debugging Problem

When an LLM-orchestrated task fails, you get:
- A blob of reasoning
- Maybe an error message
- No clear indication of which step failed
- No intermediate values

When an AEL workflow fails, you get:
- Exact step that failed
- Full trace of all previous steps
- Input/output values at each step
- Structured error with error code

### Summary

| Concern | Agent Orchestrates | AEL Orchestrates |
|---------|-------------------|------------------|
| Token cost | High, unpredictable | Low, predictable |
| Determinism | No | Yes |
| Testability | Difficult | Unit testable |
| Debugging | Painful | Structured traces |
| Retry logic | LLM decides | Configured, reliable |
| Audit trail | None | Complete |

---

## vs. Workflow Engines (Temporal, Airflow, Prefect)

*"Isn't AEL just another workflow engine?"*

No. AEL is **agent-native**. Traditional workflow engines aren't.

### The Protocol Gap

Temporal, Airflow, and Prefect don't speak MCP. To use them from an agent, you'd need to:

1. Build an MCP wrapper around the workflow engine
2. Handle the translation between MCP tool calls and workflow invocations
3. Map workflow outputs back to MCP responses
4. Build your own error handling layer

AEL speaks MCP natively. Workflows automatically appear as MCP tools. No glue code.

### The Mental Model Gap

Traditional workflow engines are designed for:
- Scheduled batch jobs
- Long-running data pipelines
- Human-triggered processes

AEL is designed for:
- Real-time agent requests
- Sub-second latency requirements
- Dynamic tool composition
- Multi-agent coordination

The execution model is different. AEL workflows are invoked like function calls, not submitted like jobs.

### The Integration Gap

Traditional workflow engines require:
- Separate deployment infrastructure
- Worker processes
- Queue management
- State persistence backends

AEL runs as a single process. Start it, connect your agent, done.

### When to Use What

| Use Case | Best Choice |
|----------|-------------|
| Nightly data pipeline | Airflow/Prefect |
| Long-running batch job | Temporal |
| Real-time agent tool calls | **AEL** |
| Sub-second execution | **AEL** |
| MCP-native integration | **AEL** |

---

## vs. MCP Gateways / Proxies

*"Can't I just put a proxy in front of my MCP servers?"*

A proxy routes requests. AEL **executes workflows**.

### What a Gateway Does

```
Agent → Gateway → MCP Server
             ↓
        (routing, auth, logging)
```

A gateway:
- Routes tool calls to the right server
- Adds authentication
- Logs requests
- Maybe rate limits

It doesn't understand what you're trying to accomplish. It just passes messages.

### What AEL Does

```
Agent → AEL → [multiple MCP servers]
          ↓
     (workflow execution)
          ↓
     Step 1: Call server A
     Step 2: Transform result
     Step 3: Call server B
     Step 4: Validate output
          ↓
     Return final result
```

AEL:
- Executes multi-step workflows
- Handles retries and errors
- Transforms data between steps
- Validates inputs and outputs
- Produces audit trails

### The Composition Problem

With a gateway, if you want to scrape → transform → publish, the agent must:

```
Call 1: firecrawl_scrape(...)     → 500 tokens
  ↓ LLM reasoning                 → 200 tokens
Call 2: (inline transformation)   → 300 tokens
  ↓ LLM reasoning                 → 200 tokens  
Call 3: kafka_publish(...)        → 100 tokens

Total: 5 calls, 1,300+ tokens, non-deterministic
```

With AEL:

```
Call 1: workflow:scrape-transform-publish(...)  → 100 tokens

Total: 1 call, 100 tokens, deterministic
```

Gateways don't compose tools. AEL does.

---

## vs. Agent Frameworks (LangChain, CrewAI, AutoGen)

*"How is AEL different from agent frameworks?"*

Agent frameworks help you **build agents**. AEL is **infrastructure agents call**.

### The Layer Difference

```
┌─────────────────────────────────────────────┐
│  Agent Framework (LangChain, etc.)          │
│  - Prompt management                        │
│  - Memory systems                           │
│  - Agent loops                              │
│  - Chain composition                        │
└─────────────────────┬───────────────────────┘
                      │ calls tools
                      ▼
┌─────────────────────────────────────────────┐
│  AEL (Execution Layer)                      │
│  - Workflow execution                       │
│  - Tool orchestration                       │
│  - Deterministic guarantees                 │
│  - Audit and governance                     │
└─────────────────────────────────────────────┘
```

AEL doesn't replace your agent framework. It's what your framework's agents call when they need reliable execution.

### Complementary, Not Competing

You can use:
- LangChain for agent orchestration + AEL for tool execution
- CrewAI for multi-agent coordination + AEL for deterministic workflows
- AutoGen for conversations + AEL for reliable actions

AEL makes your agents more reliable, regardless of which framework you use.

---

## The Category

AEL is not:
- ❌ An API gateway (we don't just route)
- ❌ A workflow engine (we're agent-native)
- ❌ An agent framework (we don't do planning)
- ❌ An MCP proxy (we add execution logic)

AEL is:
- ✅ A deterministic execution layer for AI agents
- ✅ The infrastructure that makes agents production-ready
- ✅ The "Kubernetes moment" for agent systems

**[Get Started →](getting-started/installation.md)**
