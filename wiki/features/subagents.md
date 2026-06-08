---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 2
tags: [domain/claude-code, theme/automation]
---

# Subagents

## Description

Subagents are specialized Claude instances that run in their own context window, perform a delegated task, and return a summary to the parent. They are the primary mechanism for context isolation: a task that would flood the main session with file contents or search results gets contained, and only the useful output returns. Claude Code ships three built-in subagents (Explore, Plan, General-purpose) and supports custom definitions via markdown files.

## Status & Requirements

- **Status**: stable (custom subagents); experimental (`isolation: worktree` for git worktree isolation)
- **Platforms**: All

## Configuration

**Custom subagent definition** (`.claude/agents/<name>.md` for project, `~/.claude/agents/<name>.md` for user):

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities. Use when reviewing auth, input validation, or cryptography changes.
tools: Read, Grep, Glob, Bash
model: opus
isolation: worktree
skills: my-security-skill
effort: high
---

You are a senior security engineer. Focus on...
```

**Frontmatter fields:**

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Identifier and invocation name |
| `description` | Yes | What Claude reads to decide when to delegate — write carefully |
| `tools` | No | Inherits parent tools if omitted |
| `model` | No | Aliases: `sonnet`, `opus`, `haiku`, `inherit` |
| `isolation` | No | `worktree` — isolated git worktree (auto-deleted if no changes) |
| `skills` | No | Skills to preload |
| `effort` | No | Effort level override |
| `maxTurns` | No | Max turns for this subagent |
| `background` | No | Run non-blocking by default |

**Built-in subagents:**
- **Explore**: codebase search (reads without modifying). Omits CLAUDE.md and git status.
- **Plan**: architecture and planning. Omits CLAUDE.md and git status.
- **General-purpose**: delegated tasks when no specific subagent matches.

**Invocation (natural language):** "use a subagent to investigate how our auth system handles token refresh" — Claude picks the right subagent based on description matching.

**Fork subagent**: a special type that inherits the full parent context. Useful for verification ("does my implementation match the plan?"). `claude --continue --fork-session` from CLI.

**Background vs foreground:**
- Background (default): subagent runs independently while parent continues
- Foreground: "run this subagent in the foreground" in prompt, or `--foreground` flag
- Stop all background subagents: `Ctrl+X Ctrl+K`

**What subagents inherit:**
- ✅ Their system prompt + invocation task prompt
- ✅ Project CLAUDE.md
- ✅ MCP tool definitions (if listed in `tools`)
- ✅ Skills (if listed in `skills`)
- ❌ Parent's conversation history
- ❌ Parent's system prompt
- ❌ Skills not in `skills` list

**SDK usage** (`agents` option key):
```python
ClaudeAgentOptions(
    agents={
        "security-reviewer": AgentDefinition(
            description="Reviews code for security issues",
            prompt="You are a senior security engineer...",
            tools=["Read", "Grep", "Glob"],
            model="opus",
        )
    },
    allowed_tools=["Read", "Edit", "Agent"]  # "Agent" enables subagent invocation
)
```

**Key constraint**: subagents cannot spawn their own subagents. Do not include `"Agent"` in a subagent's `tools` list.

## Use Cases

- **Context isolation**: reads, searches, and analysis that would pollute the main conversation — delegate and get only the summary
- **Parallel investigation**: spawn multiple subagents to explore different hypotheses simultaneously
- **Specialized reviewers**: security reviewer, test writer, documentation writer — each with domain-specific instructions and tools
- **Worktree-isolated work**: `isolation: worktree` creates a temporary git worktree for the subagent — no risk of partial changes leaking into main branch
- **Background research**: delegate investigation while main session continues coding

## Tradeoffs

- **vs Agent teams**: subagents report to parent only; agent teams communicate peer-to-peer and share a task list. Teams cost ~7x more. Use subagents for isolated tasks; teams for parallel peers that need to coordinate.
- **vs Dynamic workflows**: subagents are managed by Claude turn-by-turn; workflows are JS scripts that hold the loop and can coordinate hundreds of agents with intermediate state. Use workflows for scale.
- **Context isolation is one-way**: subagent cannot read parent history, but parent receives the summary. The summary quality depends on how the subagent is prompted.
- **`isolation: worktree` cleanup**: auto-deleted if no changes (Claude asks first if session was named). Changes → Claude asks to keep or delete. Manual: `git worktree remove`.

## Connections

- [[concepts/agentic-loop]] — subagents run their own agentic loop and return a summary
- [[features/agent-teams]] — experimental peer-to-peer alternative to subagents
- [[features/dynamic-workflows]] — script-based orchestration of agents at scale
- [[features/extension-model]] — subagents in the extension mechanism comparison
- [[features/agent-sdk]] — `AgentDefinition` type in SDK, `Workflow` tool for large-scale coordination

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — built-in subagents, custom definition format, fork, background
- [[sources/2026-06-08_agent-sdk-core]] — `AgentDefinition` fields, SDK subagent mechanics, `Workflow` tool
