---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 2
tags: [domain/claude-code, theme/extensions]
---

# Extension Model

## Description

Claude Code's extension model provides six distinct mechanisms for customizing and extending its behavior. Each operates at a different point in the agentic loop, has a different context cost, and a different scope (session, project, user, or organization). Choosing the right extension type avoids both under-engineering (hardcoding instructions) and over-engineering (building a plugin when a CLAUDE.md rule suffices).

## Status & Requirements

- **Status**: stable (all mechanisms); experimental for some features within mechanisms
- **Platforms**: All

## Configuration

**Extension mechanisms compared:**

| Mechanism | Runs when | Context cost | Scope | Best for |
|---|---|---|---|---|
| **CLAUDE.md** | Always (loaded at startup) | Always in context | Project / User | Persistent instructions, conventions, domain knowledge |
| **Skills** | On invocation or auto-detected | On demand (5K/skill cap) | Project / User / Plugin | Workflows, multi-step procedures, specialized tasks |
| **MCP servers** | Available as tools | Tool names at startup; full schema deferred | Project / User / Org | Connecting Claude to external services and data |
| **Subagents** | When delegated by Claude | Summary only (≈420 tok) | Project / User | Parallel tasks, context isolation, specialized roles |
| **Hooks** | Lifecycle events (pre/post tool, stop, etc.) | None — run as code | Project / User / Org | Automation, validation, permission control, notifications |
| **Plugins** | When installed and enabled | Bundled components | User / Project / Org | Packaging multiple extensions for sharing |

**Skill vs Subagent:**

| | Skill | Subagent |
|---|---|---|
| Runs in | Main context | Own context window |
| Reads files from | Main context | Own exploration |
| Returns to main | Instructions continue | Summary only |
| Use when | Workflow to follow | Task to isolate |

**Hook vs Skill:**

| | Hook | Skill |
|---|---|---|
| Trigger | Lifecycle event | User or model invocation |
| Can block actions | Yes | No |
| Has tool access | Via `agent` handler type | Yes |
| Use when | Enforcement, logging | Workflows |

**Agent teams vs Subagents:**

| | Subagents | Agent teams |
|---|---|---|
| Communication | Child reports to parent only | Peer-to-peer messaging |
| Coordination | Parent manages entirely | Shared task list |
| Cost | ~1x per subagent | ~7x (each teammate = full instance) |
| Status | Stable | Experimental |
| Use when | Delegated tasks | Parallel peers with shared state |

## Use Cases

- **CLAUDE.md**: project conventions, style guides, unknown Bash commands, architectural decisions. Keep under 200 lines.
- **Skills**: multi-step workflows (deploy, review, debug), domain-specific procedures, packaged prompts
- **MCP**: GitHub issues, Slack messages, databases, browser automation, any external service with an MCP server
- **Subagents**: codebase-wide searches (isolates reads), parallel independent tasks, specialized reviewers
- **Hooks**: pre-commit validation, security pattern scanning, external notifications, dynamic permission decisions
- **Plugins**: team-distributed skill collections, enterprise-managed extension bundles

## Tradeoffs

- **CLAUDE.md** loads every session — large files (>200 lines) reduce model effectiveness; move specialized content to skills or rules
- **Skills** are loaded on demand but add ~5,000 tokens to context when invoked; oldest removed first during compaction
- **MCP** tool schemas are deferred by default (only names load at startup); disable with `CLAUDE_CODE_DISABLE_MCP_TOOL_SEARCH=1` to pre-load all schemas
- **Subagents** add latency (separate context window setup) but dramatically reduce token usage for investigation tasks
- **Hooks** execute on every matching lifecycle event — shell handler hooks add latency to every tool call if broadly matched
- **Plugins** are the heaviest packaging unit; prefer standalone skills/agents for experimental work

## Connections

- [[concepts/agentic-loop]] — extensions attach to specific phases of the loop
- [[concepts/context-and-memory]] — each extension has a different context cost and compaction behavior
- [[frameworks/permission-architecture]] — hooks can override permission decisions; managed settings restrict extensions
- [[features/hooks]] — hooks mechanism in depth
- [[features/mcp]] — MCP integration in depth
- [[features/agent-sdk]] — all extension mechanisms are available in the SDK

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — features-overview: extension taxonomy, comparison tables, context costs
- [[sources/2026-06-08_permissions-security-extensions]] — hooks, MCP, plugins, skills in depth
