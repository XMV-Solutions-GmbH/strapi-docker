# ARM64 / multi-arch contribution notes

This document describes the changes on the `feat/arm64-multiarch` branch and
the intent behind them. The goal is to open a PR against `strapi/strapi-docker`
once the changes are validated against a real ARM64 Kubernetes cluster.

## What this branch changes

The PR is intentionally minimal — only the changes needed to support ARM64.
CI system and tooling are left as-is (Travis CI remains Travis CI).

### 1. Alpine Dockerfile: add `vips-dev`

`base/alpine/Dockerfile` now installs `vips-dev`, which is required for
`sharp` (Strapi's image processing library) to work correctly on ARM64.

In practice, `sharp` ships pre-built ARM64 binaries via npm
(`@img/sharp-linux-arm64`) and does not need to compile — but having `vips-dev`
present in the base image makes it resilient to cases where the pre-built
binary is unavailable or stale, falling back to compiling against libvips
instead of failing.

### 2. Node versions: 10/12/14 → 20/22

Node 10, 12, and 14 are all end-of-life. Updated `bin/constants.js` to
Node 20 LTS (current) and Node 22 LTS. This is a separate improvement
bundled into the same PR since the old versions don't ship arm64 base
images at all.

## What this branch does NOT change

- **CI system**: Travis CI remains as-is. Migrating to GitHub Actions is a
  separate decision for the Strapi maintainers, not part of this PR.
- **Debian base image**: Only Alpine is affected. The Debian base image
  (`base/Dockerfile`) already pulls from `node:{version}` which is multi-arch
  — it should work on ARM64 without changes.
- **Strapi application logic**: No changes to `strapi/Dockerfile` or
  `strapi/docker-entrypoint.sh`.

## Validation status

- [ ] `docker buildx build --platform linux/arm64 ./base/alpine` completes
- [ ] `sharp` loads successfully on an ARM64 node (no "bindings not found")
- [ ] Strapi admin UI reachable on a Hetzner CAX node (ARM64, Ampere Altra)

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

# Inspect the manifest — both architectures should be listed
docker buildx imagetools inspect strapi-base-arm64-test:local
```

## Upstream PR plan

Once validated:
1. Sync fork with `git fetch upstream && git rebase upstream/master`
2. Open PR at https://github.com/strapi/strapi-docker
3. Reference issue #272 (ARM64 image support request)
4. Include validation evidence (build log, `docker buildx imagetools inspect` output)
