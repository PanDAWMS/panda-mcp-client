# panda-mcp-client

PanDA MCP client tools for Claude Desktop, LM Studio, and similar LLM clients.

## Tools

| Command | Purpose |
|---|---|
| `get-panda-token` | Device-flow OIDC login — run once to obtain and cache a token |
| `panda-mcp-proxy` | stdio↔HTTPS/SSE proxy with automatic token refresh |

The proxy solves the core limitation of the older `panda_mcp_wrapper.sh`: that script
bakes the token into `mcp-remote` at startup, so sessions break after 15 minutes.
This proxy refreshes the token transparently on every request.

## Requirements

- Python 3.10+
- [uv](https://docs.astral.sh/uv/) (recommended) or pip

## Quick start

### 1. Get a token

```bash
uvx --from panda-mcp-client get-panda-token
```

Follow the browser prompt. The token is saved to `~/.panda_id_token`.

### 2. Configure your LLM client

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "panda-mcp": {
      "command": "uvx",
      "args": ["--from", "panda-mcp-client", "panda-mcp-proxy"]
    }
  }
}
```

**LM Studio** (`mcp.json`):

```json
{
  "mcpServers": {
    "panda-mcp": {
      "command": "uvx",
      "args": ["--from", "panda-mcp-client", "panda-mcp-proxy"]
    }
  }
}
```

## Environment variables

All variables are optional; the defaults match those of `panda_mcp_wrapper.sh`.

| Variable | Default | Description |
|---|---|---|
| `PANDA_SERVER` | `https://pandaserver.cern.ch:25443` | PanDA server URL |
| `VO` | `atlas` | Virtual organisation |
| `TOKEN_FILE` | `~/.panda_id_token` | Token cache file |
| `MCP_URL` | `https://aipanda120.cern.ch:8443/mcp/` | Remote MCP server URL |
| `SSL_CERT_FILE` | — | Path to CA bundle (custom CAs) |
| `REQUESTS_CA_BUNDLE` | — | Alternative to `SSL_CERT_FILE` |

## Token refresh

`panda-mcp-proxy` refreshes the token automatically using the `refresh_token` stored
in `~/.panda_id_token`. Refresh happens 5 minutes before expiry. If the refresh token
is also expired, re-run `get-panda-token`.