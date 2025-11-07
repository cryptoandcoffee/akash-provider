# Akash Provider - Docker Images

**Automated Docker image builds for [Akash Network Provider](https://github.com/akash-network/provider)**

This repository provides pre-built Docker images of the Akash Provider service, automatically tracking and building from the upstream repository.

## 🐳 Docker Images

Images are published to two registries:

### Docker Hub
```bash
docker pull cryptoandcoffee/akash-network-provider:latest
docker pull cryptoandcoffee/akash-network-provider:main
docker pull cryptoandcoffee/akash-network-provider:nightly
```

### GitHub Container Registry
```bash
docker pull ghcr.io/cryptoandcoffee/akash-network-provider:latest
docker pull ghcr.io/cryptoandcoffee/akash-network-provider:main
docker pull ghcr.io/cryptoandcoffee/akash-network-provider:nightly
```

## 📦 Available Tags

| Tag | Description | Update Frequency |
|-----|-------------|------------------|
| `latest` | Latest stable release from upstream | When new upstream release is published |
| `v*` (e.g., `v0.10.1`) | Specific version tags | When new upstream release is published |
| `main` | Latest from upstream main branch | Nightly (2 AM UTC) |
| `nightly` | Latest nightly build | Nightly (2 AM UTC) |
| `nightly-YYYYMMDD` | Dated nightly build | Nightly (2 AM UTC) |

## 🔄 Automation

This repository uses GitHub Actions workflows to:

- **Monitor** the upstream [akash-network/provider](https://github.com/akash-network/provider) repository
- **Clone** upstream source code at runtime (no fork maintenance)
- **Build** static binaries with CGO disabled
- **Package** minimal Alpine-based container images using [crane](https://github.com/google/go-containerregistry)
- **Publish** to both Docker Hub and GitHub Container Registry

### Build Schedule

- **Nightly**: Every day at 2 AM UTC
- **On-Demand**: Manual workflow dispatch available
- **Smart Building**: Only builds when upstream changes are detected

## 🚀 Usage

### Quick Start

```bash
# Run the provider service
docker run -d \
  --name akash-provider \
  cryptoandcoffee/akash-network-provider:latest \
  run
```

### With Custom Configuration

```bash
docker run -d \
  --name akash-provider \
  -v /path/to/config:/config \
  cryptoandcoffee/akash-network-provider:latest \
  run --config /config/provider.yaml
```

### Using docker-compose

```yaml
version: '3.8'
services:
  provider:
    image: cryptoandcoffee/akash-network-provider:latest
    container_name: akash-provider
    restart: unless-stopped
    volumes:
      - ./config:/config
    command: run --config /config/provider.yaml
```

## 📖 Documentation

For complete Akash Provider documentation, configuration options, and deployment guides, please refer to:

- **Upstream Repository**: https://github.com/akash-network/provider
- **Akash Documentation**: https://docs.akash.network/
- **Provider Documentation**: https://docs.akash.network/providers

## ⚙️ Build Information

### Image Specifications

- **Base**: Alpine 3.18
- **Architecture**: AMD64
- **Binary**: Statically linked (CGO disabled)
- **Build Tags**: `osusergo,netgo,static_build`
- **Entry Point**: `/usr/local/bin/provider-services`
- **Default Command**: `run`

### Build Process

1. Workflows clone upstream repository at the specified ref
2. Detect required Go version from upstream `go.mod`
3. Build static binary with security-hardened flags
4. Create minimal OCI image using Google's crane tool
5. Push to Docker Hub and GHCR with appropriate tags

## 🤝 Contributing

This repository is purely for automated builds. For issues, feature requests, or contributions related to the Akash Provider itself, please visit the upstream repository:

👉 **[akash-network/provider](https://github.com/akash-network/provider)**

## 📝 License

This repository inherits the license from the upstream Akash Provider project.

The Akash Provider is licensed under the Apache License 2.0. See the [upstream LICENSE](https://github.com/akash-network/provider/blob/main/LICENSE) for details.

## 🔗 Links

- **Upstream Provider**: https://github.com/akash-network/provider
- **Akash Network**: https://akash.network/
- **Docker Hub**: https://hub.docker.com/r/cryptoandcoffee/akash-network-provider
- **GHCR**: https://github.com/cryptoandcoffee/akash-provider/pkgs/container/akash-network-provider

---

**Note**: This is an automated build repository. It does not contain source code - workflows clone upstream at runtime for each build.

---

**Last Updated**: 2025-11-07 08:21 UTC
**Latest Release**: `v0.10.1` | **Main Branch**: `aa632cd`

