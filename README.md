# n8n Enterprise Docker Build Workflow

This repository contains a GitHub Actions workflow that automatically builds and pushes n8n Docker images with enterprise features enabled to GitHub Container Registry (GHCR).

## Workflow Overview

The workflow (`build.yml`) performs the following steps:

1. **Clone n8n Repository**: Clones the official n8n repository from `https://github.com/n8n-io/n8n` and checks out the latest stable release (or specified version)
2. **Setup Environment**: Installs Node.js 22 and pnpm package manager
3. **Apply Enterprise Bypass**: Copies and runs the `bypass.sh` script to enable enterprise features
4. **Install Dependencies**: Runs `pnpm install --frozen-lockfile`
5. **Build Project**: Executes `pnpm run build` to compile n8n
6. **Build Multi-Platform Docker Image**: Uses Docker Buildx to build images for both `linux/amd64` and `linux/arm64` architectures
7. **Push to GHCR**: Tags and pushes the multi-platform image to GitHub Container Registry

## Features

- ✅ **Multi-Platform Support**: Builds for both AMD64 and ARM64 architectures (matches official n8n platforms)
- ✅ **Enterprise Features**: Automatically applies license bypass to enable all enterprise features
- ✅ **Version Control**: Supports building specific n8n versions or latest stable release
- ✅ **Smart Caching**: Uses GitHub Actions cache for faster builds
- ✅ **Flexible Username**: Uses repository owner as username, with fallback to `davodm`

## Triggers

The workflow runs on:
- Manual trigger via GitHub Actions UI (`workflow_dispatch`) with optional version input

## Docker Image Tags

The workflow creates multiple tags for the Docker image. The username is automatically determined from the repository owner (falls back to `davodm` if not available):

- `ghcr.io/<username>/n8n:enterprise` - Main enterprise tag
- `ghcr.io/<username>/n8n:latest` - Latest build tag
- `ghcr.io/<username>/n8n:<n8n-version>` - Specific n8n version (e.g., `n8n@1.70.0`)
- `ghcr.io/<username>/n8n:<commit-hash>` - Specific n8n commit hash

**Example** (if repository owner is `davodm`):
- `ghcr.io/davodm/n8n:enterprise`
- `ghcr.io/davodm/n8n:latest`
- `ghcr.io/davodm/n8n:n8n@1.70.0`
- `ghcr.io/davodm/n8n:abc1234`

## Platform Support

The workflow builds multi-platform images supporting:
- `linux/amd64` - Intel/AMD 64-bit processors
- `linux/arm64` - ARM 64-bit processors (including Apple Silicon)

This matches the official n8n Docker image platform support.

## Required Permissions

The workflow requires the following permissions (already configured):
- `contents: read` - To checkout the repository
- `packages: write` - To push Docker images to GHCR

## Authentication

The workflow uses the `GITHUB_TOKEN` automatically provided by GitHub Actions. No additional secrets need to be configured for GHCR access.

## Usage

### Manual Trigger

1. Go to the **Actions** tab in your GitHub repository
2. Select **Build and Push n8n Enterprise Docker Image** workflow
3. Click **Run workflow**
4. Optionally specify a n8n version tag (leave empty for latest stable)
5. Click **Run workflow** button

### Building Specific Versions

When manually triggering the workflow, you can specify a specific n8n version tag (e.g., `n8n@1.70.0`). If left empty, it will build the latest stable release.

## Pulling the Image

To use the built Docker image (replace `<username>` with your repository owner or `davodm`):

```bash
# Pull the enterprise image
docker pull ghcr.io/<username>/n8n:enterprise

# Run n8n
docker run -d --name n8n -p 5678:5678 ghcr.io/<username>/n8n:enterprise
```

### Using Docker Compose

Update `compose.yml` with your username:

```yaml
services:
  n8n:
    image: ghcr.io/<username>/n8n:enterprise
    ports:
      - "5678:5678"
    # ... rest of configuration
```

Then run:
```bash
docker compose up -d
```

## Image Username

The Docker image username is automatically determined:
- **Primary**: Uses the GitHub repository owner (e.g., if repo is `davodm/n8n`, username is `davodm`)
- **Fallback**: If repository owner is not available, defaults to `davodm`

This ensures the image path matches your GitHub username while providing a sensible default.

## Troubleshooting

- **Check the Actions tab** in your GitHub repository for detailed build logs
- **Ensure Actions are enabled** in your repository settings
- **Verify GHCR access** - Make sure your account has permission to publish packages
- **Check repository visibility** - Private repos may need additional permissions
- **Verify bypass.sh exists** - The workflow requires `bypass.sh` in the repository root
- **Check Dockerfile location** - The workflow looks for `docker/images/n8n/Dockerfile` in the n8n source

## Monitoring

The workflow includes:
- **Detailed logging** for each build step
- **Image verification** to confirm multi-platform support
- **Cleanup steps** to prevent disk space issues
- **Build caching** for faster subsequent builds

## Differences from Official n8n Image

This workflow creates a modified version of n8n with:
- All enterprise features enabled (via license bypass)
- Same platform support as official n8n (`linux/amd64`, `linux/arm64`)
- Custom tagging system for version tracking
- Built from the same source code as official n8n

## License

This workflow is for development and testing purposes. Please ensure you comply with n8n's license terms when using this modified image.
