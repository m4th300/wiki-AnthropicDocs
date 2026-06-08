---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: experimental
sources_count: 1
tags: [domain/claude-code, theme/automation]
---

# Dynamic Workflows

## Description

Dynamic workflows are JavaScript scripts that orchestrate subagents at scale. When triggered, Claude writes the script; the workflow runtime executes it in the background while the session stays responsive. The script controls the loop, branching, and intermediate results — Claude's context receives only the final aggregated response. This is the mechanism for tasks that exceed what a single conversation can coordinate: sweeping an entire codebase, running parallel research across dozens of sources, or fanning out independent changes.

## Status & Requirements

- **Status**: experimental (research preview)
- **Platforms**: All
- **Min version**: part of Claude Code v2 (no specific version documented)
- **Disable**: `/config` → Dynamic workflows, or `"disableWorkflows": true` in settings, or `CLAUDE_CODE_DISABLE_WORKFLOWS=1`

## Configuration

**Trigger methods:**

1. **`ultracode` keyword**: `ultracode: audit every API endpoint under src/routes/ for missing auth checks`
2. **`/effort ultracode`**: Claude plans a workflow for every substantial task (auto-resets on new session)
3. **`/deep-research <question>`**: built-in web research workflow — fetches from multiple angles, cross-references, votes on claims, returns a cited report

**Runtime limits:**
- 16 simultaneous agents (fewer on machines with limited CPU)
- 1,000 agents total per run
- No user input during execution (only permission prompts can pause)
- Scripts cannot directly access filesystem or shell — only agents can

**Resumption:**
- If stopped, completed agents return cached results; remaining run live
- Works only within the same Claude Code session
- Exiting Claude Code during a workflow resets to zero on next launch

**Saving workflows:**
1. Open `/workflows` → select a run → press `s`
2. Saved to `.claude/workflows/` (project, shared) or `~/.claude/workflows/` (user)
3. Run as `/<name>` in future sessions; accepts inputs via an `args` global

**Comparison with other multi-agent approaches:**

| | Subagents | Skills | Agent teams | Dynamic workflows |
|---|---|---|---|---|
| Who controls execution | Claude, turn by turn | Claude, following prompt | Lead agent, turn by turn | The script |
| Intermediate results in | Claude's context | Claude's context | Shared task list | Script variables |
| Scale | Few delegated tasks | Same | Handful of long-running peers | Tens to hundreds per run |
| Resumption | Restarts the turn | Restarts | Teammates keep running | Resumed in same session |

## Use Cases

- **Codebase-wide changes**: migrate all uses of a deprecated API, audit all endpoints, enforce a naming convention across hundreds of files
- **Parallel research**: `/deep-research` — fetch from multiple sources, cross-reference, synthesize with citations
- **Independent parallel tasks**: each agent owns distinct files with no shared state
- **Plan drafting**: write multiple alternative approaches simultaneously, aggregate into one decision

## Tradeoffs

- **Context opacity**: script orchestrates agents; main conversation only sees the final output, not intermediate reasoning. Harder to debug than step-by-step subagent delegation.
- **Session-tied resumption**: stopping Claude Code mid-workflow means starting over. For long-running tasks, Routines (cloud-executed, resumable across machine restarts) may be more appropriate.
- **Experimental status**: behavior may change; not suitable for production-critical automation.
- **No user input mid-run**: only permission prompts can pause execution — cannot inject context or redirect once started.
- **`/effort ultracode` persistence**: resets on new session — can't be permanently configured.

## Connections

- [[concepts/agentic-loop]] — dynamic workflows orchestrate the loop externally via a script
- [[features/subagents]] — workflows spawn subagents at scale
- [[features/routines]] — cloud-executed alternative for tasks that must survive machine restarts
- [[features/agent-teams]] — peer-to-peer alternative at smaller scale with direct communication
- [[frameworks/agentic-patterns]] — choosing between workflow patterns

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — full workflow description, trigger methods, limits, comparison table
