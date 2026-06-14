# Virtufin Examples

[![Build Status](https://git.haenerconsulting.com/virtufin/virtufin-examples/actions/workflows/docs/badge.svg?branch=master)](https://git.haenerconsulting.com/virtufin/virtufin-examples/actions)

📖 Documentation: [examples.doc.virtufin.com](https://examples.doc.virtufin.com)

Example applications in Python, C#, and TypeScript demonstrating how to use Virtufin services with market data APIs.

## Languages

| Language | Directory | Package Registry |
|----------|-----------|-----------------|
| **Python** | `python/` | Gitea PyPI |
| **TypeScript** | `typescript/` | Gitea npm |

## Setup

### Service Ports

Each Virtufin service uses two ports:

| Service | HTTP Port | gRPC Port | Purpose |
|---------|-----------|-----------|---------|
| **API Gateway** | 5001 | 5002 | HTTP/1.1 REST + gRPC-Web; gRPC HTTP/2 h2c |
| **WebSocketManager** | 5001 | 5002 | HTTP/1.1 health/management; gRPC HTTP/2 h2c |
| **WorkManager** | 5001 | 5002 | HTTP/1.1 health/management; gRPC HTTP/2 h2c |

- **HTTP port** — REST APIs, Swagger UI, health probes, gRPC-Web (API Gateway only)
- **gRPC port** — Native gRPC over HTTP/2 cleartext (h2c) for inter-service communication. Python/C#/F# clients connect to this port using standard gRPC.

### 1. Configure Gitea Registry Access

Set environment variables with your Gitea personal access token (scope: `read:packages`):

```bash
cp .env.example .env
# Edit .env with your credentials
source .env
```

### 2. Install Dependencies

**Python:**
```bash
cd python
source ../.env
export UV_INDEX_VIRTUFIN_USERNAME=$VIRTUFIN_REGISTRY_USERNAME
export UV_INDEX_VIRTUFIN_PASSWORD=$VIRTUFIN_REGISTRY_TOKEN
uv sync
```

**C# / F#:**
```bash
cd dotnet
dotnet restore
```

**TypeScript:**
```bash
cd typescript
npm install
```

### 3. Start Virtufin Services

Use the deploy scripts from `deploy/`:

**Docker Compose:**
```bash
# Start all services (production)
./deploy/docker/scripts/deploy_all.prod.sh

# Start all services (development, builds from source)
./deploy/docker/scripts/deploy_all.dev.sh

# Start backend only (API + WorkManager + WebSocketManager)
./deploy/docker/scripts/deploy_backend.prod.sh

# Stop all services
./deploy/docker/scripts/undeploy_all.sh
```

**Prod** (`.prod.sh`) uses pre-built images from `docker.haenerconsulting.com`.
**Dev** (`.dev.sh`) builds images from source repos.

**Local native** (no Docker containers for services):
```bash
# Start backend natively (Dapr sidecars only, no Docker for services)
./deploy/local/scripts/deploy_backend.sh

# Start individual services
./deploy/local/scripts/deploy_workmanager.sh
./deploy/local/scripts/deploy_websocketmanager.sh

# Stop
./deploy/local/scripts/undeploy_backend.sh
```

Requires Dapr CLI, .NET SDK, and Redis running locally. See [deploy/local/README.md](deploy/local/README.md).

Services and their host ports:

| Service | HTTP (host) | gRPC (host) |
|---------|-------------|--------------|
| API Gateway | `5001` | `5002` |
| WebSocketManager | `15001` | `15002` |
| WorkManager | `25001` | `25002` |

See [deploy/docker/README.md](deploy/docker/README.md) for full configuration options and standalone setup.

**Kubernetes** uses the [Virtufin Helm Chart](deploy/k8s/):
```bash
helm repo add virtufin https://git.haenerconsulting.com/virtufin/helm
helm install virtufin virtufin/virtufin \
  --namespace virtufin \
  --create-namespace \
  --set stateStore.host=valkey.valkey.svc.cluster.local \
  --set pubsub.host=valkey.valkey.svc.cluster.local
```

See [deploy/k8s/README.md](deploy/k8s/README.md) for all configurable values and options.

### 4. Run Examples

**Python:**
```bash
source .env
export UV_INDEX_VIRTUFIN_USERNAME=$VIRTUFIN_REGISTRY_USERNAME
export UV_INDEX_VIRTUFIN_PASSWORD=$VIRTUFIN_REGISTRY_TOKEN
cd python
uv run websocketmanager/binance_clob_example.py
uv run workmanager/orderflow_imbalance_worker.py
```

**TypeScript:**
```bash
cd typescript
npm run wsm   # WebSocketManager example
npm run wm    # WorkManager example
npm run api   # API Gateway example
```

### 5. Run WebsocketManager Controller Example

```bash
cd scripts
source ../.env
export UV_INDEX_VIRTUFIN_USERNAME=$VIRTUFIN_REGISTRY_USERNAME
export UV_INDEX_VIRTUFIN_PASSWORD=$VIRTUFIN_REGISTRY_TOKEN
uv sync
uv run python run_websocket_manager_controller.python.py
```

Or use the bash version (no Python deps needed):

```bash
./scripts/run_websocket_manager_controller.sh
```

## Examples

| Example | Description | Languages |
|---------|-------------|-----------|
| Binance CLOB | Stream BTC/USDT order book via WebSocketManager | Python |
| Alpaca CLOB | Stream BTC/USD order book with API key auth | Python |
| Order Flow Imbalance | Calculate bid/ask imbalance via WorkManager | Python |
| Service connectivity | Connect, list, disconnect (all 3 services) | C#, F#, TypeScript |
| State store | CRUD operations via API Gateway | C#, F#, TypeScript |

## Updating Deploy Directories

The `deploy/docker/` and `deploy/k8s/` directories are [git subtrees](https://www.atlassian.com/git/tutorials/git-subtree).
To pull the latest changes from their source repositories:

```bash
# Docker Compose
git subtree pull --prefix=deploy/docker \
  https://git.haenerconsulting.com/virtufin/docker-compose.git master --squash

# Helm (Kubernetes)
git subtree pull --prefix=deploy/k8s \
  https://git.haenerconsulting.com/virtufin/helm.git master --squash

# Local (native Dapr)
git subtree pull --prefix=deploy/local \
  https://git.haenerconsulting.com/virtufin/deploy-local.git master --squash
```

## Documentation

See the [docs site](./docs/v1/README.md) for architecture overview and detailed guides.
