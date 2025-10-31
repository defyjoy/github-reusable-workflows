# GitHub Reusable Workflows

A collection of reusable GitHub Actions for building and managing Docker images.

## Actions

This repository provides the following reusable actions:

### 🐳 Docker Build Action

A composite action for building and optionally pushing Docker images with support for multiple platforms, caching, and custom configurations.

**Location:** Root-level `action.yml`  
**Usage:** `your-org/github-reusable-workflows@v1`

### 🚀 Promote Image Action

A composite action for promoting Docker images from one environment to another (e.g., dev → staging → production).

**Location:** `.github/actions/promote-image/action.yml`  
**Usage:** `your-org/github-reusable-workflows/.github/actions/promote-image@v1`

## Quick Start

### Building Docker Images

```yaml
name: Build Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build and push Docker image
        uses: your-org/github-reusable-workflows@v1
        with:
          image-name: my-app
          push: 'true'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### Promoting Images

```yaml
name: Promote to Staging

on:
  workflow_dispatch:
    inputs:
      source-tag:
        description: 'Source tag to promote'
        required: true

jobs:
  promote:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote image to staging
        uses: your-org/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: ${{ github.event.inputs.source-tag }}
          target-image: ghcr.io/owner/my-app
          target-tag: staging
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

## Docker Build Action

### Features

- ✅ Multi-platform builds (amd64, arm64, etc.)
- ✅ Build caching for faster builds
- ✅ Automatic metadata extraction and tagging
- ✅ Support for build arguments and labels
- ✅ GitHub Container Registry (GHCR) and custom registry support
- ✅ Automatic login to GHCR using GITHUB_TOKEN

### Basic Usage

```yaml
steps:
  - name: Build Docker image
    uses: your-org/github-reusable-workflows@v1
    with:
      image-name: my-app
      image-tag: latest
```

### Advanced Usage

```yaml
steps:
  - name: Build and push multi-platform image
    uses: your-org/github-reusable-workflows@v1
    with:
      dockerfile: ./Dockerfile.prod
      context: .
      image-name: my-app
      image-tag: latest
      registry: ghcr.io
      push: 'true'
      platforms: linux/amd64,linux/arm64
      build-args: '{"NODE_ENV":"production","VERSION":"1.0.0"}'
      labels: '{"maintainer":"team@example.com","version":"1.0.0"}'
      cache-from: type=registry,ref=ghcr.io/owner/my-app:buildcache
      cache-to: type=registry,ref=ghcr.io/owner/my-app:buildcache,mode=max
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `dockerfile` | Path to Dockerfile | No | `Dockerfile` |
| `context` | Build context path | No | `.` |
| `image-name` | Docker image name | Yes | - |
| `image-tag` | Docker image tag | No | `latest` |
| `registry` | Docker registry (e.g., ghcr.io, docker.io) | No | Uses GHCR if not specified |
| `push` | Whether to push the image to registry | No | `false` |
| `platforms` | Comma-separated list of platforms | No | - |
| `build-args` | Build arguments as JSON object string | No | - |
| `cache-from` | Cache sources | No | - |
| `cache-to` | Cache export | No | - |
| `labels` | Labels as JSON object string | No | - |
| `registry-username` | Docker registry username | No | - |
| `registry-password` | Docker registry password or token | No | - |

For detailed documentation, see [Docker Build Action Documentation](docs/docker-build-workflow-usage.md).

## Promote Image Action

### Features

- ✅ Promote images between environments (dev → staging → production)
- ✅ Support for multiple registries (GHCR, Docker Hub, etc.)
- ✅ Apply multiple tags during promotion
- ✅ Optional source image pulling
- ✅ Automatic registry extraction from image names
- ✅ Authentication support for both source and target registries

### Basic Usage

```yaml
steps:
  - name: Promote image to staging
    uses: your-org/github-reusable-workflows/.github/actions/promote-image@v1
    with:
      source-image: ghcr.io/owner/my-app
      source-tag: v1.2.3
      target-image: ghcr.io/owner/my-app
      target-tag: staging
      target-password: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Usage

```yaml
steps:
  - name: Promote to production with multiple tags
    uses: your-org/github-reusable-workflows/.github/actions/promote-image@v1
    with:
      source-image: ghcr.io/owner/my-app
      source-tag: v1.2.3
      target-image: ghcr.io/owner/my-app
      target-tag: production
      additional-tags: 'stable,prod,latest'
      target-password: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `source-image` | Source image name (full path including registry) | Yes | - |
| `source-tag` | Source image tag to promote | Yes | - |
| `target-image` | Target image name (full path including registry) | Yes | - |
| `target-tag` | Target image tag for promotion | Yes | - |
| `source-registry` | Source registry | No | Extracted from source-image |
| `target-registry` | Target registry | No | Extracted from target-image |
| `source-username` | Source registry username | No | Uses github.actor for GHCR |
| `source-password` | Source registry password or token | No | - |
| `target-username` | Target registry username | No | Uses github.actor for GHCR |
| `target-password` | Target registry password or token | No | - |
| `additional-tags` | Additional tags to apply (comma-separated) | No | - |
| `skip-pull` | Skip pulling source image | No | `false` |

For detailed documentation, see [Promote Image Action Documentation](docs/promote-image-action-usage.md).

## Common Workflows

### Build and Promote Pipeline

```yaml
name: Build and Promote

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      promote-to:
        description: 'Environment to promote to'
        required: false
        type: choice
        options:
          - staging
          - production

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build Docker image
        uses: your-org/github-reusable-workflows@v1
        with:
          image-name: my-app
          image-tag: ${{ github.sha }}
          push: 'true'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
        id: build

  promote:
    needs: build
    if: ${{ github.event_name == 'workflow_dispatch' && github.event.inputs.promote-to }}
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote image
        uses: your-org/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: ${{ github.sha }}
          target-image: ghcr.io/owner/my-app
          target-tag: ${{ github.event.inputs.promote-to }}
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

### Build on Push, Promote Manually

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      tag:
        description: 'Tag to promote'
        required: true

jobs:
  build:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build and push
        uses: your-org/github-reusable-workflows@v1
        with:
          image-name: my-app
          image-tag: ${{ github.sha }}
          push: 'true'
          registry-password: ${{ secrets.GITHUB_TOKEN }}

  promote-staging:
    if: github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote to staging
        uses: your-org/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: ${{ github.event.inputs.tag }}
          target-image: ghcr.io/owner/my-app
          target-tag: staging
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

## Requirements

- GitHub Actions runner with Docker support (e.g., `ubuntu-latest`)
- Appropriate permissions:
  - `contents: read` - to checkout repository
  - `packages: write` - to push images to GHCR (if pushing)

## Authentication

### GitHub Container Registry (GHCR)

For GHCR, you can use the built-in `GITHUB_TOKEN`:

```yaml
registry-password: ${{ secrets.GITHUB_TOKEN }}
```

The action will automatically use `github.actor` as the username for GHCR.

### Other Registries

For other registries (Docker Hub, etc.), provide credentials:

```yaml
registry-username: ${{ secrets.DOCKERHUB_USERNAME }}
registry-password: ${{ secrets.DOCKERHUB_TOKEN }}
```

## Versioning

This repository uses semantic versioning. Reference specific versions or use tags:

- `@v1` - Latest v1.x.x release
- `@v1.0.0` - Specific version
- `@main` - Latest from main branch (for testing)

## Documentation

- [Docker Build Action Documentation](docs/docker-build-workflow-usage.md)
- [Promote Image Action Documentation](docs/promote-image-action-usage.md)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

See [LICENSE](LICENSE) file for details.

