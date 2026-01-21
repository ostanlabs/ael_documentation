# Ploston Docker Deployment Guide

This guide covers Docker deployment for Ploston (Agent Execution Layer) and native-tools.

## Quick Start

```bash
# Start all services
docker compose up -d

# Verify services are running
docker compose ps

# View logs
docker compose logs -f ploston

# Stop services
docker compose down
```

## Architecture

```
┌─────────────────────────────────────────┐
│           Docker Network                │
│                                         │
│  ┌─────────────┐    ┌──────────────┐   │
│  │   ploston   │───▶│ native-tools │   │
│  │   :8080     │    │    :8081     │   │
│  └─────────────┘    └──────────────┘   │
│         │                   │          │
└─────────┼───────────────────┼──────────┘
          │                   │
          ▼                   ▼
     Host :8080          External APIs
     Host :9090          (Kafka, Firecrawl,
     (metrics)            Ollama)
```

## Images

### Ploston Image

**Purpose**: Main MCP server with workflow engine and tool orchestration

| Property | Value |
|----------|-------|
| Dockerfile | `Dockerfile` |
| Base | `python:3.12-slim` |
| Size Target | <500MB |
| Ports | 8080 (MCP), 9090 (metrics) |

**Build:**
```bash
docker build -t ploston:latest .
```

**Run standalone:**
```bash
docker run -p 8080:8080 -p 9090:9090 ploston:latest
```

### Native Tools Image

**Purpose**: MCP server with filesystem, network, and data tools

| Property | Value |
|----------|-------|
| Dockerfile | `docker/native-tools/Dockerfile` |
| Base | `python:3.12-slim` |
| Size Target | <400MB |
| Port | 8081 |

**Build:**
```bash
docker build -t native-tools:latest -f docker/native-tools/Dockerfile .
```

## Configuration

### Environment Variables

#### Ploston Service

| Variable | Description | Default |
|----------|-------------|---------|
| `PLOSTON_HOST` | Bind host | `0.0.0.0` |
| `PLOSTON_PORT` | HTTP port | `8080` |
| `PLOSTON_TRANSPORT` | Transport type | `http` |
| `PLOSTON_CONFIG` | Config file path | `/app/config/ploston-config.yaml` |
| `PLOSTON_TELEMETRY_ENABLED` | Enable telemetry | `true` |
| `PLOSTON_METRICS_ENABLED` | Enable Prometheus metrics | `true` |
| `PLOSTON_TRACES_ENABLED` | Enable distributed tracing | `true` |
| `LOG_LEVEL` | Log level | `INFO` |

#### Native Tools Service

| Variable | Description | Default |
|----------|-------------|---------|
| `NATIVE_TOOLS_HOST` | Bind host | `0.0.0.0` |
| `NATIVE_TOOLS_PORT` | HTTP port | `8081` |
| `FIRECRAWL_BASE_URL` | Firecrawl service URL | - |
| `FIRECRAWL_API_KEY` | Firecrawl API key | - |
| `KAFKA_BOOTSTRAP_SERVERS` | Kafka bootstrap servers | - |
| `OLLAMA_HOST` | Ollama service URL | `http://host.docker.internal:11434` |

### Volume Mounts

```yaml
volumes:
  # Configuration files
  - ./config:/app/config:ro
  
  # Workflow definitions
  - ./workflows:/app/workflows:ro
  
  # Native tools data directory
  - native-tools-data:/data
```

## Development Mode

Use `docker-compose.dev.yaml` for development with hot-reload:

```bash
# Start development environment
docker compose -f docker-compose.dev.yaml up -d

# View logs
docker compose -f docker-compose.dev.yaml logs -f

# Get a shell in the container
docker compose -f docker-compose.dev.yaml exec ploston-dev bash

# Run tests in container
docker compose -f docker-compose.dev.yaml exec ploston-dev pytest

# Stop
docker compose -f docker-compose.dev.yaml down
```

### Development Features

- Source code mounted for hot-reload
- Debug logging enabled
- Test files accessible
- Interactive shell support

## Health Checks

Both services expose health endpoints:

```bash
# Ploston health
curl http://localhost:8080/health

# Native tools health
curl http://localhost:8081/health

# Prometheus metrics
curl http://localhost:9090/metrics
```

## Connecting External Services

### Kafka

```bash
# Set Kafka bootstrap servers
export KAFKA_BOOTSTRAP_SERVERS=host.docker.internal:9092
docker compose up -d
```

### Firecrawl

```bash
# Set Firecrawl configuration
export FIRECRAWL_BASE_URL=http://host.docker.internal:3002
export FIRECRAWL_API_KEY=your-api-key
docker compose up -d
```

### Ollama

```bash
# Ollama on host machine (default)
export OLLAMA_HOST=http://host.docker.internal:11434
docker compose up -d
```

## Troubleshooting

### Container won't start

```bash
# Check logs
docker compose logs ploston

# Check health status
docker compose ps

# Rebuild images
docker compose build --no-cache
```

### Connection refused to native-tools

Ensure native-tools is healthy before Ploston starts:

```bash
# Check native-tools health
docker compose exec native-tools curl http://localhost:8081/health
```

### Permission denied errors

The containers run as non-root users. Ensure mounted volumes have correct permissions:

```bash
# Fix permissions on data directory
chmod -R 755 ./data
```

## Resource Limits

Default resource limits in production:

| Service | CPU Limit | Memory Limit |
|---------|-----------|--------------|
| ploston | 2.0 | 2GB |
| native-tools | 1.0 | 1GB |

Adjust in `docker-compose.yml` under `deploy.resources`.

## Multi-Architecture Support

Images support both `amd64` and `arm64` architectures:

```bash
# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t ploston:latest .
```
