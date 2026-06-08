---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 2
tags: [domain/claude-code, theme/cost-optimization]
---

# Prompt Caching

## Description

Prompt caching reuses already-processed content from previous turns, avoiding re-tokenization of the full conversation prefix. Claude Code manages cache layers automatically — the system prompt is the most stable layer (rarely invalidated), project context is the middle layer, and conversation history is the most volatile. Cache hits are billed at approximately 10% of standard input token rates, making it the primary cost optimization lever for long sessions.

## Status & Requirements

- **Status**: stable
- **Platforms**: All (TTL varies by provider)

## Configuration

**Cache layers (top to bottom, each invalidating layers below):**

| Layer | Content | Invalidated when |
|---|---|---|
| System prompt | Main instructions, tool definitions, output style | Tool set changes or Claude Code update |
| Project context | CLAUDE.md, auto memory, non-path rules | Session start, `/clear`, `/compact` |
| Conversation | Messages, tool calls, tool results | Every turn |

**Actions that INVALIDATE cache:**
- `/model` change
- `/effort` change
- Enabling fast mode mid-session
- Connecting/disconnecting a non-deferred MCP server
- Enabling/disabling a non-deferred MCP plugin
- Globally denying a tool (`Bash`, `WebFetch`)
- `/compact`
- Claude Code upgrade

**Actions that PRESERVE cache:**
- Editing files
- Editing CLAUDE.md mid-session (loaded once at startup)
- Changing permission mode
- Running skills/commands
- `/recap`, rewinding conversation

**TTL by context:**

| Context | TTL |
|---|---|
| Claude subscription | 1 hour (automatic) |
| Subscription with usage credits exceeded | 5 minutes (auto-downgrade) |
| API key, Bedrock, Vertex, Foundry | 5 minutes (default) |

**Opt-in options:**
- 1h TTL on API/cloud: `ENABLE_PROMPT_CACHING_1H=1`
- Force 5m TTL: `FORCE_PROMPT_CACHING_5M=1`

**Disable selectively:**
```bash
DISABLE_PROMPT_CACHING=1            # all models
DISABLE_PROMPT_CACHING_HAIKU=1      # Haiku only
DISABLE_PROMPT_CACHING_SONNET=1     # Sonnet only
DISABLE_PROMPT_CACHING_OPUS=1       # Opus only
```

**SDK optimization — `exclude_dynamic_sections`:**
```python
ClaudeAgentOptions(system_prompt={"type":"preset","preset":"claude_code","exclude_dynamic_sections":True})
```
Moves machine-specific context (cwd, OS, shell) from system prompt to first user message → system prompt is identical across sessions → higher cache hit rate. Slight trade-off: context details are slightly less authoritative.

**Monitoring:**
- `cache_creation_input_tokens`: tokens written to cache (billed at write rate)
- `cache_read_input_tokens`: tokens served from cache (~10% of standard rate)

## Use Cases

- **Parallel sessions**: sessions in the same directory can share the project context cache
- **Long interactive sessions**: subscription users get automatic 1h TTL — warm cache across turns for cheap reads
- **CI/scripting optimization**: use `exclude_dynamic_sections` + stable `allowed_tools` to maximize cache hits across runs
- **Cost monitoring**: track `cache_read_input_tokens` vs `cache_creation_input_tokens` ratio — a low ratio means frequent invalidation

## Tradeoffs

- **Fast mode cache warning**: activating fast mode mid-session invalidates the cache for all existing context. Activate at the START of a session to avoid paying full input rates for existing content.
- **5m TTL on third-party providers**: Bedrock, Vertex, and Foundry default to 5-minute TTL. Long-running scripts with pauses > 5 minutes between turns get no benefit from caching the project context layer.
- **MCP connections invalidate**: each non-deferred MCP server connection/disconnection invalidates the system prompt layer. Use tool search (deferred) for large MCP catalogs to avoid this.

## Connections

- [[concepts/context-and-memory]] — cache layers map directly to context window layers
- [[features/agent-sdk]] — `exclude_dynamic_sections` flag for SDK cache optimization
- [[features/mcp]] — MCP tool search (deferred) preserves cache; direct server connections invalidate it

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_memory-settings-integrations]] — cache layers, TTL table, invalidation/preservation rules
- [[sources/2026-06-08_agent-sdk-core]] — `exclude_dynamic_sections`, subagent cache behavior
