---
type: source
created: 2026-06-08
updated: 2026-06-08
status: published
author: Anthropic
date_source: 2026-06-08
raw_file: raw/hooks.md, raw/hooks-reference.md, raw/mcp.md, raw/managed-mcp.md, raw/mcp-quickstart.md, raw/plugins.md, raw/plugins-reference.md, raw/plugin-marketplaces.md, raw/skills.md, raw/prompt-library.md
tags: [domain/claude-code, theme/extensions]
sources_count: 10
---

# Extensions Ecosystem — Hooks, MCP, Plugins, Skills

## Résumé

Claude Code's extension ecosystem has four interlocking mechanisms: hooks (lifecycle event handlers that can block or modify tool calls), MCP (external tool and data server protocol), plugins (distributable bundles packaging skills, agents, hooks, and MCP), and skills (on-demand markdown instruction files). Together they cover the full spectrum from lightweight instructions (skills) to enterprise-managed infrastructure (managed MCP). Each mechanism operates at a distinct point in the agentic loop with distinct context costs and governance controls.

## Points clés

**Hooks**
- 30+ lifecycle events: `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`, `Stop`, `SubagentStart/Stop`, `PreCompact`, `FileChanged`, `CwdChanged`, `Notification`, `Elicitation`, `TeammateIdle`, `TaskCompleted`, `WorktreeCreate/Remove`, etc.
- 5 handler types: `command` (shell), `http` (POST to endpoint), `mcp_tool` (call MCP tool), `prompt` (single-turn LLM), `agent` (spawn subagent, experimental)
- Configuration location: `hooks` key in `settings.json` at any scope level, or plugin `hooks/hooks.json`, or skill/agent frontmatter
- Exit code 2 = blocking error (blocks tool call, rejects prompt, prevents stop, etc.)
- `hookSpecificOutput.permissionDecision`: `deny|allow|ask|defer` from `PreToolUse` handlers
- `updatedToolInput`: hooks can mutate the tool's input before execution
- Async non-blocking: `{"async": true, "asyncTimeout": 30000}`
- Path placeholders: `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}`

**MCP (Model Context Protocol)**
- Open standard for connecting Claude to external data sources and services
- Transport types: `stdio` (local subprocess, most common), `http`/`SSE` (remote)
- Tool naming: `mcp__<server>__<tool>` (e.g. `mcp__sqlite__query`)
- MCP tools must be in `allowed_tools` to auto-approve (not covered by `acceptEdits` mode)
- Tool search: only tool names load at startup; full schemas fetched on demand (deferred)
- Add: `claude mcp add <name> <command>` or `claude mcp add --transport http <name> <url>`
- Shareable: `.mcp.json` at project root; `claude mcp add --scope project`
- OAuth-authenticated servers: `claude mcp add --transport http <name> <url>` → then `/mcp` → Authenticate
- Connection states: Connected ✓ / Needs auth ! / Failed ✗ / Pending approval ⏸

**Managed MCP (Enterprise)**
- `managed-mcp.json` at `/Library/Application Support/ClaudeCode/managed-mcp.json` (macOS) or `/etc/claude-code/managed-mcp.json` (Linux)
- When present: Claude Code loads ONLY these servers; users cannot add others
- Control models: fixed deployment / approved catalog / denylist-only / plugins-only / no restrictions
- Allowlist/denylist matching: `serverUrl` (wildcard), `serverCommand` (exact), `serverName` (label — not safe alone)
- `allowManagedMcpServersOnly: true` makes allowlist authoritative

**Plugins**
- Directory structure: `skills/`, `agents/`, `hooks/hooks.json`, `.mcp.json`, `.lsp.json`, `monitors/`, `output-styles/`, `themes/`, `bin/`, `settings.json`
- Manifest: `plugin.json` with `name` (required), `displayName`, `version`, `description`, `defaultEnabled`
- CLI: `claude plugin install/uninstall/enable/disable/update/list/details/validate/prune`
- Test locally: `claude --plugin-dir ./my-plugin` or `claude --plugin-url <url>`
- Monitors (`monitors/monitors.json`): background shell commands whose stdout lines are delivered to Claude as notifications
- `userConfig`: declared values that Claude Code prompts for at plugin activation; available as `${user_config.KEY}`
- Official marketplaces: `claude-plugins-official` (Anthropic) and `claude-community`

**Skills**
- Location: `.claude/skills/<name>/SKILL.md` (project) or `~/.claude/skills/<name>/SKILL.md` (user)
- Invocation: `/skill-name` (manual) or auto-detected by Claude based on description/trigger
- SKILL.md frontmatter: `description`, `tools`, `disable-model-invocation`, `trigger`, `subagent`, `model`
- Bundled skills: `/batch`, `/code-review` (alias `/review`), `/debug`, `/loop`, `/plan`
- In SDK: `allowed-tools` frontmatter is IGNORED; control via `allowedTools` option instead
- Legacy: `.claude/commands/` still works; new code should use `skills/`

## Liens wiki

- [[features/extension-model]] — comparative overview of all mechanisms
- [[features/hooks]] — hooks in depth
- [[features/mcp]] — MCP in depth
- [[frameworks/permission-architecture]] — hooks and managed settings integrate with permissions

## Tensions

Aucune tension identifiée.

## Questions soulevées

- Can `prompt` type hooks cause significant latency per tool call? What's the practical use case for per-turn LLM evaluation vs. a more efficient pattern matcher?
- How do plugin monitors interact with context window size? Is their output always injected into the next turn?
