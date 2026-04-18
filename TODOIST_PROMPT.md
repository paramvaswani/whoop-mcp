# Paste this into a new Claude Code chat to build a Todoist MCP server

---

Build a Todoist MCP server that mirrors the structure of my existing `whoop-mcp-server` at `/Users/p/Library/Mobile Documents/com~apple~CloudDocs/claude-projects/05-whoop-mcp/`. Use the `anthropic-skills:mcp-builder` skill. Read that whoop-mcp-server as a reference for naming, Zod pagination shape, shared response formatters, error mapping, and README style.

## Target location

`/Users/p/Library/Mobile Documents/com~apple~CloudDocs/claude-projects/06-todoist-mcp/`

## Stack

- TypeScript, ES2022, Node 18+
- `@modelcontextprotocol/sdk` v1+, `zod`
- `McpServer` + `StdioServerTransport` + `registerTool` (modern API — no deprecated `server.tool()`)
- Server name: `todoist-mcp-server`

## Auth

Todoist uses a simple bearer API token — no OAuth. User gets it from Todoist → Settings → Integrations → Developer → "API token". Read it from `TODOIST_API_TOKEN` env var. Fail fast on startup if missing.

## API

Todoist REST API v2. Base URL: `https://api.todoist.com/rest/v2`. All requests need `Authorization: Bearer <token>`. Sync API is richer but REST v2 covers 95% of needs cleanly — use REST v2.

Reference: https://developer.todoist.com/rest/v2/

## Tools to implement (prefix each with `todoist_`)

### Tasks (most used — make these solid)

- `todoist_list_tasks` — GET `/tasks`. Filter params: `project_id`, `section_id`, `label`, `filter` (Todoist filter query like `today | overdue`), `lang`, `ids`. No native pagination in REST v2, so return everything but cap to a `limit` (client-side, default 50, max 200).
- `todoist_get_task` — GET `/tasks/{id}`.
- `todoist_create_task` — POST `/tasks`. Body: `content` (required), `description`, `project_id`, `section_id`, `parent_id`, `labels[]`, `priority` (1-4, where 4 is highest), `due_string`, `due_date`, `due_datetime`, `due_lang`, `assignee_id`, `duration`, `duration_unit` (minute|day).
- `todoist_update_task` — POST `/tasks/{id}`. Same fields as create, all optional.
- `todoist_close_task` — POST `/tasks/{id}/close`. Marks complete.
- `todoist_reopen_task` — POST `/tasks/{id}/reopen`.
- `todoist_delete_task` — DELETE `/tasks/{id}`. `destructiveHint: true`.

### Projects

- `todoist_list_projects` — GET `/projects`.
- `todoist_get_project` — GET `/projects/{id}`.
- `todoist_create_project` — POST `/projects` with `name`, `parent_id`, `color`, `is_favorite`, `view_style`.

### Sections

- `todoist_list_sections` — GET `/sections?project_id=...`.

### Labels

- `todoist_list_labels` — GET `/labels`.

### Comments

- `todoist_list_comments` — GET `/comments?task_id=...` or `?project_id=...`.
- `todoist_create_comment` — POST `/comments`.

Skip shared labels, collaborators, and sync API endpoints for v1.

## Conventions (match whoop-mcp-server)

- Snake*case tool names with `todoist*` prefix.
- Every tool: `title`, `description` (with args + return schema), `inputSchema` (Zod), `annotations` (readOnlyHint / destructiveHint / idempotentHint / openWorldHint).
- Use `.strict()` on Zod objects where applicable.
- Shared `responseFormatSchema` with `markdown` (default) and `json`.
- Error mapper covering 401 (bad token), 403, 404, 429 with actionable messages.
- Return both `content: [{type: "text", text}]` and `structuredContent` for rich output.
- `CHARACTER_LIMIT = 25000` — truncate list responses that exceed this with a `truncation_message`.

## File layout (mirror whoop-mcp-server)

```
06-todoist-mcp/
├── package.json            # "todoist-mcp-server", bin entry, tsx dev script, tsc build
├── tsconfig.json           # ES2022, Node16, strict
├── .env.example            # TODOIST_API_TOKEN=
├── .gitignore
├── README.md               # Setup + Claude Code registration snippet + tool table
└── src/
    ├── index.ts            # McpServer + stdio + register*Tools()
    ├── constants.ts        # BASE_URL, CHARACTER_LIMIT
    ├── types.ts            # ResponseFormat enum, Task/Project/etc interfaces
    ├── services/todoist-client.ts   # todoistFetch<T>(method, path, body?, query?) + error mapper
    ├── schemas/common.ts   # responseFormatSchema, limit shape
    └── tools/
        ├── tasks.ts
        ├── projects.ts
        ├── sections.ts
        ├── labels.ts
        └── comments.ts
```

No separate auth script needed — just the env var.

## Done when

- `pnpm install && pnpm run build` completes clean.
- `node dist/index.js` fails loudly if `TODOIST_API_TOKEN` is unset.
- README has the exact `~/.claude/mcp_settings.json` snippet the user can paste.
- Tools appear in Claude Code on restart and can list/create/close tasks end-to-end.

Log the session to the Notion "Claude Sessions" database (data_source_id `2a972407-306a-450d-9351-330f62e90d95`) with tags `[productivity, setup]` when done.
