---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: experimental
sources_count: 1
tags: [domain/claude-code, theme/automation]
---

# Agent Teams

## Description

Agent teams coordinate multiple independent Claude Code sessions working together on a shared task. Unlike subagents (which report to a parent only), teammates communicate peer-to-peer via a mailbox and coordinate via a shared task list. The team leader creates the team, generates teammates, and coordinates work. Status: experimental, disabled by default — requires opt-in.

## Status & Requirements

- **Status**: experimental
- **Min version**: Claude Code v2.1.32+
- **Activation**: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (in settings.json `env` or shell environment)
- **Display modes**: `in-process`, `tmux`, `auto` (default)
- **tmux/split panes**: requires tmux or iTerm2 (not VS Code, Windows Terminal, Ghostty)

## Configuration

**Activate:**
```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

**Display mode configuration** (`~/.claude/settings.json`):
```json
{ "teammateMode": "tmux" }
```
Or CLI: `claude --teammate-mode tmux`

**Architecture:**

| Component | Role |
|---|---|
| Team leader | Creates team, generates teammates, coordinates |
| Teammates | Distinct Claude Code instances |
| Shared task list | States: pending / in-progress / completed |
| Mailbox | Inter-agent messaging (peer-to-peer) |

Storage: `~/.claude/teams/{team-name}/config.json` and `~/.claude/tasks/{team-name}/`

**Invocation:** Natural language — just ask for a team: "create a team to work on this refactor, with one agent handling the database layer, one the API, and one the tests."

**Navigation:** `Shift+Down` navigates between teammates in in-process mode.

**Stop all teammates:** `Ctrl+X Ctrl+K`

**Optimal parameters:** 3-5 teammates, 5-6 tasks per teammate, distinct file ownership per teammate.

## Use Cases

- **Parallel research**: multiple teammates independently investigate different hypotheses, then report to leader
- **Inter-layer coordination**: frontend + backend + tests teammates, each owning their domain
- **New module implementation**: divide a new feature into frontend, API, database, and test teammates
- **Concurrent hypothesis testing**: debug a hard problem by running multiple approaches in parallel with direct comparison

## Tradeoffs

- **Cost**: approximately 7x more tokens than a standard session (each teammate = full Claude instance with own context). Mitigate: use Sonnet for teammates, keep teams small (3-5), clean up when done.
- **vs Subagents**: subagents report to parent only, managed entirely by parent; teammates communicate peer-to-peer with shared task list. Teams are more powerful for parallel peers that need to share state; subagents are cheaper for isolated delegated tasks.
- **Not resumable**: `/resume` doesn't restore in-process teammates after session end. If resumption is needed, use subagents or dynamic workflows.
- **File conflict risk**: teams work best when each teammate owns distinct files. Multiple agents modifying the same files causes conflicts.
- **One team at a time**: no nested teams; only one active team per leader.
- **Split pane requirements**: `tmux` display mode requires tmux or iTerm2 — not available in VS Code integrated terminal, Windows Terminal, or Ghostty.
- **Experimental**: behavior may change; task state may lag; stopping can be slow.

## Connections

- [[features/subagents]] — cheaper alternative for most parallel tasks
- [[features/dynamic-workflows]] — script-based orchestration at greater scale
- [[frameworks/agentic-patterns]] — decision guide: subagents vs. teams vs. workflows

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — architecture, activation, display modes, optimal parameters, limitations
