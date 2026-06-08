---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
min_version: 
sources_count: 2
tags: [domain/agent-sdk]
---

# Agent SDK

## Description

The Claude Agent SDK is a Python and TypeScript library that exposes the agentic loop, tools, and context management of Claude Code as programmable building blocks. It sits between the raw Anthropic Client SDK (where you implement your own tool loop) and fully managed agents running on Anthropic's infrastructure. From June 15, 2026, Agent SDK usage on subscription plans draws from a separate monthly credit pool.

## Status & Requirements

- **Status**: stable
- **Languages**: Python (`pip install claude-agent-sdk`, requires 3.10+) | TypeScript (`npm install @anthropic-ai/claude-agent-sdk`, includes native Claude Code binary)
- **Plans**: All — separate Agent SDK credit pool from 2026-06-15
- **Min version for Opus 4.7**: v0.2.111+ (earlier versions get `thinking.type.enabled` error)

## Configuration

**Minimal Python example:**
```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="Fix the failing tests",
        options=ClaudeAgentOptions(
            permission_mode="acceptEdits",
            allowed_tools=["Read", "Edit", "Bash(npm test)"],
        )
    ):
        print(message)

asyncio.run(main())
```

**Key `ClaudeAgentOptions` fields (Python) / `Options` (TypeScript):**

| Field | Type | Description |
|---|---|---|
| `permission_mode` | `PermissionMode` | `default`/`acceptEdits`/`plan`/`dontAsk`/`bypassPermissions` |
| `allowed_tools` | `list[str]` | Auto-approved tools (e.g. `["Read", "Bash(npm test)"]`) |
| `disallowed_tools` | `list[str]` | Bare name removes tool; scoped name blocks matching calls |
| `system_prompt` | `str\|SystemPromptPreset` | `{type:"preset", preset:"claude_code"}` or custom string |
| `mcp_servers` | `dict` | MCP server configs (stdio or HTTP/SSE) |
| `max_turns` | `int` | Max tool-use turns (default: unlimited) |
| `resume` | `str` | Session ID to resume |
| `continue_conversation` | `bool` | Continue last session in cwd |
| `hooks` | `dict` | Event hook callbacks |
| `agents` | `dict` | Subagent definitions |
| `enable_file_checkpointing` | `bool` | Track Write/Edit for rewind |
| `effort` | `str` | `low`/`medium`/`high`/`xhigh`/`max` |
| `model` | `str` | Model name or alias |

**System prompt presets:**
- No option: minimal prompt (no Claude Code directives)
- `{type:"preset", preset:"claude_code"}`: full Claude Code prompt
- Custom string: user-controlled (must add security rules manually)
- Append to preset: `{type:"preset", preset:"claude_code", "append":"Always docstring methods."}`

**`exclude_dynamic_sections=True`**: moves machine-specific context (cwd, OS, shell) to first user message — makes system prompt identical across sessions, improving prompt cache hit rate.

**Production hosting:**
- Subprocess model: 1 session = 1 `claude` subprocess; N concurrent sessions = N subprocesses
- Recommended per agent: 1 GiB RAM, 5 GiB disk, 1 CPU
- Multi-tenant isolation: set `cwd` + `CLAUDE_CONFIG_DIR` per tenant; use `setting_sources=[]`
- Session models: ephemeral (one container per task), long-lived (`ClaudeSDKClient`/`startup()`), hybrid (ephemeral + `SessionStore`)

## Use Cases

- **CI automation**: headless code review, test repair, migration scripts
- **Custom coding tools**: branded agents with specific constraints and toolsets
- **Multi-agent orchestration**: fan-out research, parallel codebase analysis (use `Workflow` tool for 50+ agents)
- **Embedded agents**: integrate Claude Code's agentic loop into web apps or backend services

## Tradeoffs

**vs raw Anthropic Client SDK**: SDK handles the entire tool execution loop, session persistence, and permission UI; Client SDK gives more control but requires reimplementing all of that.

**vs fully managed agents**: SDK runs locally (or on your infrastructure), giving access to local MCP servers, filesystem, and full tool suite; managed agents run on Anthropic's infrastructure and are stateless.

**Streaming vs single-turn**:
- Streaming (`ClaudeSDKClient` / `startup()` + AsyncIterable): supports images, dynamic message queueing, real-time feedback, full tool integration
- Single-turn (plain string): simpler, no image attachments, no dynamic interruption

**Cost**: from 2026-06-15, draws from a separate credit pool on subscription plans — monitor `ResultMessage` cost fields to track spend.

## Connections

- [[concepts/agentic-loop]] — the SDK exposes the same agentic loop used by Claude Code
- [[features/extension-model]] — hooks, MCP, subagents, and skills are all available in the SDK
- [[frameworks/permission-architecture]] — permission modes and evaluation order in SDK context
- [[concepts/context-and-memory]] — session storage, CLAUDE.md loading, compaction

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_agent-sdk-core]] — core architecture, Python/TS differences, sessions, hosting
- [[sources/2026-06-08_agent-sdk-extensions]] — hooks, MCP, custom tools, permissions, tool search
