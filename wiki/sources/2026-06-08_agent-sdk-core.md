---
type: source
created: 2026-06-08
updated: 2026-06-08
status: published
author: Anthropic
date_source: 2026-06-08
raw_file: raw/agent-sdk-overview.md, raw/agent-sdk-overview-page.md, raw/agent-sdk-quickstart.md, raw/agent-sdk-python.md, raw/agent-sdk-typescript.md, raw/agent-sdk-agent-loop.md, raw/agent-sdk-hosting.md, raw/agent-sdk-streaming-vs-single-mode.md, raw/agent-sdk-sessions.md, raw/agent-sdk-session-storage.md, raw/agent-sdk-file-checkpointing.md, raw/agent-sdk-subagents.md, raw/agent-sdk-modifying-system-prompts.md
tags: [domain/agent-sdk]
sources_count: 13
---

# Agent SDK — Core Architecture

## Résumé

The Claude Agent SDK (Python: `claude-agent-sdk`, TypeScript: `@anthropic-ai/claude-agent-sdk`) exposes the same agentic loop, tools, and context management that power Claude Code as a programmable library. The central entry point is a `query()` function returning an async generator of typed messages. The SDK runs Claude Code as a managed subprocess, handling stdio transport, tool execution, and session persistence automatically. From June 15, 2026, Agent SDK usage on subscription plans draws from a separate monthly credit pool distinct from interactive usage.

## Points clés

**Core mechanics**
- `query()` → async generator of: `SystemMessage`, `AssistantMessage`, `UserMessage`, `ResultMessage`
- Built-in tools: `Read`, `Write`, `Edit`, `Bash`, `Monitor`, `Glob`, `Grep`, `WebSearch`, `WebFetch`, `AskUserQuestion`
- Tool execution: read-only tools can run in parallel; `Edit`/`Write`/`Bash` are sequential
- Effort levels: `low`, `medium`, `high`, `xhigh` (Opus 4.7 default), `max`
- `ResultMessage` subtypes: `success`, `error_max_turns`, `error_max_budget_usd`, `error_during_execution`

**Python vs TypeScript**
- Python: `ClaudeAgentOptions` (snake_case), `ClaudeSDKClient` class for persistent connections
- TypeScript: plain `Options` object (camelCase), `startup()` warmup function
- Assistant content: Python `message.content`, TypeScript `message.message.content` (extra `.message`)
- Python has `"dontAsk"` permission mode; TypeScript has `"multiStep"` and `"free"` instead
- TypeScript default effort: `"high"`; Python: unset

**Sessions**
- Stored as JSONL at `~/.claude/projects/<encoded-cwd>/`
- Continue last: `continue_conversation=True` / `continue: true`
- Resume by ID: `resume=session_id` / `resume: sessionId`
- Fork: `fork_session=True` / `forkSession()`
- External `SessionStore` interface for multi-host deployments (S3, Redis, Postgres adapters in SDK repo)

**File checkpointing** (`enable_file_checkpointing`)
- Tracks Write/Edit/NotebookEdit only — NOT Bash
- Checkpoint ID = `UserMessage.uuid`; rewind: `client.rewind_files(checkpoint_id)`
- Incompatible with `session_store` and `persistSession: false`

**Streaming vs single-turn**
- Streaming (`ClaudeSDKClient` / `startup()` + AsyncIterable): supports images, dynamic queueing, real-time feedback
- Single-turn (plain string prompt): simpler, no image attachments, no dynamic interruption

**Subagents** (`agents` option key)
- `AgentDefinition` fields: `description`, `prompt`, `tools`, `model`, `skills`, `maxTurns`, `background`, `permissionMode`
- Subagents cannot spawn their own subagents
- Enable: add `"Agent"` to parent's `allowedTools`
- `Workflow` tool (TypeScript v0.3.149+) for coordinating hundreds of agents

**System prompt configuration**
- No option: minimal prompt
- `{type: "preset", preset: "claude_code"}`: full Claude Code prompt
- Custom string: user-controlled, must include security rules manually
- `exclude_dynamic_sections=True`: moves machine context to first user message, improves cache hit rate

**Production hosting**
- Subprocess model: 1 session = 1 `claude` subprocess; N concurrent = N subprocesses
- Recommended resources: 1 GiB RAM, 5 GiB disk, 1 CPU per agent
- Multi-tenant: set `cwd` + `CLAUDE_CONFIG_DIR` per tenant, use `setting_sources=[]`
- Observability: `CLAUDE_CODE_ENABLE_TELEMETRY=1` + OTEL env vars

## Liens wiki

- [[features/agent-sdk]] — this source directly populates this page
- [[concepts/agentic-loop]] — the loop that the SDK exposes programmatically
- [[frameworks/permission-architecture]] — permission modes and tool approval in SDK context
- [[features/sessions]] — session persistence and continuity mechanics

## Tensions

Aucune tension identifiée avec le contenu existant du wiki.

## Questions soulevées

- How does the SDK's separate credit pool (from 2026-06-15) interact with bypassPermissions mode and enterprise billing?
- What is the practical performance difference between streaming and single-turn mode for short tasks?
