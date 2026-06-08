---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 2
tags: [domain/claude-code, theme/extensions]
---

# Hooks

## Description

Hooks are user-defined handlers that execute automatically at specific points in Claude Code's lifecycle. They are the primary mechanism for automation, validation, and programmatic permission control — running as shell commands, HTTP POST requests, MCP tool calls, LLM prompts, or subagents in response to lifecycle events. Hooks run synchronously by default (blocking the next action until complete) and can modify tool inputs, deny tool calls, inject context, or fire-and-forget side effects.

## Status & Requirements

- **Status**: stable (core hooks), experimental (`agent` handler type)
- **Platforms**: All

## Configuration

**Structure** — 3-level nesting: Event → Matcher group → Handler list:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/validate-bash.sh",
            "timeout": 30
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [{ "type": "command", "command": "notify-send 'Claude finished'" }]
      }
    ]
  }
}
```

**Configuration scopes** (all are additive — hooks from all scopes run):

| Location | Scope |
|---|---|
| `~/.claude/settings.json` | All projects |
| `.claude/settings.json` | This project (shareable) |
| `.claude/settings.local.json` | This project (gitignored) |
| Plugin `hooks/hooks.json` | When plugin active |
| Skill/agent YAML frontmatter | Component lifetime |
| Managed policy | Organization |

**Matcher syntax:**
- `"*"` or omitted: match all
- `"Bash"` or `"Write|Edit"`: exact match or pipe-delimited list
- `"^mcp__"` or `"mcp__memory__.*"`: JavaScript regex (when non-alphanumeric chars present)

**Handler types:**

| Type | Config | Use for |
|---|---|---|
| `command` | `{"type":"command","command":"script.sh","timeout":30,"async":false}` | Shell scripts, validators |
| `http` | `{"type":"http","url":"http://localhost:8080/hook","headers":{...}}` | Webhook endpoints |
| `mcp_tool` | `{"type":"mcp_tool","server":"my_server","tool":"audit","input":{...}}` | Calling connected MCP tools |
| `prompt` | `{"type":"prompt","prompt":"Is this safe? $ARGUMENTS","model":"claude-opus-4-7"}` | LLM evaluation (expensive) |
| `agent` | `{"type":"agent","prompt":"Review: $ARGUMENTS","timeout":60}` | Subagent with tool access (experimental) |

**Handler I/O contract:**

Input (stdin JSON, common fields): `session_id`, `transcript_path`, `cwd`, `hook_event_name`, `permission_mode`, `effort`, `agent_id`, `agent_type`

Output (stdout JSON):
```json
{
  "continue": true,
  "suppressOutput": false,
  "systemMessage": "Warning: sensitive file",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Blocked by policy",
    "updatedToolInput": {"command": "safe-alternative"}
  }
}
```

**Exit codes:**
- `0`: success; Claude parses stdout
- `2`: blocking error — prevents the triggered action (blocks tool call, rejects prompt, prevents stop, etc.)
- Other: non-blocking error; execution continues

**Async non-blocking pattern:**
```json
{ "type": "command", "command": "send-to-logging.sh", "async": true, "asyncTimeout": 30000 }
```
`asyncRewake: true` wakes Claude if the async handler exits with code 2.

**Special hook outputs:**

`SessionStart` output: `additionalContext`, `sessionTitle`, `watchPaths`, `reloadSkills`, `initialUserMessage`

`PreToolUse` permission control: `permissionDecision` (`deny|allow|ask|defer`), `updatedToolInput` (mutate input before execution)

`Stop` output: exit code 2 = prevents stop, continues conversation

**Path placeholders:** `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}`

**Disable all hooks:** `{"disableAllHooks": true}` in settings. View: `/hooks` (read-only browser).

## Use Cases

- **Security validation**: `PreToolUse` on `Bash` → pattern match for dangerous commands → exit 2 to block
- **Audit logging**: `PostToolUse` async → POST to audit service without blocking
- **External notifications**: `Stop` → send Slack/Discord message when Claude finishes
- **Environment setup**: `SessionStart` → inject branch name, ticket context into `additionalContext`
- **Permission relay**: `PermissionRequest` → relay to mobile app for remote approval
- **Anti-drift**: `PreCompact` → block compaction if certain context must be preserved
- **Teammate coordination**: `TeammateIdle` → assign next task before teammate goes idle

## Tradeoffs

- **Latency**: synchronous shell hooks add latency to every matched tool call. Use `"async": true` for logging/notifications; reserve blocking hooks for genuine validation
- **Complexity**: hooks from all scopes run additively — hard to debug when multiple hooks conflict
- **`prompt` handler cost**: each `prompt`-type hook makes a separate Claude API call — can significantly increase costs if triggered frequently
- **`agent` handler**: experimental; spawns a full subagent context, higher latency and cost than `command` type
- **Exit code 2 semantics vary by event**: `Stop` exit 2 prevents stop; `PreToolUse` exit 2 blocks tool call; `PostToolUse` exit 2 is non-blocking — read the reference carefully

## Connections

- [[frameworks/permission-architecture]] — hooks integrate into the permission evaluation order at the top
- [[features/extension-model]] — hooks vs. skills vs. subagents comparison
- [[features/agent-sdk]] — same hooks available in SDK via `hooks` option key
- [[features/mcp]] — `mcp_tool` handler type calls MCP tools from hooks

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_extensions-ecosystem]] — full event list, handler types, I/O contract, exit codes
- [[sources/2026-06-08_agent-sdk-extensions]] — hooks in SDK context, lifecycle events, matcher syntax
