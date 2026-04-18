# whoop-mcp-server

Local MCP server wrapping the [WHOOP Developer API v2](https://developer.whoop.com/). Exposes cycles, recovery, sleep, workouts, profile, and body measurement as typed tools for any MCP-compatible client (Claude Code, Claude Desktop, Cursor, Zed).

- Stdio transport, no network listening after OAuth
- OAuth2 with offline refresh; tokens stored at `~/.whoop-mcp/tokens.json` with `0600` perms
- Zero runtime deps beyond `@modelcontextprotocol/sdk` + `zod`
- 11 read-only tools, all annotated `readOnlyHint: true`

## Setup

### 1. Register a WHOOP developer app

1. Go to [developer.whoop.com](https://developer.whoop.com/), sign in, create a new app.
2. Add redirect URI: `http://localhost:4567/callback`
3. Enable these scopes: `offline`, `read:profile`, `read:body_measurement`, `read:cycles`, `read:recovery`, `read:sleep`, `read:workout`
4. Copy the Client ID and Client Secret.

### 2. Install

```bash
git clone https://github.com/<you>/whoop-mcp-server.git
cd whoop-mcp-server
pnpm install           # or npm install
cp .env.example .env   # fill in WHOOP_CLIENT_ID + WHOOP_CLIENT_SECRET
pnpm run build
```

### 3. Authorize (one-time)

```bash
pnpm run auth
```

Opens your browser to WHOOP's OAuth consent. On success, tokens land at `~/.whoop-mcp/tokens.json`. The server auto-refreshes access tokens on the fly (refresh tokens rotate; the store is rewritten on every refresh).

**Cloudflare note:** `id.whoop.com` sits behind Cloudflare's bot check. If you run the auth flow inside a headless or automated browser, Cloudflare will block login with a silent loop. Open the printed URL in a regular Chrome/Safari window instead.

### 4. Register with your MCP client

**Claude Code** — edit `~/.claude.json` (or run `claude mcp add`):

```json
{
  "mcpServers": {
    "whoop": {
      "type": "stdio",
      "command": "node",
      "args": ["/absolute/path/to/whoop-mcp-server/dist/index.js"],
      "env": {
        "WHOOP_CLIENT_ID": "…",
        "WHOOP_CLIENT_SECRET": "…"
      }
    }
  }
}
```

**Claude Desktop** — same shape, in `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows).

Restart the client. Try: _"Use whoop_get_profile"_ or _"What was my recovery last night?"_

### 5. Smoke-test (optional)

```bash
pnpm exec tsx scripts/smoke-test.ts
```

Exercises all 11 tools against the live API. Expects `~/.whoop-mcp/tokens.json` to exist.

## Tools

| Tool                         | Scope                   | Purpose                        |
| ---------------------------- | ----------------------- | ------------------------------ |
| `whoop_get_profile`          | `read:profile`          | User name, email, user_id      |
| `whoop_get_body_measurement` | `read:body_measurement` | Height, weight, max HR         |
| `whoop_list_cycles`          | `read:cycles`           | Paginated physiological cycles |
| `whoop_get_cycle`            | `read:cycles`           | Single cycle by ID             |
| `whoop_list_recoveries`      | `read:recovery`         | Paginated recovery scores      |
| `whoop_get_cycle_recovery`   | `read:recovery`         | Recovery for a cycle           |
| `whoop_list_sleep`           | `read:sleep`            | Paginated sleep activities     |
| `whoop_get_sleep`            | `read:sleep`            | Single sleep by UUID           |
| `whoop_get_cycle_sleep`      | `read:cycles`           | Sleep for a cycle              |
| `whoop_list_workouts`        | `read:workout`          | Paginated workouts             |
| `whoop_get_workout`          | `read:workout`          | Single workout by UUID         |

All list tools accept `limit` (≤25), `start`, `end` (ISO 8601), `next_token`, and `response_format` (`markdown` or `json`).

## Architecture

```
src/
├── index.ts                   # McpServer + stdio transport
├── constants.ts               # URLs, scopes, token path
├── types.ts                   # StoredTokens, PaginatedResponse, ResponseFormat
├── auth/token-manager.ts      # Singleton cache, auto-refresh, rotation
├── services/whoop-client.ts   # whoopGet() + WhoopApiError mapper
├── schemas/common.ts          # Pagination + response_format Zod shapes
└── tools/                     # One file per resource
    ├── profile.ts
    ├── cycles.ts
    ├── recovery.ts
    ├── sleep.ts
    ├── workouts.ts
    └── activity-mapping.ts
scripts/
├── auth.ts                    # One-time OAuth flow (localhost callback)
└── smoke-test.ts              # End-to-end tool check
```

Refresh tokens are long-lived but single-use; the manager rotates them on every refresh and rewrites the store. Access tokens refresh automatically 1 minute before expiry, with in-flight deduplication so concurrent tool calls share one refresh.

## Troubleshooting

- **`Could not read WHOOP tokens…`** — run `pnpm run auth` first.
- **`WHOOP auth failed (401)`** — refresh token is invalid or revoked. Re-run `pnpm run auth`.
- **`WHOOP scope forbidden (403)`** — the scope wasn't granted during consent. Re-authorize and tick all boxes.
- **Cloudflare loop on consent page** — you're inside a headless/automated browser. Paste the URL into regular Chrome.
- **`invalid_request` during auth** — the redirect URI registered on the WHOOP app dashboard doesn't exactly match `http://localhost:4567/callback`.

## License

MIT. See [LICENSE](LICENSE).
