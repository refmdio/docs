# MCP Server

RefMD ships a dedicated Model Context Protocol (MCP) server that exposes your workspace to AI assistants. The service now supports **remote MCP connections**, so hosted tools such as Claude or Cursor can reach RefMD over the public internet instead of requiring a local tunnel.

## Remote MCP support

- **SSE & HTTP transports**: `/sse` streams events over Server-Sent Events while `/mcp` offers one-shot HTTP exchanges. Either works with remote-capable MCP clients.
- **PKCE OAuth flow**: Users approve access via a browser prompt and issue time-limited access and refresh tokens. The consent screen asks for a RefMD API token generated under **Profile → API tokens** to complete the OAuth exchange.
- **Persistent token storage**: Configure SQLite, PostgreSQL, or MySQL to keep OAuth sessions across restarts when running the server long term.
- **Hosted friendly**: Deploy behind HTTPS and point remote MCP clients at the exposed `/sse` endpoint.

## Configuration essentials

```bash
REFMD_API_BASE="https://refmd.example.com"
OAUTH_CLIENT_IDS="remote-client"
OAUTH_ALLOWED_REDIRECTS="https://assistant.example.com/oauth/callback"
MCP_DB_DRIVER="sqlite"
MCP_DB_SQLITE_PATH="./data/refmd-mcp.sqlite"
npm start
```

Set `REFMD_API_BASE` to the RefMD instance you want to expose. `OAUTH_CLIENT_IDS` and `OAUTH_ALLOWED_REDIRECTS` lock down who can complete the OAuth flow. Choose a database driver (`sqlite`, `postgres`, or `mysql`) so remote sessions survive restarts.

## Connecting remote clients

1. Deploy `mcp-server` somewhere reachable by your assistants (e.g. Fly.io, container platform, or your own VPS). Terminate TLS with a reverse proxy if needed.
2. Ensure the OAuth redirect URL used by the client is listed in `OAUTH_ALLOWED_REDIRECTS`.
3. Point the client at the hosted `/sse` endpoint. Example with Claude:
   ```bash
   claude mcp add --transport sse refmd https://your-domain.example.com/sse
   ```
4. Complete the browser-based OAuth approval by pasting a personal API token from **Profile → API tokens** in RefMD. The assistant can now browse and edit documents through the remote MCP connection.

## Running with Docker

RefMD ships production images, and the MCP server follows the same convention: build (or pull) the container and run it alongside your RefMD deployment.

```bash
# Build the image inside this repository
docker build -t refmd-mcp ./mcp-server

docker run -p 3334:3334 \
  -e REFMD_API_BASE="https://refmd.example.com" \
  -e OAUTH_CLIENT_IDS="remote-client" \
  -e OAUTH_ALLOWED_REDIRECTS="https://assistant.example.com/oauth/callback" \
  -e MCP_DB_DRIVER="sqlite" \
  -e MCP_DB_SQLITE_PATH="/data/refmd-mcp.sqlite" \
  -v refmd-mcp-data:/data \
  refmd-mcp
```

Mount a persistent volume (as shown above) so SQLite-based token storage survives restarts. Substitute PostgreSQL or MySQL variables when using a shared database service. If you host RefMD with docker-compose, add an MCP service with the same environment to keep everything within a single stack.

## Next steps

- See `mcp-server/README.md` in the repository for full environment variable reference and Docker usage.
- Integrate the MCP server container into your existing orchestration setup (e.g. docker-compose, Fly.io, Kubernetes) using the same environment variables and volume configuration shown above.
