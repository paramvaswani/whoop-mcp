# MCP Registry Publication Kit

**Status:** ready for Param's review. Do NOT execute publish steps autonomously — they are irreversibly public.

## What changed (2026 update)

The old plan "PR whoop-mcp into `modelcontextprotocol/servers`" is **obsolete**. As of mid-2025, that repo stopped accepting third-party servers. The new path is the **MCP Server Registry** (`registry.modelcontextprotocol.io`). Submission is via npm publish + `mcp-publisher` CLI, not PR.

References:

- https://github.com/modelcontextprotocol/servers/blob/main/CONTRIBUTING.md — new "we don't accept new server implementations" policy
- https://github.com/modelcontextprotocol/registry — the registry repo
- Quickstart: https://github.com/modelcontextprotocol/registry/blob/main/docs/modelcontextprotocol-io/quickstart.mdx
- Schema: https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json

## Publication flow (when Param is awake)

### 1. Update `package.json`

Add the `mcpName` field and change the package name to a scoped/reserved name.

```jsonc
{
  "name": "@paramxclaudedev/mcp-server-whoop",
  "mcpName": "io.github.paramxclaudedev/whoop",
  "version": "1.0.0",
  // ...existing fields
}
```

Rationale: npm scoped name matches GitHub handle for provenance; `mcpName` is the reverse-DNS identifier used by the registry.

### 2. Flip the GitHub repo to public

Currently `paramxclaudedev/whoop-mcp` is private. Registry requires a public `repository.url`. Flip it via:

```bash
gh repo edit paramxclaudedev/whoop-mcp --visibility public --accept-visibility-change-consequences
```

Pre-flight: commit history is already clean of real secrets (verified 2026-04-19 before publication kit was drafted).

### 3. Write `server.json`

Save the block below as `server.json` at the repo root.

```json
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json",
  "name": "io.github.paramxclaudedev/whoop",
  "description": "Read WHOOP Developer API v2 — profile, body measurements, cycles, recovery, sleep, workouts — as typed tools.",
  "repository": {
    "url": "https://github.com/paramxclaudedev/whoop-mcp",
    "source": "github"
  },
  "version": "1.0.0",
  "packages": [
    {
      "registryType": "npm",
      "identifier": "@paramxclaudedev/mcp-server-whoop",
      "version": "1.0.0",
      "transport": { "type": "stdio" },
      "environmentVariables": [
        {
          "name": "WHOOP_CLIENT_ID",
          "description": "WHOOP developer app Client ID (register at https://developer.whoop.com).",
          "isRequired": true,
          "isSecret": false
        },
        {
          "name": "WHOOP_CLIENT_SECRET",
          "description": "WHOOP developer app Client Secret.",
          "isRequired": true,
          "isSecret": true
        }
      ]
    }
  ]
}
```

### 4. Install the publisher CLI

```bash
curl -fsSL https://raw.githubusercontent.com/modelcontextprotocol/registry/main/scripts/install.sh | sh
# installs `mcp-publisher` to ~/.local/bin/
```

### 5. Publish to npm + register

```bash
cd ~/Code/claude-projects/05-whoop-mcp

# One-time npm login
npm login  # browser flow

# Build + publish
pnpm run build
npm publish --access public

# Authenticate with registry via GitHub
mcp-publisher login github

# Submit server.json
mcp-publisher publish
```

Expected: entry appears at `https://registry.modelcontextprotocol.io/servers?search=whoop` within a few minutes.

## Post-publish checklist

- [ ] Verify the registry listing loads
- [ ] Add the registry URL to the README
- [ ] Submit to community "awesome" lists (punkpeye/awesome-mcp-servers, wong2/awesome-mcp-servers) via their PR flows
- [ ] Tweet / LinkedIn announcement (optional — content-first GTM)
- [ ] Bump memory file `project_mcp_servers.md` with the registry URL

## Risks + gotchas

- **npm scoped package**: if `@paramxclaudedev` isn't yet reserved on npm, `npm login` then `npm access get-current-user` → create the scope.
- **WHOOP API ToS**: the README already surfaces this. Users need their own WHOOP developer app — Param's Client ID/Secret are not shipped.
- **Cloudflare consent loop**: documented in README troubleshooting. No fix required.
- **Refresh-token rotation**: current implementation rewrites `~/.whoop-mcp/tokens.json` on every refresh. This is correct for single-user local use. For multi-user/server deployments, `tokens.json` needs to move out of the home dir — that's post-1.0 work.

## What was NOT done overnight

- Actual npm publish (public-facing — explicitly scoped out)
- Actual registry submission (public-facing)
- Flipping the GitHub repo to public (public-facing)
- Adding `mcpName` to package.json on disk (would land in the next commit — held back so the repo doesn't suggest a publish happened)

Everything above is Param's call. The kit is ready.
