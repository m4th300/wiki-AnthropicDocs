---
type: source
created: 2026-06-08
updated: 2026-06-08
status: published
author: Anthropic
date_source: 2026-06-08
raw_file: raw/permissions.md, raw/permission-modes.md, raw/sandboxing.md, raw/sandbox-environments.md, raw/security.md, raw/security-guidance.md, raw/data-usage.md, raw/zero-data-retention.md, raw/authentication.md
tags: [domain/claude-code, theme/security]
sources_count: 9
---

# Permissions, Sandboxing & Security

## Résumé

Claude Code's security model combines a layered permission system (allow/deny/ask rules + 6 modes), OS-level Bash sandboxing (Seatbelt on macOS, bubblewrap on Linux/WSL2), prompt injection protections, and configurable data retention policies. The `bypassPermissions` mode is the escape hatch for fully automated runs — it requires a non-root user and is only appropriate inside a container or VM. Enterprise plans gain Zero Data Retention (ZDR) and managed permission controls.

## Points clés

**Permissions**
- 6 modes: `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions` (see [[frameworks/permission-architecture]])
- Config format in `settings.json`: `{"permissions": {"allow": [...], "deny": [...]}}`
- Bare tool name (`Bash`) removes from context; scoped pattern (`Bash(rm *)`) allows tool but denies matching calls
- Protected paths: never auto-approved (`.git`, `.vscode`, `.env`, etc.) except in `bypassPermissions`

**Sandboxing** (Bash commands only — does NOT cover Read/Edit/Write/MCP/hooks)
- macOS: Seatbelt; Linux/WSL2: bubblewrap (`sudo apt-get install bubblewrap socat`)
- Enable: `/sandbox` in-session
- Configurable filesystem (allowWrite, denyRead) and network (allowedDomains, deniedDomains) constraints
- `failIfUnavailable: true` (managed settings): hard error if sandbox unavailable
- Security limitations: TLS not terminated at proxy (HTTPS content not inspected), domain fronting possible, Unix sockets = full host access

**Security model**
- Prompt injection protections: contextual analysis, input sanitization, network request approval, isolated context windows for WebFetch
- `auto` mode classifier: blocks `curl | bash`, production deploys, bulk cloud deletions, IAM grants, push to main
- Encrypted credential storage: macOS Keychain, Linux `~/.claude/.credentials.json` (0600), Windows `%USERPROFILE%\.claude\.credentials.json`
- WebFetch: hostname checked against Anthropic blocklist before every fetch

**Data usage**
- Training: only Free/Pro/Max with setting enabled; Team/Enterprise/API/cloud providers: NOT trained unless explicit opt-in
- Retention: Free/Pro/Max (no training): 30 days; Enterprise: 30 days; Enterprise ZDR: zero persistence
- Local session transcripts: 30 days by default (`cleanupPeriodDays`)
- Telemetry disable: `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`, `DISABLE_TELEMETRY`, `DISABLE_ERROR_REPORTING`

**Zero Data Retention (ZDR)**
- Enterprise plan only, direct Anthropic platform only (not Bedrock/Vertex/Foundry)
- Prompts and responses not retained after response returns
- Features disabled under ZDR: Claude Code on the web, Remote sessions, `/feedback`
- Analytics still collects metadata (not prompts/responses)

**Authentication** (priority order, first match wins)
1. Cloud provider credentials (Bedrock/Vertex/Foundry env vars)
2. `ANTHROPIC_AUTH_TOKEN` (`Bearer` header — for LLM gateways)
3. `ANTHROPIC_API_KEY` (`X-Api-Key` — direct API)
4. `apiKeyHelper` script (for dynamic/rotating credentials)
5. `CLAUDE_CODE_OAUTH_TOKEN` (long-lived token from `claude setup-token`)
6. OAuth subscription credentials from `/login` (default)

**Security guidance plugin** (`security-guidance@claude-plugins-official`)
- Layer 1: pattern matching on every file edit (no cost): `eval(`, `os.system`, `.innerHTML =`, `.github/workflows/` etc.
- Layer 2: end-of-turn diff review (separate Claude call)
- Layer 3: agentic review on each `git commit`/`git push` (max 20/hour)
- Custom patterns: `.claude/security-patterns.yaml` (max 50 rules, YAML/JSON/YAML)
- Custom guidance: `.claude/claude-security-guidance.md` (max 8 KB)

## Liens wiki

- [[frameworks/permission-architecture]] — the core permission model
- [[features/sandboxing]] — OS-level Bash sandbox detail
- [[features/hooks]] — programmatic permission decisions via PreToolUse hook

## Tensions

Aucune tension identifiée.

## Questions soulevées

- When `auto` mode falls back (after 20 blocks), does it notify the user, or silently switch to a different mode?
- Does the security guidance plugin's pattern matching cover Jupyter notebook edits (`NotebookEdit` tool)?
