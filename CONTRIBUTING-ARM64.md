# ARM64 / multi-arch contribution notes

This document describes the changes on the `feat/arm64-multiarch` branch and
the intent behind them. The goal is to open a PR against `strapi/strapi-docker`
once the changes are validated against a real ARM64 Kubernetes cluster.

## What this branch changes

### 1. CI: Travis CI → GitHub Actions

The existing `.travis.yml` (Node 12, single-arch `docker build`) is replaced
by `.github/workflows/build-and-push.yml`, which:

- Uses `docker/setup-qemu-action` to enable ARM64 cross-compilation on
  GitHub-hosted AMD64 runners
- Uses `docker/setup-buildx-action` + `docker/build-push-action` for
  multi-platform builds
- Builds `strapi/base` (Debian + Alpine) and `strapi/strapi` for
  `linux/amd64,linux/arm64`
- Publishes to Docker Hub only on push to main (not on PRs)
- Uses GitHub Actions cache (`type=gha`) to speed up layer caching

### 2. Node versions: 10/12/14 → 20/22

Node 10, 12, and 14 are all end-of-life. Updated to Node 20 LTS (current)
and Node 22 LTS.

### 3. Alpine Dockerfile: add `vips-dev`

The Alpine base image now installs `vips-dev`, which is required for `sharp`
(Strapi's image processing library) to compile from source on ARM64 as a
fallback. In practice, `sharp` ships pre-built ARM64 binaries via npm
(`@img/sharp-linux-arm64`) and does not need to compile — but having `vips-dev`
present makes the image resilient to cases where the pre-built binary is
unavailable or stale.

## Validation status

- [ ] `docker buildx build --platform linux/arm64 ./base/alpine` completes
- [ ] `sharp` loads successfully on an ARM64 node (no "bindings not found")
- [ ] Strapi admin UI reachable on a Hetzner CAX node (ARM64, Ampere Altra)
- [ ] Image push to GHCR from GitHub Actions succeeds

Validation is running against the XMV Solutions basics cluster
(ARM64, Hetzner CAX). Results will be documented here before the upstream PR
is opened.

## How to test locally

```bash
# Requires Docker Desktop or Docker Engine with buildx
docker buildx create --use --name multiarch
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --build-arg NODE_VERSION=20-alpine \
  -t strapi-base-arm64-test:local \
  ./base/alpine

# Inspect the manifest to confirm both architectures
docker buildx imagetools inspect strapi-base-arm64-test:local
```

## Upstream PR plan

Once validated:
1. Sync fork with `git fetch upstream && git rebase upstream/main`
2. Open PR at https://github.com/strapi/strapi-docker
3. Reference issue #272 (ARM64 image request)
4. Include validation evidence (GitHub Actions run logs, `docker manifest inspect` output)
