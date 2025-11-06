# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Akash Provider Daemon - a Go-based service that enables providers to participate in the Akash Network decentralized cloud marketplace. The provider listens to blockchain events and manages Kubernetes clusters to provision compute capacity based on won bids.

## Development Environment Setup

This project requires `direnv` for environment variable management. Run the following before any make commands:

```bash
# Install direnv if not already installed
brew install direnv  # macOS
# or for Linux: follow https://direnv.net/docs/installation.html

# Allow direnv for this project
direnv allow

# Set required environment variables
export AKASH_DIRENV_SET=1
export AP_ROOT=$(pwd)
export GOTOOLCHAIN=go1.21.0
```

## Common Development Commands

### Building
```bash
# Build all binaries
make bins

# Build provider-services binary specifically
go build -o provider-services ./cmd/provider-services

# Build with specific tags (includes ledger support)
make BUILD_TAGS="osusergo,netgo,static_build,ledger" bins

# Build for release
make goreleaser-release GORELEASER_CONFIG=.goreleaser.yaml
```

### Testing
```bash
# Run unit tests
make test

# Run tests without cache
make test-nocache

# Run tests with race detection
make test-full

# Run specific test
go test -v ./bidengine/... -run TestBidEngineService

# Run integration tests (requires Kubernetes cluster)
make test-k8s-integration

# Run e2e tests (requires full environment)
make test-e2e-integration

# Generate test coverage report
make test-coverage
```

### Code Quality
```bash
# Run linter
make lint

# Format code
go fmt ./...

# Generate mocks
make mocks

# Update dependencies
go mod tidy
go mod download
```

### Running the Provider
```bash
# Basic provider run command (requires blockchain connection)
./provider-services run \
  --from <provider-key-name> \
  --cluster-k8s \
  --bid-price-strategy scale \
  --bid-price-cpu-scale 0.01 \
  --bid-price-memory-scale 0.0025 \
  --bid-price-storage-scale 0.00025 \
  --deployment-ingress-domain <your-domain.com>

# Check provider status
./provider-services status <provider-address>

# Query lease status
./provider-services lease-status --dseq <deployment-seq> --provider <address>

# Stream lease logs
./provider-services lease-logs --dseq <deployment-seq> --provider <address> --service <service-name>
```

## High-Level Architecture

The provider consists of four main services orchestrated through a pubsub event bus:

1. **BidEngine Service** (`bidengine/`)
   - Monitors blockchain for new orders via `EventOrderCreated` events
   - Evaluates orders against provider capabilities and pricing strategy
   - Places bids automatically based on configured pricing
   - Key files: `bidengine/service.go`, `bidengine/order.go`, `bidengine/pricing.go`

2. **Cluster Service** (`cluster/`)
   - Manages Kubernetes deployments for won leases
   - Tracks resource inventory and capacity via operators
   - Handles hostname/ingress management
   - State machine for deployment lifecycle (deploy-pending → deploy-active → deploy-complete)
   - Key files: `cluster/service.go`, `cluster/manager.go`, `cluster/kube/`

3. **Manifest Service** (`manifest/`)
   - Receives deployment manifests from tenants via REST API
   - Validates and matches manifests to active leases
   - Dispatches to cluster service for deployment
   - Key files: `manifest/service.go`, `manifest/manager.go`

4. **Gateway Service** (`gateway/`)
   - REST API (port 8443) and gRPC server (port 8444) with TLS
   - JWT authentication (ES256K) and mTLS support
   - Endpoints for manifest submission, lease queries, logs, and status
   - Key files: `gateway/rest/router.go`, `gateway/grpc/server.go`

## Event Flow

```
1. Order Created on Blockchain
   ↓
2. BidEngine evaluates and places bid
   ↓
3. If bid wins → Lease Created event
   ↓
4. Cluster Service reserves resources
   ↓
5. Manifest Service receives manifest from tenant
   ↓
6. Cluster Service deploys to Kubernetes
   ↓
7. Gateway provides status/logs/shell access
```

## Key Kubernetes Integration Points

- **Client Configuration**: Uses `cluster/kube/clientcommon/` for K8s client setup
- **Manifest Builder**: `cluster/kube/builder/` converts Akash manifests to K8s resources
- **Custom Operators**: Integrates with inventory, hostname, and IP operators via CRDs
- **Resources Created**: Namespace, Deployment/StatefulSet, Service, Ingress, NetworkPolicy, PVC

## Provider Configuration

Key configuration flags for `provider-services run`:
- `--cluster-k8s`: Enable Kubernetes cluster management
- `--bid-price-strategy`: Pricing strategy (scale, randomRange, shellScript)
- `--bid-price-*-scale`: Resource pricing multipliers
- `--deployment-ingress-domain`: Base domain for generated hostnames
- `--overcommit-pct-*`: Resource overcommit percentages
- `--cluster-public-hostname`: Public hostname for provider
- `--gateway-listen-address`: REST API listen address
- `--inventory-resource-poll-period`: How often to check K8s capacity

## Important Package Dependencies

- `pkg.akt.dev/node`: Akash blockchain node (may be local or remote)
- `github.com/boz/go-lifecycle`: Service lifecycle management
- `k8s.io/client-go`: Kubernetes client
- `github.com/spf13/cobra`: CLI framework
- `github.com/spf13/viper`: Configuration management

## Debugging Tips

1. Enable debug logging with `--log-level debug`
2. Monitor provider status via REST API: `curl https://<provider>:8443/status`
3. Check lease deployments: `kubectl get all -n lease-<dseq>-<gseq>-<oseq>`
4. View provider logs for specific subsystems (bidengine, cluster, manifest)
5. Use `provider-services lease-events` to stream lease lifecycle events

## Common Issues and Solutions

1. **"No direnv" error**: Install direnv and run `direnv allow`
2. **Provider not bidding**: Check provider attributes match order requirements
3. **Deployments stuck pending**: Verify Kubernetes cluster has sufficient resources
4. **Manifest rejected**: Ensure manifest resources match order specifications
5. **TLS errors**: Provider generates self-signed certs by default; use `--tls-cert-file` for custom certs