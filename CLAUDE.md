# CLAUDE.md

This file provides guidance to Claude Code when working with the tanky repository.

## What This Project Is

**tanky** is a custom bootc image that layers additional features on top of [tank-os](https://github.com/LobsterTrap/tank-os). It uses the bootc layering pattern: `FROM quay.io/rh-ee-chbutler/tank-os:latest`. Published to GHCR at `ghcr.io/butler54/tanky`.

The base tank-os image provides:
- Fedora bootc base
- OpenClaw gateway (rootless Podman)
- service-gator MCP server
- Basic provisioning (cloud-init, SSH)

tanky adds:
- **signal-cli** — Signal messaging integration
- **Tailscale** — Mesh networking and remote access
- **Home Assistant integration** — External HA access via OpenClaw skill
- **SELinux fixes** — Quadlet volume label corrections for persistent state

## Build Commands

### Local build

```bash
make build IMAGE_OWNER=butler54
```

The Containerfile expects the base image `quay.io/rh-ee-chbutler/tank-os:latest` to exist.

### Build QCOW2 disk image

Same as tank-os — use bootc-image-builder or Podman Desktop BootC extension. The resulting VM will have all tanky features.

## CI/CD Pipeline

The repo has the same CI/CD structure as tank-os:
- `.github/workflows/build-release.yml` — Multi-arch build + signing on git tags
- `.github/workflows/create-release.yml` — Semantic versioning on push to main
- `.github/workflows/pr.yaml` — PR validation builds
- `.github/workflows/commitlint.yml` — Conventional commit enforcement
- `.github/workflows/scorecard.yml` — OpenSSF security analysis

Releases are triggered by conventional commits on main. The pipeline builds, signs (cosign), and attests (SBOM, provenance) the tanky image.

## Layering Architecture

The Containerfile follows this pattern:

```dockerfile
FROM quay.io/rh-ee-chbutler/tank-os:latest
# Install Tailscale, signal-cli, java
# COPY rootfs/ (overlays on top of base)
# Enable tailscaled.service
```

**Rootfs overlay** contains only files that add to or override the base:
- `etc/systemd/user/signal-cli.service` — signal-cli daemon unit
- `etc/containers/systemd/users/1000/*.container` — Quadlet SELinux fixes

## Important Notes

- **Base image dependency**: tanky builds require `quay.io/rh-ee-chbutler/tank-os:latest` to exist. If tank-os hasn't been released yet, the build will fail.
- **Secrets**: Same as tank-os — use Podman secrets for API keys, managed via `tank-openclaw-secrets` helper (provided by base image).
- **Upstream contributions**: tanky is fork-specific. Features that should go upstream belong in tank-os, not here.
- **Conventional commits**: PRs must have conventional commit titles (feat:, fix:, chore:, docs:, etc.) or commitlint will block them.

## Testing Locally

1. Build tank-os base image (or use published one)
2. Build tanky layered image
3. Use Podman Desktop BootC extension to create disk image
4. Boot VM and verify features work

SSH into the VM:
```bash
ssh openclaw@<vm-ip>
systemctl --user status signal-cli  # if registered
systemctl status tailscaled
openclaw status --deep
```

## File Structure

```
tanky/
├── bootc/
│   ├── Containerfile          # Layers on tank-os
│   └── rootfs/                # Overlay files
├── .github/workflows/         # CI/CD pipelines
├── docs/                      # Feature-specific docs
├── examples/cloud-init/       # Provisioning examples
├── Makefile                   # Build helpers
└── commitlint.config.js       # Conventional commit rules
```
