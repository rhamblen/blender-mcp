# Changelog

## 1.6.1 (fork, 2026-07-21)

Fork of [ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp) @ `da4e16d`, adapted for
Docker deployment on our own infra (same conventions as `pihole-mcp`):

- Add `streamable-http` transport support, selected via `MCP_TRANSPORT` env var
  (defaults to `stdio`, unchanged for local/Claude Desktop use).
- Add a `/health` endpoint for container healthchecks (streamable-http only).
- Add `docker/` packaging: `Dockerfile` (repo-root build context), `docker-compose.example.yml`,
  `.env.example`.
- Add `.github/workflows/docker.yml` to publish multi-arch images to GHCR on release.
- Add `docs/Deployment.md`, `server.json`.

No changes to the Blender addon (`addon.py`) or any tool behavior.
