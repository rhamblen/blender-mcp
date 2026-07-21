# Build context is the repo root (see docker-compose.example.yml); this file
# only COPYs pyproject.toml, main.py, and src/ from there.
FROM python:3.12-slim AS build
WORKDIR /build
COPY pyproject.toml README.md LICENSE main.py ./
COPY src/ src/
RUN pip install --no-cache-dir build && python -m build --wheel

FROM python:3.12-slim
WORKDIR /app
RUN groupadd -g 1001 appgroup && useradd -u 1001 -g appgroup -M -s /usr/sbin/nologin appuser
COPY --from=build /build/dist/*.whl /tmp/wheel/
RUN pip install --no-cache-dir /tmp/wheel/*.whl && rm -rf /tmp/wheel
USER appuser
ENV MCP_TRANSPORT=streamable-http \
    MCP_HOST=0.0.0.0 \
    MCP_PORT=8000 \
    BLENDER_MCP_DISABLE_TELEMETRY=true
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=5s --retries=3 --start-period=10s \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health', timeout=3)" || exit 1
CMD ["blender-mcp"]
