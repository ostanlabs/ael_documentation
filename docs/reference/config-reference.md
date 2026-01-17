# Configuration Reference

Complete reference for AEL configuration options.

## Configuration File

AEL looks for configuration in these locations (in order):

1. Path specified with `--config` CLI option
2. `./ael-config.yaml` (current directory)
3. `~/.config/ael/config.yaml` (user config)
4. `/etc/ael/config.yaml` (system config)

## Configuration Sections

### `mcp` - MCP Server Configuration

```yaml
mcp:
  enabled: true                    # Enable MCP server
  transport: stdio                 # Transport: stdio | http
  
  exposure:
    workflows: true                # Expose workflows as MCP tools
    tools: true                    # Expose tools via MCP
  
  http:
    host: "0.0.0.0"               # HTTP host
    port: 8080                     # HTTP port
    cors_origins: ["*"]            # CORS allowed origins
    tls:
      enabled: false               # Enable TLS
      cert_file: ""                # TLS certificate file
      key_file: ""                 # TLS key file
```

### `tools` - Tool Configuration

```yaml
tools:
  mcp_servers:
    # MCP server definitions
    filesystem:
      command: "npx"               # Command to spawn (stdio)
      args: ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
      env: {}                      # Environment variables
      timeout: 30                  # Connection timeout (seconds)
    
    # HTTP MCP server example
    remote-server:
      url: "http://localhost:8081/mcp"  # Server URL (http)
      transport: http              # Transport type
      timeout: 30
  
  system_tools:
    python_exec_enabled: true      # Enable python_exec tool
```

### `workflows` - Workflow Configuration

```yaml
workflows:
  directory: "./workflows"         # Workflow files directory
  watch: true                      # Watch for file changes
```

### `execution` - Execution Configuration

```yaml
execution:
  default_timeout: 300             # Default workflow timeout (seconds)
  step_timeout: 30                 # Default step timeout (seconds)
  max_steps: 100                   # Maximum steps per workflow
  
  retry:
    max_attempts: 3                # Maximum retry attempts
    initial_delay: 1.0             # Initial delay (seconds)
    max_delay: 30.0                # Maximum delay (seconds)
    backoff_multiplier: 2.0        # Exponential backoff multiplier
```

### `python_exec` - Python Execution Configuration

```yaml
python_exec:
  timeout: 30                      # Code execution timeout (seconds)
  max_tool_calls: 10               # Max tool calls per execution
  default_imports:                 # Auto-imported modules
    - json
    - re
    - datetime
  
  packages:
    default_profile: standard      # Package profile: minimal | standard | data_science
```

### `logging` - Logging Configuration

```yaml
logging:
  level: INFO                      # Log level: DEBUG | INFO | WARNING | ERROR
  format: colored                  # Format: colored | json
  
  components:
    workflow: true                 # Log workflow events
    step: true                     # Log step events
    tool: true                     # Log tool events
    sandbox: true                  # Log sandbox events
  
  options:
    show_params: true              # Show parameters in logs
    show_results: true             # Show results in logs
    truncate_at: 200               # Truncate long values
```

### `plugins` - Plugin Configuration

```yaml
plugins:
  - name: logging                  # Plugin name
    type: builtin                  # Type: builtin | file | package
    priority: 50                   # Execution priority (lower = earlier)
    enabled: true                  # Enable/disable plugin
    config: {}                     # Plugin-specific config
  
  - name: custom-plugin
    type: file
    path: "./plugins/custom.py"    # Path to plugin file
    priority: 100
```

### `security` - Security Configuration

```yaml
security:
  require_auth: false              # Require API authentication
  
  api_keys:
    - name: "default"              # Key name
      key: "${API_KEY}"            # Key value (use env var)
      scopes:                      # Allowed scopes
        - read
        - write
        - execute
```

### `telemetry` - Telemetry Configuration

```yaml
telemetry:
  enabled: true                    # Enable telemetry
  service_name: "ael"              # Service name for traces
  service_version: "1.0.0"         # Service version
  
  storage:
    type: memory                   # Storage: memory | file | postgres
  
  metrics:
    enabled: true                  # Enable metrics
    prometheus_enabled: true       # Expose /metrics endpoint
  
  tracing:
    enabled: false                 # Enable tracing
    sample_rate: 1.0               # Sampling rate (0.0-1.0)
    export_to_otlp: false          # Export to OTLP collector
  
  logging:
    enabled: false                 # Enable OTEL logging
    export_to_otlp: false          # Export logs to OTLP
    include_trace_context: true    # Include trace IDs in logs
  
  export:
    otlp:
      enabled: false               # Enable OTLP export
      endpoint: "http://localhost:4317"  # OTLP endpoint
      insecure: true               # Use insecure connection
      protocol: grpc               # Protocol: grpc | http
      headers: {}                  # Auth headers
```

## Environment Variables

Use `${VAR_NAME}` syntax to reference environment variables:

```yaml
tools:
  mcp_servers:
    api-server:
      env:
        API_KEY: "${MY_API_KEY}"
```

## Complete Example

See [examples/ael-config.yaml](https://github.com/ostanlabs/agent-execution-layer/blob/main/examples/ael-config.yaml) for a complete example.

