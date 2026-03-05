# Docker Deployment

`ploston bootstrap` manages Docker for you under the hood. This page explains what it deploys, how to operate it, and how to customise the setup.

---

## What bootstrap deploys

```
┌──────────────────────────────────────────────────┐
│              ploston-network (bridge)             │
│                                                  │
│  ┌──────────────┐   ┌────────────┐  ┌────────┐  │
│  │   ploston    │──▶│native-tools│  │ redis  │  │
│  │  (CP) :8082  │   │    :8081   │  │ :6379  │  │
│  └──────┬───────┘   └────────────┘  └────────┘  │
│         │                                        │
└─────────┼──────────────────────────────────────  ┘
          │ WebSocket
    ┌─────▼──────┐
    │local runner│   (on your machine, not in Docker)
    └────────────┘
```

Three containers, one network:

| Container | Port | Role |
|-----------|------|------|
| `ploston` | 8082 | Control Plane — workflow engine, MCP frontend, REST API |
| `native-tools` | 8081 | Built-in MCP server — filesystem, HTTP, data, Kafka, Firecrawl |
| `redis` | 6379 | Config store and pub/sub for runner coordination |

The local runner runs **outside** Docker on your machine and connects to the CP over WebSocket. Your local MCP servers (GitHub, filesystem, custom tools) never enter the container.

---

## Compose file location

Bootstrap generates the compose file at:

```
~/.ploston/docker-compose.yaml
```

All `ploston bootstrap` subcommands use this file automatically. You can also use `docker compose` directly:

```bash
docker compose -f ~/.ploston/docker-compose.yaml <command>
```

---

## Common operations

```bash
# Deploy (first time or after image updates)
ploston bootstrap

# Start already-deployed stack (fast, no re-pull)
ploston bootstrap up

# Stop without removing volumes
ploston bootstrap down

# Check status
ploston bootstrap status

# Stream logs
ploston bootstrap logs
ploston bootstrap logs --service ploston   # single container
```

---

## Variants

### With observability stack

```bash
ploston bootstrap --with-observability
```

Adds three additional containers:

| Container | Port | Role |
|-----------|------|------|
| `prometheus` | 9090 | Metrics collection |
| `grafana` | 3000 | Dashboards (pre-built Ploston boards included) |
| `loki` | 3100 | Log aggregation |

Access Grafana at `http://localhost:3000` (default login: admin / admin).

### Kubernetes

```bash
ploston bootstrap --target k8s
```

Generates Helm charts and `kubectl`-ready manifests in `~/.ploston/k8s/`. See the [Kubernetes deployment section](#kubernetes-deployment) below.

---

## Environment variables

The compose file reads these from `~/.ploston/.env` (created by `ploston init --import`):

### Control Plane (`ploston`)

| Variable | Default | Description |
|----------|---------|-------------|
| `AEL_HOST` | `0.0.0.0` | Bind address |
| `AEL_PORT` | `8082` | HTTP port |
| `LOG_LEVEL` | `INFO` | Log level |
| `PLOSTON_METRICS_ENABLED` | `true` | Prometheus metrics |
| `PLOSTON_TRACES_ENABLED` | `true` | OpenTelemetry tracing |

### Native Tools (`native-tools`)

| Variable | Default | Description |
|----------|---------|-------------|
| `NATIVE_TOOLS_PORT` | `8081` | HTTP port |
| `FIRECRAWL_BASE_URL` | — | Firecrawl service URL |
| `FIRECRAWL_API_KEY` | — | Firecrawl API key |
| `KAFKA_BOOTSTRAP_SERVERS` | — | Kafka bootstrap servers |
| `OLLAMA_HOST` | `http://host.docker.internal:11434` | Ollama for ML tools |

Add or override variables by editing `~/.ploston/.env`.

---

## Health checks

Both containers expose health endpoints:

```bash
curl http://localhost:8082/health   # Control Plane
curl http://localhost:8081/health   # native-tools
```

A healthy response looks like:

```json
{
  "status": "healthy",
  "service": "ploston",
  "version": "1.3.0"
}
```

`ploston bootstrap status` runs these checks and shows a summary.

---

## Connecting external services

### Kafka

```bash
# In ~/.ploston/.env
KAFKA_BOOTSTRAP_SERVERS=host.docker.internal:9092
```

Then restart: `ploston bootstrap down && ploston bootstrap up`

### Firecrawl (local)

```bash
FIRECRAWL_BASE_URL=http://host.docker.internal:3002
FIRECRAWL_API_KEY=your-key
```

### Ollama

Ollama defaults to `host.docker.internal:11434` — no configuration needed if it's running locally on the default port.

---

## Updating images

```bash
# Pull latest images and redeploy
ploston bootstrap

# Or manually
docker compose -f ~/.ploston/docker-compose.yaml pull
docker compose -f ~/.ploston/docker-compose.yaml up -d
```

---

## Kubernetes deployment

After `ploston bootstrap --target k8s`, the generated files live at `~/.ploston/k8s/`:

```
~/.ploston/k8s/
├── namespace.yaml
├── configmap.yaml
├── secrets.yaml          # edit before applying
├── deployment-ploston.yaml
├── deployment-native-tools.yaml
├── deployment-redis.yaml
└── services.yaml
```

Apply to your cluster:

```bash
kubectl apply -f ~/.ploston/k8s/
kubectl rollout status deployment/ploston -n ploston
```

For Helm-based deployment:

```bash
helm install ploston ~/.ploston/helm/ \
  --namespace ploston \
  --create-namespace \
  --values ~/.ploston/helm/values.yaml
```

---

## Troubleshooting

### Control Plane won't start

```bash
# Check logs
ploston bootstrap logs --service ploston

# Common causes:
# - Port 8082 already in use → change AEL_PORT in ~/.ploston/.env
# - Redis not ready yet → wait 10s and retry ploston bootstrap up
```

### native-tools unhealthy

```bash
curl http://localhost:8081/health
# Look at "dependencies" field — shows Kafka/Ollama/Firecrawl status
# Degraded dependencies don't break the CP; those specific tools just won't work
```

### Port conflicts

Edit `~/.ploston/.env` to change ports, then regenerate the compose file:

```bash
ploston bootstrap --regenerate
```

### Runner can't connect to Control Plane

```bash
ploston runner status
# If "disconnected" — check that the CP is healthy first
# Then: ploston runner restart
```

### Full reset

```bash
ploston bootstrap down --volumes   # removes Redis data too
ploston bootstrap                  # fresh deploy
ploston init --import              # re-run setup
```
