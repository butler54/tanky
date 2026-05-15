# tanky

Custom layered bootc image built on top of [tank-os](https://github.com/LobsterTrap/tank-os).

## What This Is

`tanky` is a Fedora bootc image that layers signal-cli, Tailscale, and Home Assistant integration on top of the base `tank-os` image. It provides a complete OpenClaw appliance with optional messaging, remote access, and smart home integration.

**Base Image:** `quay.io/rh-ee-chbutler/tank-os:latest`  
**Layered Image:** `quay.io/rh-ee-chbutler/tanky:latest`

## Features Added by tanky

- **signal-cli** — Signal messaging for OpenClaw channels
- **Tailscale** — Secure remote access and mesh networking
- **Home Assistant integration** — Smart home control via OpenClaw skills
- **SELinux fixes** — Quadlet volume label fixes for persistent state

See `docs/` for setup instructions for each feature.

## Quick Start

```bash
# Build locally
make build IMAGE_REGISTRY=quay.io REGISTRY_USER=rh-ee-chbutler

# Or pull the published image
podman pull quay.io/rh-ee-chbutler/tanky:latest
```

## Documentation

- `docs/signal.md` — Signal messaging setup
- `docs/tailscale.md` — Tailscale mesh networking
- `docs/home-assistant.md` — Home Assistant integration
- `docs/private-registries.md` — Private registry authentication

For base tank-os documentation, see https://github.com/LobsterTrap/tank-os
