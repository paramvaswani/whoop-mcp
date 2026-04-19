# Upstream publishing prep (drafted Apr 19 overnight)

This file stages what's needed to publish whoop-mcp-server publicly. No public action has been taken — Param reviews in the morning and decides when/where to ship.

## State of the package

- Version `1.0.0` in `package.json` ✓
- README has Setup / Authorize / Register / Tool list ✓
- CHANGELOG exists ✓
- LICENSE exists ✓
- Zero runtime deps beyond `@modelcontextprotocol/sdk` + `zod` ✓
- OAuth tokens stored at `~/.whoop-mcp/tokens.json`, mode 0600 ✓
- Stdio transport, no listening server post-auth ✓
- 11 tools, all `readOnlyHint: true` ✓

This reads upstream-ready today.

## Open question: where to publish

Three venues. Pick one in AM.

1. **Standalone public repo → npm publish.** Simplest, most control. Name `whoop-mcp-server` likely free. Param maintains as a solo open-source package. Markets well on HN / the MCP community list.

2. **PR to modelcontextprotocol/servers.** The official reference registry. BUT — verify the repo is still accepting new server submissions, as the ecosystem has been shifting toward a distributed npm/package model. Check README for "Community Servers" section and submission guidelines.

3. **Submit to the MCP community list** (mcp.so or similar registry site). Low effort, less discoverability than option 1.

Recommendation: **option 1** — publish as a standalone repo first, then get listed in community registries once it has a star or two. Lowest-friction path to reach "real users can install whoop-mcp-server".

## Pre-publish checklist (do these before pushing public)

- [ ] Bump README "install" block to use `npm install -g whoop-mcp-server` once published (currently still shows `git clone <you>/...`)
- [ ] Verify no client-secret or client-id committed in `src/` or `scripts/`. Grep confirmed on Apr 17; re-verify before publish.
- [ ] Remove `.env` from repo (should already be gitignored — verify with `git ls-files`)
- [ ] Add a short demo GIF or screenshot of Claude calling `whoop_list_recoveries`
- [ ] Decide license — already MIT per LICENSE file; keep that
- [ ] Add `keywords` to package.json: `mcp`, `model-context-protocol`, `whoop`, `biometrics`, `health`, `wearables`
- [ ] Add `repository.url` and `bugs.url` fields
- [ ] Add `publishConfig: { "access": "public" }` so scoped packages don't fail
- [ ] Run `npm pack --dry-run` and review the file list — nothing in `node_modules/`, no `.env`, no `tokens.json`

## PR template (if going via option 2 — modelcontextprotocol/servers)

Title: `Add whoop-mcp-server: biometric data via WHOOP v2 API`

Body:

> This PR adds whoop-mcp-server to the community servers list.
>
> **What it does:** Exposes WHOOP wearable data (recovery, sleep, strain, workouts, cycles, profile, body measurements) as MCP tools for any MCP-compatible client.
>
> **Why it's useful:** WHOOP ships a real developer API but no one has wrapped it for MCP yet. This unlocks Claude-native access to personal biometric history for agent workflows — morning brief pipelines, recovery-aware scheduling, biometric-settled prediction markets.
>
> **Tool count:** 11, all read-only.
>
> **Auth:** OAuth2 with offline refresh; tokens at `~/.whoop-mcp/tokens.json` mode 0600.
>
> **Transport:** stdio.
>
> **Runtime deps:** `@modelcontextprotocol/sdk`, `zod`.
>
> **Status:** v1.0.0, tested end-to-end against the real WHOOP API.
>
> Repo: https://github.com/<param-handle>/whoop-mcp-server
>
> Happy to iterate on naming, scope (I kept it read-only intentionally), or anything else.

## Demo story (for README / post)

Good hook: _"Claude, how did I sleep this week and which days was my recovery above 70?"_ → two tool calls, one English answer, zero API wrestling.

Better hook: _"Settle this bet against @friend on whether I crack 80% recovery Tuesday."_ → this is where Keep comes in. The MCP is the read path; the oracle settler is the write path.

## What I did NOT touch

- Did not push anything to GitHub
- Did not publish to npm
- Did not change `package.json` version or metadata
- Did not create a PR anywhere
- Did not modify OAuth scopes

All above is text-only staging. Run the checklist when you're ready.
