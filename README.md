# whoop-mcp-server

Local MCP server wrapping the [WHOOP Developer API v2](https://developer.whoop.com/). Exposes cycles, recovery, sleep, workouts, profile, and body measurement as typed tools for Claude Code.

## Setup

### 1. Create a WHOOP app

Go to [developer.whoop.com](https://developer.whoop.com/), sign in, create a new app.

- **Redirect URI**: `http://localhost:4567/callback`
- Copy the Client ID and Client Secret.

### 2. Install

```bash
cd "/Users/p/Library/Mobile Documents/com~apple~CloudDocs/claude-projects/05-whoop-mcp"
pnpm install
cp .env.example .env
# edit .env with your Client ID + Secret
pnpm run build
```

### 3. Authorize (one-time)

```bash
pnpm run auth
```

Opens your browser, you log in to WHOOP and approve scopes. Tokens saved to `~/.whoop-mcp/tokens.json`. The server auto-refreshes access tokens on the fly after this.

### 4. Register with Claude Code

Add to `~/.claude/mcp_settings.json` (or run `claude mcp add`):

```json
{
  "mcpServers": {
    "whoop": {
      "command": "node",
      "args": [
        "/Users/p/Library/Mobile Documents/com~apple~CloudDocs/claude-projects/05-whoop-mcp/dist/index.js"
      ],
      "env": {
        "WHOOP_CLIENT_ID": "your_client_id",
        "WHOOP_CLIENT_SECRET": "your_client_secret"
      }
    }
  }
}
```

Restart Claude Code. Test with: _"Use whoop_get_profile"_ or _"Get my last 7 recoveries."_

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

All list tools accept `limit` (≤25), `start`, `end` (ISO 8601), `next_token`, and `response_format`.

## Architecture

```
src/
├── index.ts                   # McpServer + stdio transport
├── constants.ts               # URLs, scopes, token path
├── types.ts                   # StoredTokens, PaginatedResponse, ResponseFormat
├── auth/token-manager.ts      # Load refresh token, cache access token, auto-refresh
├── services/whoop-client.ts   # whoopGet() + error mapper
├── schemas/common.ts          # Pagination + response_format Zod shapes
└── tools/
    ├── profile.ts
    ├── cycles.ts
    ├── recovery.ts
    ├── sleep.ts
    └── workouts.ts
scripts/auth.ts                # One-time OAuth flow (localhost callback server)
```

Tokens live at `~/.whoop-mcp/tokens.json` (mode 0600). Refresh token is long-lived; access tokens refresh automatically 1 minute before expiry.

## Lifebet context

This server exists to validate the Lifebet oracle concept — Claude can query WHOOP data directly during dev so you can prototype market-settlement logic without writing a one-off integration every time.
