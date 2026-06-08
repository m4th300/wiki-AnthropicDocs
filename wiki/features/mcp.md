---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 2
tags: [domain/claude-code, theme/extensions, theme/integrations]
---

# MCP (Model Context Protocol)

## Description

MCP is an open standard for connecting AI tools to external data sources and services. In Claude Code, MCP servers give Claude new tools — for Slack, Jira, databases, browsers, filesystems, and anything else that implements the protocol. Tool schemas are deferred by default: only tool names load at startup; full schemas are fetched on demand, keeping the context window lean. MCP servers run as local subprocesses (stdio transport) or remote HTTP/SSE services.

## Status & Requirements

- **Status**: stable
- **Platforms**: All for stdio and HTTP; Managed MCP requires Teams/Enterprise
- **Tool search**: enabled by default except on Vertex AI and third-party proxies

## Configuration

**Add an MCP server:**
```bash
# Stdio (local process)
claude mcp add sqlite uvx mcp-server-sqlite@latest database.db
claude mcp add playwright -- npx -y @playwright/mcp@latest

# HTTP/SSE (remote server)
claude mcp add --transport http claude-code-docs https://code.claude.com/docs/mcp

# With auth header
claude mcp add --transport http github <url> --header "Authorization: Bearer <token>"

# Project-scoped (shareable via .mcp.json)
claude mcp add --scope project --transport http <name> <url>
```

**`.mcp.json`** at project root (shared with team via source control):
```json
{
  "mcpServers": {
    "sqlite": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-server-sqlite@latest", "database.db"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://api.example.com/mcp/sse"
    }
  }
}
```

**Configuration scopes:**

| Scope | File | Available to |
|---|---|---|
| `local` (default) | `~/.claude.json` + project entry | You, this project only |
| `project` | `.mcp.json` at project root | All collaborators |
| `user` | `~/.claude.json` `mcpServers` key | You, all projects |

**Tool naming:** `mcp__<server-name>__<tool-name>` (e.g. `mcp__sqlite__query`)

**Tool search** (context optimization):
- `ENABLE_TOOL_SEARCH=true`: always enabled
- `ENABLE_TOOL_SEARCH=auto`: enabled if tools exceed 10% of context window
- `ENABLE_TOOL_SEARCH=auto:N`: N% threshold
- `ENABLE_TOOL_SEARCH=false`: disable; all schemas load at startup
- Disable selectively: `CLAUDE_CODE_DISABLE_MCP_TOOL_SEARCH=1`

**Permission requirement:** MCP tools must be explicitly listed in `allowed_tools` / `permissions.allow` to auto-approve. `acceptEdits` mode does NOT auto-approve MCP tools.

**OAuth-authenticated servers:**
```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
# Then: /mcp → select server → Authenticate
```

**Troubleshooting:**
- Connection timeout: `MCP_TIMEOUT=60000 claude`
- Reset project choices: `claude mcp reset-project-choices`
- Connection states: `✓ Connected` / `! Needs authentication` / `✗ Failed` / `⏸ Pending approval`

**Managed MCP** (Teams/Enterprise, `managed-mcp.json`):
- `managed-mcp.json` path: macOS `/Library/Application Support/ClaudeCode/managed-mcp.json`, Linux `/etc/claude-code/managed-mcp.json`
- When present: Claude Code loads ONLY these servers; users cannot add others
- `allowManagedMcpServersOnly: true` in managed settings: makes allowlist authoritative
- Denylist: `deniedMcpServers` with `serverUrl`, `serverCommand`, or `serverName` matching keys
- Error messages: users see explicit "blocked by enterprise policy" messages

## Use Cases

- **Code intelligence**: `typescript-lsp@claude-plugins-official` for symbol navigation (replaces file reads)
- **Codebase context**: `github`, `gitlab` MCP servers for issue/PR context without manual copy-paste
- **Browser automation**: `playwright` or `puppeteer` for web UI testing
- **Database access**: `sqlite`, `postgres` MCP servers for direct data querying
- **Team knowledge**: Slack, Notion, Jira MCP servers for context from team tools
- **External APIs**: any service implementing the MCP protocol

## Tradeoffs

- **Stdio servers**: must be installed on the user's machine; not available in cloud sessions (Routines, Web)
- **HTTP servers**: available in cloud sessions; require auth handling; may have cold start latency
- **Tool search**: reduces context usage but adds a round-trip per tool batch; disable with large catalogs if tool selection is accurate
- **`acceptEdits` mode gap**: unlike file edit tools, MCP tools require explicit `allow` rules — easy to forget when setting up new servers
- **Security**: MCP servers execute code; Anthropic reviews official connectors but cannot audit third-party servers. Treat MCP servers like npm packages.

## Connections

- [[features/extension-model]] — MCP in the context of all extension mechanisms
- [[features/hooks]] — `mcp_tool` handler type lets hooks call MCP tools
- [[frameworks/permission-architecture]] — MCP tool approval and managed MCP controls
- [[features/agent-sdk]] — MCP in SDK context, tool naming, in-process MCP server creation

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_extensions-ecosystem]] — add commands, transport types, tool search, managed MCP
- [[sources/2026-06-08_agent-sdk-extensions]] — MCP in SDK context, OAuth2 notes, in-process MCP server
