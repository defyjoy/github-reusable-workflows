# Docker Build Action

This composite action builds Docker images with support for multiple platforms, caching, and registry pushing.

## Usage

### Basic Example

```yaml
name: Build Docker Image

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build Docker image
        uses: defyjoy/github-reusable-workflows@v1
        with:
          image-name: my-app
          image-tag: latest
```

### Advanced Example with Registry Push

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build and push Docker image
        uses: defyjoy/github-reusable-workflows@v1
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

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `dockerfile` | Path to Dockerfile | No | `Dockerfile` |
| `context` | Build context path | No | `.` |
| `image-name` | Docker image name | Yes | - |
| `image-tag` | Docker image tag | No | `latest` |
| `registry` | Docker registry (e.g., ghcr.io, docker.io) | No | Uses GHCR if not specified |
| `push` | Whether to push the image to registry | No | `false` |
| `platforms` | Comma-separated list of platforms (e.g., linux/amd64,linux/arm64) | No | - |
| `build-args` | Build arguments as JSON object string | No | - |
| `cache-from` | Cache sources | No | - |
| `cache-to` | Cache export | No | - |
| `labels` | Labels as JSON object string | No | - |

## Secrets

Since this is a composite action, secrets should be passed as input values. For sensitive data like passwords, use GitHub secrets:

| Input | Description | Required |
|--------|-------------|----------|
| `registry-username` | Docker registry username | Only if pushing to external registry |
| `registry-password` | Docker registry password or token | Only if pushing |

## Features

- ✅ Multi-platform builds (amd64, arm64, etc.)
- ✅ Build caching for faster builds
- ✅ Automatic metadata extraction and tagging
- ✅ Support for build arguments and labels
- ✅ GitHub Container Registry (GHCR) and custom registry support
- ✅ Automatic login to GHCR using GITHUB_TOKEN

## Image Tagging

The workflow automatically generates tags based on:
- Branch names (for branches)
- PR numbers (for pull requests)
- Semantic versions (for tags like v1.0.0)
- Commit SHA (with prefix)
- Custom tag from `image-tag` input

## Examples

### Build for Single Platform

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build and push Docker image
        uses: defyjoy/github-reusable-workflows@v1
        with:
          image-name: api-server
          push: 'true'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### Build with Custom Dockerfile Path

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build Docker image
        uses: defyjoy/github-reusable-workflows@v1
        with:
          dockerfile: ./docker/Dockerfile
          image-name: backend-service
```

### Build with Build Arguments

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build Docker image
        uses: defyjoy/github-reusable-workflows@v1
        with:
          image-name: web-app
          build-args: '{"NODE_ENV":"production","API_URL":"https://api.example.com"}'
```

### Multi-platform Build

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build and push multi-platform Docker image
        uses: defyjoy/github-reusable-workflows@v1
        with:
          image-name: multi-arch-app
          platforms: linux/amd64,linux/arm64,linux/arm/v7
          push: 'true'
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### Build with Registry Cache

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build with cache
        uses: defyjoy/github-reusable-workflows@v1
        with:
          image-name: cached-build
          push: 'true'
          cache-from: type=registry,ref=ghcr.io/owner/cached-build:buildcache
          cache-to: type=registry,ref=ghcr.io/owner/cached-build:buildcache,mode=max
          registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### Using Action from Same Repository

If using the action from the same repository:

```yaml
steps:
  - name: Build Docker image
    uses: ./.github/workflows@v1
    with:
      image-name: my-app
      push: 'true'
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

Or reference it directly:

```yaml
steps:
  - name: Build Docker image
    uses: ./@v1
    with:
      image-name: my-app
      push: 'true'
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

