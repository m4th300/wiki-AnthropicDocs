---
type: framework
created: 2026-06-08
updated: 2026-06-08
status: current
sources_count: 2
tags: [domain/claude-code]
---

# Settings Hierarchy

## Description

A systematic map of how Claude Code resolves configuration conflicts. There are 5 settings scopes. Higher scopes win for scalar values; arrays merge. Enterprise-managed settings override everything, including `--dangerously-skip-permissions`. Understanding the hierarchy explains unexpected behavior when CLI flags seem to have no effect (managed policy overrides) or when project settings conflict with user settings.

## Mechanism

**The 5 scopes, highest to lowest priority:**

| Priority | Scope | Location | Can override |
|---|---|---|---|
| 1 (highest) | Managed | Platform-specific (distributed by admin) | Everything |
| 2 | CLI arguments | `--model`, `--permission-mode`, etc. | 3–5 |
| 3 | Local project | `.claude/settings.local.json` (gitignored) | 4–5 |
| 4 | Project | `.claude/settings.json` (committed) | 5 |
| 5 (lowest) | User | `~/.claude/settings.json` | Nothing |

**Merge rules:**
- **Scalars** (model, permissionMode, etc.): higher scope wins, lower ignored
- **Arrays** (allowedTools, denyTools, etc.): all scopes are merged together
- **Array implication**: a `denyTools` in user settings persists even if project settings don't include it

**Managed settings locations by platform:**

| Platform | Managed settings path |
|---|---|
| macOS | `/Library/Application Support/ClaudeCode/managed-settings.json` |
| Linux | `/etc/claude-code/managed-settings.json` |
| Windows | `HKLM:\Software\Anthropic\ClaudeCode\` registry |
| GitHub Actions | `managed-settings.json` in the repo (via workflow) |
| Dev container | `.devcontainer/managed-settings.json` |

**Enterprise-controlled keys in managed settings:**

| Key | Effect |
|---|---|
| `permissionMode: "bypassPermissions"` | Allow in managed mode only |
| `disableApiKeyFallback: true` | Prevent use of personal API key |
| `disableAllTelemetry: true` | Block all telemetry |
| `allowedMcpServers: [...]` | Whitelist approved MCP servers |
| `deniedMcpServers: [...]` | Block specific MCP servers |
| `allowedTools: [...]` | Whitelist specific tools |
| `denyTools: [...]` | Block specific tools org-wide |
| `trustedCertificates: [...]` | SSL certs for proxies |

**Local project scope** (`.claude/settings.local.json`):
- Gitignored by default — safe for personal overrides (API keys, debug flags)
- Wins over project settings for that developer only
- Use for: `allowedTools` experiments, personal model preference, local debug env vars

**Project scope** (`.claude/settings.json`):
- Checked in — applies to everyone on the team
- Use for: `allowedTools` for CI, project model, project-level hooks, environment variables

**User scope** (`~/.claude/settings.json`):
- Applies across all projects for this user
- Use for: personal model preference, global hooks, default permission mode

**Tool permission resolution** (additive across all scopes):
```
effective_allowed = user.allowedTools ∪ project.allowedTools ∪ local.allowedTools ∪ ...
effective_denied  = user.denyTools    ∪ project.denyTools    ∪ local.denyTools    ∪ ...
```
A tool in `denyTools` at ANY scope is denied, even if `allowedTools` at a higher scope lists it.

**Common settings fields:**

```json
{
  "model": "claude-opus-4-5",
  "smallModel": "claude-haiku-4-5-20251001",
  "permissionMode": "default",
  "allowedTools": ["Bash", "Read", "Edit"],
  "denyTools": ["WebSearch"],
  "env": { "MY_VAR": "value" },
  "hooks": { "PreToolUse": [{ "matcher": "*", "hooks": [...] }] },
  "cleanupPeriodDays": 30,
  "includeCoAuthoredBy": true,
  "outputFormat": "stream-json"
}
```

**CLAUDE.md scoped rules** (separate from settings.json but part of the configuration system):
- 5 scope levels: `~/.claude/CLAUDE.md`, project `CLAUDE.md`, `**/*.test.js` path-scope, etc.
- Path-scoped rules in CLAUDE.md apply only when Claude is working with matching files
- Do not use `~/.claude/CLAUDE.md` for truly sensitive data — it applies to all projects

## Limits

- **No per-tool granularity in managed denyTools**: you can only deny an entire tool, not "Bash except for `ls` commands"
- **Array merge is additive**: there is no "reset" operator to clear inherited array entries at a lower scope
- **Managed settings are static files**: they are not dynamically evaluated — no per-user or per-repo conditions

## Connections

- [[frameworks/permission-architecture]] — permission modes and rule syntax, separate from settings hierarchy
- [[concepts/context-and-memory]] — CLAUDE.md scoping, auto memory, project context loading

## Sources

- [[sources/2026-06-08_memory-settings-integrations]] — full hierarchy, managed settings, per-scope use cases
- [[sources/2026-06-08_permissions-security]] — enterprise keys, tool denial, managed policy
