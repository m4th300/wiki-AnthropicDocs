---
type: framework
created: 2026-06-08
updated: 2026-06-08
status: current
sources_count: 2
tags: [domain/claude-code, theme/security]
---

# Permission Architecture

## Description

Claude Code's permission architecture is a layered allow/deny/ask system that controls which tool calls Claude can make without explicit user approval. It operates at three levels: a base permission mode (sets default behavior), declarative allow/deny rules (per-tool or per-pattern), and hooks (programmatic per-call decisions). The system is designed to balance safety with automation — enabling unattended runs in CI while remaining conservative in interactive sessions.

## Mechanism

**Evaluation order (strict, first match wins):**

1. **Hooks** — `PreToolUse` and `PermissionRequest` handlers can `deny`, `allow`, `ask`, or `defer`. Decision priority within hooks: `deny > defer > ask > allow`.
2. **Deny rules** — `disallowed_tools` / `permissions.deny` entries. Bare tool name removes it from context entirely; scoped pattern (`Bash(rm *)`) blocks matching calls while keeping the tool visible.
3. **Permission mode** — sets the floor for unmatched tools:

| Mode | Auto-approves | Suitable for |
|---|---|---|
| `default` | Read-only tools | Sensitive/interactive work |
| `acceptEdits` | File edits + common FS commands (mkdir, rm, mv, cp, sed, touch, rmdir) | Coding iteration |
| `plan` | Read-only only | Exploration before modification |
| `auto` | Everything, with background safety classifier | Long tasks, reduced interruption |
| `dontAsk` | Only pre-approved tools + read-only Bash | Locked CI |
| `bypassPermissions` | Everything (includes protected paths) | Sandboxed containers/VMs only |

4. **Allow rules** — `allowed_tools` / `permissions.allow`. Auto-approves matching tools/patterns.
5. **`canUseTool` callback** (SDK) or **user prompt** (interactive) — final fallback for unmatched cases.

**Rule syntax:**

```json
{
  "permissions": {
    "allow": ["Bash(npm run test *)", "Read(~/.zshrc)", "mcp__puppeteer"],
    "deny":  ["Bash(curl *)", "Read(./.env)", "Bash(git push *)"]
  }
}
```

Pattern matching:
- `Bash` (bare) → all usages
- `Bash(npm run build)` → exact match
- `Bash(npm run test *)` → prefix wildcard
- `Bash(* --version)` → suffix wildcard
- `Read(./**/dist/**)` → recursive path glob
- `mcp__puppeteer` → all tools on puppeteer MCP server
- `mcp__puppeteer__puppeteer_navigate` → specific MCP tool
- `Agent(Explore)` → specific subagent

**Settings hierarchy (highest to lowest priority):**

```
Managed policy > CLI args > .claude/settings.local.json > .claude/settings.json > ~/.claude/settings.json
```

A deny rule at any level cannot be overridden by a lower level. Arrays merge across levels; scalars: higher scope wins.

**Protected paths** (never auto-approved except in `bypassPermissions`):
`.git`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.claude` (except commands/agents/skills/worktrees), `.gitconfig`, `.bashrc`, `.zshrc`, `.npmrc`, `.mcp.json`, etc.

**Auto-approved read-only Bash commands** (no prompt in any mode):
`ls`, `cat`, `echo`, `pwd`, `head`, `tail`, `grep`, `find`, `wc`, `which`, `diff`, `stat`, `du`, `cd`, read-only git forms. Wrappers stripped: `timeout 30 npm test` → matched against `npm test`.

**`auto` mode classifier** (v2.1.83+, requires Sonnet 4.6 or Opus 4.6+)
- Blocks by default: `curl | bash`, production deployments/migrations, bulk cloud storage deletions, IAM permission grants, direct push to main
- Falls back (disables auto mode) after 3 consecutive or 20 total blocks
- Reads CLAUDE.md from the project — project-level behavioral rules affect the classifier

**`bypassPermissions`** (v2.1.126+)
- Requires non-root on Linux/macOS
- Exception: `rm -rf /` and `rm -rf ~` always prompt
- All tool calls run without prompts (hooks can still block)
- Recommended only inside containers or VMs

**Enterprise controls** (managed settings):

| Key | Effect |
|---|---|
| `allowManagedPermissionRulesOnly` | Only managed rules apply; user/project rules ignored |
| `allowManagedMcpServersOnly` | Only managed MCP servers |
| `allowManagedHooksOnly` | Only managed hooks |
| `strictPluginOnlyCustomization` | Block all user/project customization |
| `disableBypassPermissionsMode` | Prevent `bypassPermissions` |

## Limits

- **Hooks can be bypassed**: `bypassPermissions` mode runs everything without prompts, but hooks still execute. Managed `allowManagedHooksOnly` prevents this at org level.
- **No secrets protection by default**: deny rules must be configured explicitly for `.env`, credentials files, etc.
- **Pattern matching is not sandboxing**: permission rules control Claude's tool calls, not OS-level isolation. For real isolation, combine with the Bash sandbox or run in a container.
- **`auto` mode can fall back**: after hitting the block threshold, it disables itself — unattended runs may need a fallback strategy.
- **Bash is not granular at the OS level**: Claude Code's `Bash(rm *)` rule matches command strings, not OS operations. A command like `python -c "import os; os.remove('file')"` won't match a `Bash(rm *)` deny rule.

## Connections

- [[features/extension-model]] — hooks (PreToolUse) and managed settings both integrate into the permission layer
- [[concepts/agentic-loop]] — permissions gate each action step in the loop
- [[features/hooks]] — the programmatic extension of the permission layer
- [[features/sandboxing]] — OS-level isolation that complements permission rules
- [[features/agent-sdk]] — same permission model with Python/TypeScript API (`canUseTool` callback, modes)

## Sources

- [[sources/2026-06-08_permissions-security-extensions]] — modes, rules, auto mode, bypassPermissions, enterprise controls
- [[sources/2026-06-08_agent-sdk-extensions]] — SDK permission evaluation order, `canUseTool`, dynamic mode switching
