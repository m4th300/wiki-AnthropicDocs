---
type: source
created: 2026-06-08
updated: 2026-06-08
status: published
author: Anthropic
date_source: 2026-06-08
raw_file: raw/memory.md, raw/sessions.md, raw/prompt-caching.md, raw/settings.md, raw/env-vars.md, raw/model-config.md, raw/server-managed-settings.md, raw/admin-setup.md, raw/amazon-bedrock.md, raw/google-vertex-ai.md, raw/microsoft-foundry.md, raw/llm-gateway.md, raw/github-actions.md, raw/github-enterprise.md, raw/gitlab-ci-cd.md, raw/network-config.md, raw/channels.md, raw/channels-reference.md
tags: [domain/claude-code, theme/integrations]
sources_count: 18
---

# Memory, Settings & Integrations

## Résumé

This cluster covers the persistence and configuration infrastructure of Claude Code: a five-scope settings hierarchy (managed → CLI → local → project → user), two memory systems (CLAUDE.md for human instructions, auto memory for Claude's own notes), prompt caching mechanics that directly impact cost, and the full integration surface — three cloud providers (Bedrock, Vertex, Foundry), LLM gateway support, GitHub Actions, GitHub Enterprise, GitLab CI/CD, and the Channels system for event-driven agent triggering.

## Points clés

**Settings hierarchy** (5 scopes, highest to lowest priority)
1. Managed policy — `/etc/claude-code/managed-settings.json` (Linux) / OS equivalents
2. CLI args — session only
3. Local — `.claude/settings.local.json`
4. Project — `.claude/settings.json`
5. User — `~/.claude/settings.json`

Arrays merge across levels; scalars: higher scope wins. Drop-in directory: `/etc/claude-code/managed-settings.d/` (files merged alphabetically). Server-managed settings (Teams/Enterprise): delivered from `claude.ai/admin-settings`, polled hourly, applied on next startup.

**Key settings fields**: `model`, `outputStyle`, `effortLevel`, `autoMemoryEnabled`, `autoMemoryDirectory`, `cleanupPeriodDays`, `claudeMdExcludes`, `permissions` (allow/deny), `env`, `availableModels` (restrict selector), `minimumVersion` (enforce min version), `disableAllHooks`, `autoMode`

**Model aliases and default models by plan**

| Alias | Anthropic API | Bedrock/Vertex/Foundry |
|---|---|---|
| `opus` | Opus 4.8 | Opus 4.6 |
| `sonnet` | Sonnet 4.6 | Sonnet 4.5 |
| `haiku` | Haiku 4.5 | Haiku 4.5 |

Effort levels: `low/medium/high/xhigh/max`. Default: `high` (Opus 4.8/4.6, Sonnet 4.6); `xhigh` (Opus 4.7). `xhigh` falls back to highest supported if unsupported.

**Prompt caching** (direct cost impact)
- Cache layers: system prompt → project context (CLAUDE.md, memory, rules) → conversation
- Change in one layer invalidates all layers below
- Cache-invalidating actions: model change, effort change, fast mode toggle, non-deferred MCP connect/disconnect, globally denying a tool, `/compact`, upgrade
- TTL: 1 hour (subscription), 5 minutes (API key / Bedrock / Vertex / Foundry)
- Opt into 1h TTL on API: `ENABLE_PROMPT_CACHING_1H=1`

**Cloud providers**

| Provider | Enable | Auth | Limitation |
|---|---|---|---|
| Amazon Bedrock | `CLAUDE_CODE_USE_BEDROCK=1` + `AWS_REGION` | `aws configure` / env vars / SSO / API key | No WebSearch, 5m cache TTL |
| Google Vertex AI | `CLAUDE_CODE_USE_VERTEX=1` + `CLOUD_ML_REGION` + `ANTHROPIC_VERTEX_PROJECT_ID` | `gcloud auth application-default login` | MCP Tool Search disabled by default, 5m cache TTL |
| Microsoft Foundry | `CLAUDE_CODE_USE_FOUNDRY=1` + `ANTHROPIC_FOUNDRY_RESOURCE` | `ANTHROPIC_FOUNDRY_API_KEY` or Entra ID | No interactive setup, 5m cache TTL |

**LLM Gateway** — any proxy implementing Anthropic Messages / Bedrock InvokeModel / Vertex rawPredict format
- `ANTHROPIC_BASE_URL=https://gateway.example.com` for Anthropic format
- LiteLLM: per-provider passthrough via `ANTHROPIC_BEDROCK_BASE_URL`/`ANTHROPIC_VERTEX_BASE_URL` + `CLAUDE_CODE_SKIP_*_AUTH=1`
- Warning: LiteLLM versions 1.82.7 and 1.82.8 were compromised with malware

**GitHub Actions** (`anthropics/claude-code-action@v1`)
- Trigger: comment `@claude` on PR or issue → Claude creates PRs, implements features, fixes bugs
- Quick setup: `/install-github-app` in Claude Code terminal
- Key inputs: `anthropic_api_key`, `prompt`, `claude_args`, `trigger_phrase` (default: `@claude`), `use_bedrock`, `use_vertex`
- Scheduled runs: use `schedule:` trigger with `prompt:` input
- OIDC auth available for both Bedrock and Vertex AI (no static secrets needed)

**GitHub Enterprise Server**
- Teams and Enterprise plans only
- Admin setup: connect via `claude.ai/admin-settings/claude-code` → enter hostname → create GitHub App
- Features on GHES: Code Review, Claude Security, Routines, Teleport, contribution metrics, plugin marketplaces
- Plugin marketplaces: use full git URL (not `owner/repo` shorthand)

**GitLab CI/CD** (beta, maintained by GitLab)
- `before_script`: install Claude Code + `/bin/gitlab-mcp-server`
- Input via `AI_FLOW_INPUT` variable, optional `AI_FLOW_CONTEXT`
- OIDC auth available for Bedrock and Vertex AI

**Channels** (research preview, v2.1.80+)
- A channel is an MCP server that pushes events into an active Claude Code session via stdin
- Built-in: Telegram bot, Discord bot, iMessage (macOS)
- Types: unidirectional (Claude receives, acts) and bidirectional (Claude can reply)
- Permission relay (v2.1.81+): channel can approve Bash/Write/Edit requests from a remote device
- Enterprise: blocked by default; admin enables via `channelsEnabled: true` in managed settings
- Security: always verify sender identity (`message.from.id`); unverified channels = prompt injection vector

## Liens wiki

- [[concepts/context-and-memory]] — CLAUDE.md scopes, auto memory, session storage
- [[features/prompt-caching]] — cache layer details
- [[features/sessions]] — session storage and continuation
- [[frameworks/settings-hierarchy]] — detailed settings system
- [[frameworks/ci-cd-integration]] — GitHub Actions / GitLab CI patterns

## Tensions

Aucune tension identifiée.

## Questions soulevées

- How does `forceRemoteSettingsRefresh: true` interact with offline/disconnected environments? What's the UX when the fetch fails?
- Are Channels available in the Agent SDK? The docs describe them in Claude Code context — is there an SDK equivalent?
