---
type: concept
created: 2026-06-08
updated: 2026-06-08
status: current
sources_count: 3
tags: [domain/claude-code]
---

# Agentic Loop

## Description

The agentic loop is the execution cycle at the heart of Claude Code (and the Agent SDK): Claude gathers context, takes action, verifies results, and repeats — chaining tool calls until the work is done or interrupted. This is what separates an agent from a chat interface: without tools, Claude can only respond with text; with tools, it reads files, runs commands, and modifies state.

Anthropic's framing: "Claude Code is the harness, Claude is the model." The harness provides the loop machinery (tools, context, permissions); the model decides what to do each iteration.

## Development

**Three phases, repeated**

1. **Gather context** — read files, search the codebase, fetch documentation, check git state. Claude decides what it needs to understand the task.
2. **Act** — edit files, run shell commands, create commits. Actions that mutate state require explicit permission (unless auto-approved by mode).
3. **Verify results** — check output, run tests, compare against expectations. If results don't match the goal, the loop continues.

The loop terminates when Claude judges the work complete, when the user interrupts with `Esc`, or when a configured limit is hit (`max_turns`, `max_budget_usd`).

**Tool execution mechanics**
- Read-only tools (`Read`, `Glob`, `Grep`, MCP tools with `readOnlyHint`) execute in parallel
- Stateful tools (`Edit`, `Write`, `Bash`) execute sequentially
- Only tool-use turns count toward `max_turns`

**Controlling the loop**

| Mechanism | When it triggers | What it does |
|---|---|---|
| `/goal <condition>` | After each turn | Haiku evaluates condition; loop continues until satisfied |
| `Stop` hook | End of each turn | Shell/script decides whether to continue or stop |
| `/loop [interval]` | On a time schedule | Re-invokes prompt; self-paced if no interval |
| `max_turns` (SDK) | After N tool-use turns | Hard stop |
| `Esc` | Any time | Immediate interrupt, preserves work done |

**`/goal` mechanics** (v2.1.139+)
- Condition: up to 4,000 characters; include a measurable final state, a stated check, and a turn bound ("or stop after 20 turns")
- Evaluator model: Haiku by default (fast, cheap); configurable
- Auto-clears when condition is satisfied
- Non-interactive: `claude -p "/goal <condition>"` — runs to completion

**Context survival across the loop**
- CLAUDE.md and auto memory re-inject after compaction
- Path-scoped rules (`.claude/rules/*.md` with `paths:` frontmatter) are lost after compaction until the matching file is read again
- Subagent context economy: ~6,100 tokens read → ~420 tokens returned to parent — keeps the main loop's context clean

**The loop in the SDK**

The same three phases run inside every `query()` call. Message stream:
1. `SystemMessage` (subtype `init`) — setup complete
2. `AssistantMessage` — text + tool use blocks
3. `UserMessage` — tool results
4. Repeat until no tool calls
5. `AssistantMessage` + `ResultMessage` (subtype `success` or error)

Effort level (`low`/`medium`/`high`/`xhigh`/`max`) controls the thinking budget per iteration — higher effort = deeper reasoning per turn, not more turns.

## Connections

- [[features/agent-sdk]] — the SDK that exposes this loop programmatically
- [[frameworks/permission-architecture]] — permissions gate every action step in the loop
- [[concepts/context-and-memory]] — context is the working state the loop reads and writes
- [[features/headless-mode]] — non-interactive loop execution for CI and automation
- [[features/dynamic-workflows]] — external JS scripts that orchestrate the loop at scale
- [[features/hooks]] — hooks attach to specific phases (pre/post tool, stop, etc.)

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — loop mechanics, modes, goal, checkpointing
- [[sources/2026-06-08_agent-sdk-core]] — loop internals from SDK perspective, effort levels, message stream
- [[sources/2026-06-08_agent-sdk-extensions]] — hooks in the loop, tool search, permission evaluation order
