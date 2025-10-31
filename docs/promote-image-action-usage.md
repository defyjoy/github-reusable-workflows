# Promote Docker Image Action

This composite action promotes Docker images from one environment to another by pulling the source image, retagging it for the target environment, and pushing it to the registry.

## Usage

### Basic Example - Promote from Dev to Staging

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
        uses: defyjoy/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: ${{ github.event.inputs.source-tag }}
          target-image: ghcr.io/owner/my-app
          target-tag: staging
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

### Promote with Additional Tags

```yaml
jobs:
  promote:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote to production with multiple tags
        uses: defyjoy/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: v1.2.3
          target-image: ghcr.io/owner/my-app
          target-tag: production
          additional-tags: 'stable,prod,latest'
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

### Promote Between Different Registries

```yaml
jobs:
  promote:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote from Docker Hub to GHCR
        uses: defyjoy/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: docker.io/myorg/my-app
          source-tag: dev-latest
          target-image: ghcr.io/owner/my-app
          target-tag: staging
          source-username: ${{ secrets.DOCKERHUB_USERNAME }}
          source-password: ${{ secrets.DOCKERHUB_TOKEN }}
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

### Promote with Custom Source Authentication

```yaml
jobs:
  promote:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote image
        uses: defyjoy/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/other-org/my-app
          source-tag: v1.0.0
          target-image: ghcr.io/owner/my-app
          target-tag: production
          source-username: ${{ secrets.SOURCE_USERNAME }}
          source-password: ${{ secrets.SOURCE_TOKEN }}
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

### Promote Without Pulling (Image Already Local)

```yaml
jobs:
  promote:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build image
        uses: defyjoy/github-reusable-workflows@v1
        with:
          image-name: my-app
          image-tag: build-123
          
      - name: Promote without pulling
        uses: defyjoy/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: build-123
          target-image: ghcr.io/owner/my-app
          target-tag: staging
          skip-pull: 'true'
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `source-image` | Source image name (full path including registry) | Yes | - |
| `source-tag` | Source image tag to promote | Yes | - |
| `target-image` | Target image name (full path including registry) | Yes | - |
| `target-tag` | Target image tag for promotion | Yes | - |
| `source-registry` | Source registry (e.g., ghcr.io, docker.io) | No | Extracted from source-image |
| `target-registry` | Target registry (e.g., ghcr.io, docker.io) | No | Extracted from target-image |
| `source-username` | Source registry username | No | Uses github.actor for GHCR |
| `source-password` | Source registry password or token | No | - |
| `target-username` | Target registry username | No | Uses github.actor for GHCR |
| `target-password` | Target registry password or token | No | - |
| `additional-tags` | Additional tags to apply (comma-separated) | No | - |
| `skip-pull` | Skip pulling source image | No | `false` |

## Outputs

| Output | Description |
|--------|-------------|
| `promoted-image` | Full path of promoted image (target-image:target-tag) |

## Features

- ✅ Promote images between environments (dev → staging → production)
- ✅ Support for multiple registries (GHCR, Docker Hub, etc.)
- ✅ Apply multiple tags during promotion
- ✅ Optional source image pulling
- ✅ Automatic registry extraction from image names
- ✅ Authentication support for both source and target registries

## Common Use Cases

### 1. Promote After Successful Tests

```yaml
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: echo "Tests pass"

  promote:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote to staging
        uses: defyjoy/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: ${{ github.sha }}
          target-image: ghcr.io/owner/my-app
          target-tag: staging
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Manual Promotion to Production

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to promote (e.g., v1.2.3)'
        required: true

jobs:
  promote:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote to production
        uses: defyjoy/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: ${{ github.event.inputs.version }}
          target-image: ghcr.io/owner/my-app
          target-tag: production
          additional-tags: 'stable,latest'
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

### 3. Promote Specific Build

```yaml
on:
  workflow_dispatch:
    inputs:
      build-number:
        description: 'Build number to promote'
        required: true

jobs:
  promote:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Promote build to staging
        uses: defyjoy/github-reusable-workflows/.github/actions/promote-image@v1
        with:
          source-image: ghcr.io/owner/my-app
          source-tag: build-${{ github.event.inputs.build-number }}
          target-image: ghcr.io/owner/my-app
          target-tag: staging
          target-password: ${{ secrets.GITHUB_TOKEN }}
```

## Notes

- **Registry Extraction**: If you don't specify `source-registry` or `target-registry`, the action will automatically extract it from the image name (e.g., `ghcr.io/owner/app` → registry is `ghcr.io`).

- **Authentication**: For GitHub Container Registry (GHCR), you can use `GITHUB_TOKEN` which is automatically available. For other registries, you'll need to provide credentials.

- **Skip Pull**: Use `skip-pull: 'true'` when the source image is already available locally (e.g., after a build step in the same job).

- **Additional Tags**: Use comma-separated values without spaces (e.g., `"stable,prod"`) for multiple tags.

