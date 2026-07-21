# Deployment

The recommended path is to pull the prebuilt multi-arch image from GHCR — no
local build. Images are published on every release by
[`.github/workflows/docker.yml`](../.github/workflows/docker.yml).

```bash
docker pull ghcr.io/rhamblen/blender-mcp:1.6.1   # or :latest
```

## Prerequisite: the Blender addon

This container is only the MCP-protocol server. It does **not** contain
Blender or the addon — install `addon.py` into Blender itself (on whatever
host/container Blender runs on) and enable it there, same as upstream. The
addon opens a local command socket (default port `9876`) inside Blender's own
process; make sure that port is reachable from wherever this container runs
(open/forward it if Blender is in its own container, e.g. on UR1).

## Docker Compose

The build/deploy files live under [`docker/`](../docker):

| File | Purpose |
|---|---|
| `docker/Dockerfile` | Multi-stage build (build context is the **repo root**). |
| `docker/docker-compose.example.yml` | Example stack: pulls the GHCR image, publishes to `127.0.0.1` only, hardened. |
| `docker/.env.example` | Template for `BLENDER_HOST`/`BLENDER_PORT` and the published MCP port. |

### Steps

```bash
cd docker
cp docker-compose.example.yml docker-compose.yml
cp .env.example .env
# Edit .env: BLENDER_HOST/BLENDER_PORT must point at wherever the Blender
# addon's socket is reachable (e.g. the UR1 host running Blender).

docker compose pull
docker compose up -d
```

## Building from source

```bash
docker build -f docker/Dockerfile -t blender-mcp:local .
```

## Verify a deploy

```bash
curl http://127.0.0.1:8877/health
# {"ok":true,"service":"blender-mcp","version":"1.6.1"}

docker ps --filter name=blender-mcp --format '{{.ID}}  {{.Image}}  {{.Status}}'
```

Then connect your MCP client:

```json
{ "mcpServers": { "blender": { "url": "http://127.0.0.1:8877/mcp" } } }
```

If your Claude client runs on a different machine than this container, put a
reverse proxy or Tailscale in front — the same pattern used for the other MCP
deployments here (e.g. `pihole-mcp`) — rather than publishing the port to the
LAN directly.

## Networking

- **Blender in its own container on the same host** — a bridge container
  reaching another container's published port works normally as long as that
  port is actually published/forwarded (not just bound to the Blender
  container's internal loopback). If Blender's container only exposes a
  streamed-desktop port (e.g. noVNC/WebRTC), the addon socket port needs its
  own separate forward — it is not the same thing as the desktop-streaming
  port.
- **Blender on a different host** — reachable normally over the LAN, no
  special setup, same as any other `BLENDER_HOST`.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Could not connect to Blender. Make sure the Blender addon is running.` | `BLENDER_HOST`/`BLENDER_PORT` unreachable from this container, or the addon isn't enabled/running in Blender. | Confirm the addon is enabled in Blender's UI and its socket port is open/forwarded; test with `nc -zv <BLENDER_HOST> <BLENDER_PORT>` from inside the container. |
| Container healthy but MCP client can't connect | `MCP_TRANSPORT` not set to `streamable-http`, or client pointed at the wrong path. | Confirm `MCP_TRANSPORT=streamable-http` is set and the client URL ends in `/mcp` (not `/`). |
