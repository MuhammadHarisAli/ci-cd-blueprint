# CI/CD Blueprint

A production-ready CI/CD blueprint for NestJS applications with Docker, GitHub Actions, and Kubernetes.

## What's Included

| Component | Technology |
|-----------|-----------|
| Framework | NestJS |
| Container | Docker (multi-stage build) |
| CI/CD | GitHub Actions |
| Registry | GitHub Container Registry (GHCR) |
| Orchestration | Kubernetes + Kustomize |

## Prerequisites

Before you start, install these tools:

- [Node.js 20](https://nodejs.org/)
- [Docker](https://docs.docker.com/get-docker/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) (for Kubernetes deployment)
- [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/) (for K8s manifest management)

## Quick Start

### 1. Clone and Install

```bash
git clone https://github.com/MuhammadHarisAli/ci-cd-blueprint.git
cd ci-cd-blueprint
npm install
```

### 2. Run Locally (Without Docker)

```bash
npm run start:dev
```

Test the health endpoint:

```bash
curl http://localhost:3000/health
```

You should see:

```json
{"status":"ok","info":{},"error":{},"details":{}}
```

### 3. Run Locally (With Docker)

```bash
docker-compose up
```

Or build and run manually:

```bash
docker build -t ci-cd-blueprint:latest .
docker run -d -p 3000:3000 ci-cd-blueprint:latest
curl http://localhost:3000/health
```

## How CI/CD Works

```
You push code to GitHub
        |
GitHub Actions runs CI Pipeline
  |-- Lint & Format (ESLint + Prettier)
  |-- Unit Tests (Jest with coverage)
  |-- Build & Push Docker image to GHCR
        |
Image stored at:
ghcr.io/muhammadharisali/ci-cd-blueprint:latest
        |
You deploy to Kubernetes with deploy.sh
```

## CI Pipeline Details

Automatically triggers on push to `main` or `develop`, and on all pull requests.

| Job | What It Does | If It Fails |
|-----|-------------|-------------|
| **Lint & Format** | Runs `npm run lint` and `npm run format:check` | PR blocked, fix code style |
| **Unit Tests** | Runs `npm run test:coverage` | PR blocked, fix broken tests |
| **Build & Push** | Builds Docker image, pushes to GHCR | PR blocked, check Dockerfile |

## Deploy to Kubernetes

### Prerequisites

- Kubernetes cluster running (EKS, GKE, minikube, etc.)
- `kubectl` configured to talk to your cluster
- `kustomize` installed

### Deploy to Development

```bash
./scripts/deploy.sh development ghcr.io/muhammadharisali/ci-cd-blueprint:sha-abc1234
```

Replace `sha-abc1234` with the actual image tag from GitHub Packages.

### What the Script Does

1. Updates the image tag in the Kustomize overlay
2. Applies Kubernetes manifests to the cluster
3. Waits for the deployment to roll out successfully

## Project Structure

```
ci-cd-blueprint/
|-- src/                          # NestJS application code
|   |-- health/                   # Health check module
|   |   |-- health.module.ts
|   |   |-- health.controller.ts
|   |-- app.module.ts             # Root module
|   |-- main.ts                   # Application entry point
|-- .github/
|   |-- workflows/
|   |   |-- ci.yml                # GitHub Actions CI pipeline
|-- infra/
|   |-- k8s/
|   |   |-- base/                 # Base Kubernetes manifests
|   |   |   |-- deployment.yaml   # Pod definition
|   |   |   |-- service.yaml      # Internal load balancer
|   |   |   |-- kustomization.yaml
|   |   |-- overlays/
|   |   |   |-- development/      # Dev environment config
|   |   |   |   |-- kustomization.yaml
|-- scripts/
|   |-- deploy.sh                 # Kubernetes deployment script
|-- Dockerfile                    # Multi-stage production build
|-- docker-compose.yml            # Local development stack
|-- package.json                  # Node.js dependencies & scripts
|-- README.md                     # This file
```

## Health Endpoints

Kubernetes uses these to manage your pods:

| Endpoint | Type | What Kubernetes Does If It Fails |
|----------|------|----------------------------------|
| `GET /health` | Liveness probe | Kills the pod and starts a new one |
| `GET /ready` | Readiness probe | Stops sending traffic until it recovers |

## Adding Staging and Production

Copy the development overlay and adjust:

```bash
# Create staging overlay
cp -r infra/k8s/overlays/development infra/k8s/overlays/staging

# Edit staging/kustomization.yaml
# - Change namespace to "staging"
# - Change namePrefix to "staging-"
# - Increase replicas to 2
# - Increase resource limits
```

Do the same for `production` with 3+ replicas and higher resource limits.

## Environment Variables

The app reads these from the environment:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | Server port |
| `NODE_ENV` | `production` | Environment mode |

Add more in `infra/k8s/base/deployment.yaml` under `env:`.

## Troubleshooting

### Docker build fails

```bash
# Clean build with no cache
docker build --no-cache -t ci-cd-blueprint:latest .
```

### Image not found in GHCR

- Check GitHub Actions logs for push errors
- Verify `Settings -> Actions -> General -> Workflow permissions` is set to "Read and write"

### kubectl not found

```bash
# macOS
brew install kubectl

# Or download directly
curl -LO "https://dl.k8s/release/$(curl -L -s https://dl.k8s/release/stable.txt)/bin/darwin/amd64/kubectl"
```

### kustomize not found

```bash
# macOS
brew install kustomize
```

## Next Steps

1. [ ] Add staging and production K8s overlays
2. [ ] Add a database (PostgreSQL) to docker-compose
3. [ ] Add monitoring (Prometheus metrics endpoint)
4. [ ] Add a CD workflow for automatic deployment
5. [ ] Add Terraform for infrastructure provisioning

## License

MIT
