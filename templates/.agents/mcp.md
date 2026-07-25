# MCP Servers

MCP config location differs by agent runtime — both must be kept in sync when a server is
added or removed:

- Claude Code: `.mcp.json` at the project root.
- Codex: `.codex/config.toml` (project-scoped; only loads for trusted projects) or
  `~/.codex/config.toml` (user-global, applies to every project). This is separate from
  where Codex looks for *skills* (`.agents/skills`, not `.codex/`) — don't conflate the
  two.

## Installed servers

- **angular-mcp** — installed first, before any other scaffolding step. Use it for Angular
  CLI operations (generate, migrations, best-practice lookups) instead of guessing. Call
  its `list_projects` tool first when workspace context matters (multi-project workspace,
  unclear which project a command targets) — don't assume there's only one project.
<!-- SCAFFOLD: repeat this bullet for every additional MCP server installed for the chosen UI library -->
- **{{UI_LIB_MCP_NAME}}** — {{UI_LIB_MCP_PURPOSE}}. Only present if the project's UI
  library ships one; remove this bullet if it doesn't apply.

If an MCP server is unavailable or not connected when a task needs it, stop and tell the
user. Do not silently fall back to guessing at APIs or CLI flags.

## Known Codex Gotcha

On Codex, some Angular MCP tool calls (`get_best_practices`, `search_documentation`) can
return `Unexpected response type` even while the server is healthy. Do not treat this as
the server being down and do not change project source to work around it. Instead, fall
back to the readable MCP resource (`instructions://best-practices`) and to the tools that
are known to work (`list_projects`, `run_target`). This is a client-side quirk, not a
project bug.

## Review Checklist

- [ ] Every installed server listed here matches what's actually in `.mcp.json` /
      `.codex/config.toml`.
- [ ] Both config locations exist for any server the current runtime added, or the other
      runtime's config is explicitly noted as "add if you're on Codex/Claude".
