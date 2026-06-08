---
type: concept
created: 2026-06-08
updated: 2026-06-08
status: current
sources_count: 3
tags: [domain/claude-code]
---

# Context and Memory

## Description

Context is the working memory of a Claude Code session: everything the model can "see" in a single turn — conversation history, file contents, command outputs, instructions, tool schemas. Memory is the subset that persists across turns and sessions, divided into CLAUDE.md files (human-written, static) and auto memory (Claude-written, dynamic). Understanding what survives compaction and what doesn't is critical for designing reliable long-running sessions.

## Development

**Context window composition** (approximate startup token counts)

| Element | Tokens | Visibility |
|---|---|---|
| System prompt | ~4,200 | Not shown in terminal |
| Project CLAUDE.md | ~1,800 | Not shown |
| Auto memory (MEMORY.md) | ~680 | Not shown (200 lines or 25 KB max) |
| Environment info | ~280 | Not shown |
| MCP tool names (deferred) | ~120 | Not shown |
| Skill descriptions | ~450 | Not shown |
| User CLAUDE.md | ~320 | Not shown |

Everything else (file reads, command outputs, conversation turns) accumulates dynamically.

**What survives compaction**

| Element | After `/compact` |
|---|---|
| System prompt & output style | Unchanged — not in message history |
| Root CLAUDE.md & unscoped rules | Re-injected from disk |
| Auto memory | Re-injected from disk |
| Invoked skill bodies | Re-injected, capped at 5,000 tokens/skill, 25,000 total |
| Path-scoped rules (`paths:` frontmatter) | **Lost** — re-loaded only when matching file is read |
| Nested CLAUDE.md in subdirectories | **Lost** — re-loaded when a file in that subdirectory is read |
| Hooks | N/A — run as code, not stored in context |

**What invalidates the prompt cache** (important for cost)
- Model change, effort change, enabling fast mode
- Connecting/disconnecting a non-deferred MCP server
- Globally denying a tool (`Bash`, `WebFetch`)
- `/compact` or Claude Code upgrade
- Does NOT invalidate: file edits, CLAUDE.md edits mid-session, changing permission mode, running skills

**CLAUDE.md scopes (widest to narrowest)**

| Scope | Path | Loaded at |
|---|---|---|
| Managed policy | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) | Startup |
| User | `~/.claude/CLAUDE.md` | Startup |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Startup |
| Local | `./CLAUDE.local.md` | Startup |
| Subdirectory | `foo/bar/CLAUDE.md` | On demand, when reading files in that dir |

All files are **concatenated** (not replaced). CLAUDE.md changes mid-session take effect only after the next session start (files are loaded once at startup).

**`.claude/rules/` — path-scoped rules**

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# Rules that apply only to API files
```

Loaded on demand when Claude reads matching files; lost after compaction until re-triggered. Rules without `paths:` load at startup like CLAUDE.md.

**Auto memory**

| Property | Value |
|---|---|
| Storage | `~/.claude/projects/<project>/memory/` |
| Scope | Per git repository; shared across worktrees |
| Loaded at startup | First 200 lines or 25 KB of MEMORY.md (concise index) |
| Other files | On demand |

Claude writes to auto memory; humans typically don't edit it directly. Disable: `autoMemoryEnabled: false` or `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`.

**Session persistence**

Sessions are stored as JSONL at `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`. Retention: 30 days by default (`cleanupPeriodDays`). The conversation history is stored — not filesystem state. For filesystem state, use checkpointing.

**Subagent context economy**

A spawned subagent gets its own context window with its system prompt, project CLAUDE.md, MCP tools, skills, and the task prompt — but NOT the parent's conversation history. Typical: 6,100 tokens consumed by subagent → 420 tokens returned as summary. This isolation is key for cost efficiency in long sessions.

**Commands**

| Command | What it shows/does |
|---|---|
| `/context` | Live usage breakdown with optimization suggestions |
| `/memory` | Which CLAUDE.md and auto memory files loaded at startup |
| `/compact [instructions]` | Trigger manual compaction with optional focus |
| `Esc+Esc` → "Summarize" | Summarize a portion of the conversation |

## Connections

- [[concepts/agentic-loop]] — context is the working state the loop reads and writes
- [[frameworks/permission-architecture]] — permissions settings are part of the loaded context
- [[features/sessions]] — session storage and continuation mechanics
- [[features/prompt-caching]] — how context layers map to cache TTL and cost
- [[features/agent-sdk]] — SDK exposes the same context model; `exclude_dynamic_sections` flag for cache optimization

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — context window composition, compaction, subagent economy
- [[sources/2026-06-08_memory-settings-integrations]] — CLAUDE.md scopes, auto memory structure, session storage
- [[sources/2026-06-08_agent-sdk-core]] — SDK session model, `SessionStore`, file checkpointing
