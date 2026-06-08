---
type: source
created: 2026-06-08
updated: 2026-06-08
status: published
author: Anthropic
date_source: 2026-06-08
raw_file: raw/agent-sdk-hooks.md, raw/agent-sdk-mcp.md, raw/agent-sdk-permissions.md, raw/agent-sdk-plugins.md, raw/agent-sdk-skills.md, raw/agent-sdk-slash-commands.md, raw/agent-sdk-tool-search.md, raw/agent-sdk-user-input.md, raw/agent-sdk-custom-tools.md
tags: [domain/agent-sdk, theme/extensions]
sources_count: 9
---

# Agent SDK — Extensions & Customization

## Résumé

The Agent SDK exposes all of Claude Code's extension mechanisms as first-class options: hooks intercept lifecycle events and can mutate or block tool calls; custom tools are defined with `@tool`/`tool()` and exposed via an in-process MCP server; external MCP servers connect via stdio or HTTP/SSE; plugins bundle skills, agents, hooks, and MCP config together; tool search lazily loads large catalogs to preserve context. The permission system has a strict evaluation order: hooks → deny rules → permission mode → allow rules → `canUseTool` callback.

## Points clés

**Custom tools**
- Python: `@tool("name", "desc", {"param": type})` decorator on async fn
- TypeScript: `tool(name, description, zodSchema, handler)` function
- Register: `create_sdk_mcp_server(name, version, tools)` → pass to `mcp_servers` option
- Tool naming in model: `mcp__<server>__<tool>` (e.g. `mcp__weather__get_temperature`)
- Return types: `{"content": [{"type": "text", "text": "..."}]}`, `is_error: True` for non-fatal errors
- Annotations: `readOnlyHint` (enables parallel calls), `destructiveHint`, `idempotentHint`, `openWorldHint`

**Hooks in SDK**
- Register via `hooks={...}` in `ClaudeAgentOptions` or `hooks` key in TypeScript Options
- Matcher syntax: exact string, pipe-delimited list (`Write|Edit`), or regex (when non-alphanumeric chars present)
- Decision output: `{"hookSpecificOutput": {"permissionDecision": "deny|allow|ask|defer", "updatedInput": {...}}}`
- Priority: `deny > defer > ask > allow`
- Async non-blocking: return `{"async_": True, "asyncTimeout": 30000}`
- Available events (TypeScript superset): `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `UserPromptSubmit`, `Stop`, `SubagentStart/Stop`, `PreCompact`, `PermissionRequest`, `Notification`, `SessionStart/End`, `TaskCompleted`, `ConfigChange`, `WorktreeCreate/Remove`

**MCP in SDK**
- Stdio: `{"command": "npx", "args": [...], "env": {...}}`
- HTTP/SSE: `{"type": "sse", "url": "...", "headers": {...}}`
- In-process: pass `create_sdk_mcp_server()` result directly (no network)
- `.mcp.json` at project root loaded if `"project"` in `settingSources`
- MCP tools must be in `allowed_tools` to auto-approve (not covered by `acceptEdits`)
- OAuth2: must complete flow in application and pass token via headers

**Permissions in SDK**
- Evaluation order: hooks → `disallowed_tools` deny list → `bypassPermissions` → `allowed_tools` allow list → `canUseTool` callback
- `allowed_tools`: auto-approved; unlisted fall through to mode
- `disallowed_tools`: bare name = removed from context; scoped `Bash(rm *)` = tool visible but pattern refused
- Modes: `default`, `dontAsk` (deny if not in allowedTools), `acceptEdits`, `bypassPermissions`, `plan`, `auto` (TS only)
- `bypassPermissions`, `acceptEdits`, `auto` are inherited by subagents, cannot be overridden
- Dynamic mode switch: `await client.set_permission_mode("acceptEdits")` / `q.setPermissionMode("acceptEdits")`

**Tool search** (`ENABLE_TOOL_SEARCH` env var)
- `true`/`false`/`auto`/`auto:N` (% of context window threshold)
- Supported: Sonnet 4+, Opus 4+ only (not Haiku)
- Disabled by default on Vertex AI and third-party proxies
- Max 10,000 tools; 3-5 returned per search query

**User input / canUseTool callback**
- Triggers: tool not auto-approved, or `AskUserQuestion` tool call
- Python: `can_use_tool` callback; TypeScript: `canUseTool`
- Allow: `PermissionResultAllow(updated_input=...)` / `{behavior: "allow", updatedInput}`
- Deny: `PermissionResultDeny(message=...)` / `{behavior: "deny", message}`
- `AskUserQuestion`: 1-4 questions per call, 2-4 options each; not in subagents

**Skills & plugins in SDK**
- Skills: filesystem-only (`SKILL.md` files); `skills` option: `"all"`, `["name1"]`, or `[]`
- `allowed-tools` frontmatter in SKILL.md is IGNORED in SDK (control via `allowedTools` option instead)
- Plugins: `{type: "local", path: "./my-plugin"}` — directory with skills/, agents/, hooks/, .mcp.json
- Plugin skill invocation: `/plugin-name:skill-name`

**Slash commands in SDK**
- Sent as ordinary prompt strings
- `/compact`: emits `compact_boundary` system message with token metadata
- `/clear`: resets context (requires v2.1.117+); single `query()` calls already start fresh

## Liens wiki

- [[features/agent-sdk]] — this source extends the SDK feature page
- [[features/hooks]] — full hooks system detail
- [[features/mcp]] — MCP server integration
- [[features/extension-model]] — comparative overview of all extension mechanisms

## Tensions

Aucune tension identifiée. Note: some TypeScript-only permission modes (`multiStep`, `free`, `auto`) have no Python equivalent — this is an intentional API surface difference, not a discrepancy.

## Questions soulevées

- What does `auto` permission mode's "classifier" do exactly — is it the same as Claude Code's `auto mode` classifier?
- When does tool search get re-triggered after compaction? Is it immediate or on the next tool call?
