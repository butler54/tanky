# Home Assistant Integration

## Overview

tanky integrates with Home Assistant through OpenClaw's HA skill. Home Assistant runs on a **separate host** -- it is not deployed as a Quadlet container on the tanky system. The connection uses the Tailscale mesh network, so no port forwarding or firewall rules are needed. OpenClaw's HA skill communicates with the HA REST API using a long-lived access token.

## Architecture

The integration follows this network path:

```
tanky host (OpenClaw container)
  └── HA skill reads ~/.config/home-assistant/config.json (bind-mounted from host)
        └── HTTPS request over Tailscale
              └── Home Assistant instance (e.g. homeassistant.story-beta.ts.net)
                    └── REST API (/api/)
```

- The OpenClaw container runs on the tanky host with the HA skill installed at `/home/node/.openclaw/workspace/skills/home-assistant/`.
- The skill reads its configuration from `~/.config/home-assistant/config.json`, which is bind-mounted from the bootc host filesystem into the container.
- Requests go over the Tailscale tailnet to the HA instance's REST API.
- **No Home Assistant components run on the tanky host itself.**

## Setup

1. **Deploy Home Assistant** on a separate host (or use an existing instance).

2. **Connect both hosts to the same Tailscale network.** See `docs/tailscale.md` for Tailscale setup on tanky.

3. **Verify connectivity** from the tanky host:

   ```bash
   curl https://YOUR-HA-HOST.ts.net/api/ \
     -H "Authorization: Bearer YOUR-TOKEN"
   ```

   Expected response: `{"message": "API running."}`

4. **Create the config directory:**

   ```bash
   mkdir -p ~/.config/home-assistant
   ```

5. **Create the config file** at `~/.config/home-assistant/config.json`:

   ```json
   {
     "url": "https://YOUR-HA-HOST.ts.net",
     "token": "YOUR-LONG-LIVED-ACCESS-TOKEN"
   }
   ```

6. **Restart the OpenClaw container** to pick up the new config:

   ```bash
   systemctl --user restart openclaw
   ```

7. **Verify** the HA skill loads and can reach the HA instance.

## Creating a Long-Lived Access Token

1. Open your Home Assistant web UI.
2. Click your user profile (bottom-left corner).
3. Scroll to **Long-Lived Access Tokens** under the Security tab.
4. Click **Create Token** and give it a name (e.g. `openclaw`).
5. Copy the token -- it is shown only once.
6. Store the token in `~/.config/home-assistant/config.json` as shown above.

## Token Rotation

1. Create a new token in the HA web UI (follow the steps above).
2. Update `~/.config/home-assistant/config.json` with the new token value.
3. Restart OpenClaw:

   ```bash
   systemctl --user restart openclaw
   ```

4. Revoke the old token in the HA web UI (Profile -> Security -> Long-Lived Access Tokens -> delete the old entry).

## Reboot Persistence

The config file at `~/.config/home-assistant/config.json` lives on the bootc host filesystem, not inside a container. It persists across reboots and across `bootc switch` upgrades. The OpenClaw container bind-mounts this path, so no reconfiguration is needed after updates.

## Known Limitations

- **Missing jq in OpenClaw container** -- The `ha.sh` CLI wrapper inside the OpenClaw container requires `jq`, which is not installed. This means the CLI wrapper will not work. This is an upstream OpenClaw issue, not a tanky concern.
- **LLM inference server dependency** -- The OpenClaw agent requires an LLM inference server (e.g. `donnager`) to be running for the HA skill to process natural-language requests. Without it, the skill is installed but the agent cannot act on commands.

## References

- HA REST API documentation: https://developers.home-assistant.io/docs/api/rest/
- Tailscale setup: see `docs/tailscale.md`
- OpenClaw HA skill path (inside container): `/home/node/.openclaw/workspace/skills/home-assistant/`
- Config file path (on host): `~/.config/home-assistant/config.json`
