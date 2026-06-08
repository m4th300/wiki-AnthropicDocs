---
type: framework
created: 2026-06-08
updated: 2026-06-08
status: current
sources_count: 3
tags: [domain/claude-code, theme/automation]
---

# Agentic Patterns

## Description

A decision framework for choosing between Claude Code's multi-agent coordination mechanisms: Subagents, Agent Teams, Dynamic Workflows, and the Agent SDK. Each mechanism has a distinct tradeoff profile across cost, complexity, scalability, and resumability. Choosing the wrong mechanism often means 7x unnecessary cost (teams instead of subagents) or a fragile workaround (subagents instead of a script-driven workflow).

## Mechanism

**Primary decision tree:**

```
One task, fully delegatable?
  → Subagent (cheapest isolated delegation)

Multiple parallel tasks, each needing distinct file ownership + peer communication?
  → Agent Teams (experimental, 7x cost, peer-to-peer mailbox)

Multiple tasks, large scale (>10 agents) or need scripting logic?
  → Dynamic Workflows (JS script, up to 1,000 agents)

Need to run without session (machine off, scheduled)?
  → Routines (cloud) or GitHub Actions

Need to invoke Claude Code from your own code?
  → Agent SDK
```

**Full comparison table:**

| Dimension | Subagents | Agent Teams | Dynamic Workflows | Agent SDK |
|---|---|---|---|---|
| Max concurrent agents | 10 (interactive) | 10 | 16 concurrent / 1,000 total | Unlimited (your infra) |
| Communication | Parent → child only | Peer-to-peer mailbox | Orchestrator script | Your logic |
| Session-tied | Yes | Yes | Yes (resumable via ID) | No |
| Resumable after session end | No | No | Yes (ID-based) | N/A |
| Experimental | No | Yes | Yes (ultracode) | Stable |
| Cost multiplier | ~1x per agent | ~7x total | ~1x per agent | ~1x per agent |
| Setup | Natural language | Natural language | JS script | Code |
| Parallel file editing | Yes (separate files) | Yes (with ownership rules) | Yes | Yes |
| Nested orchestration | No (no nested subagents) | No (one team) | Yes (hierarchical) | Yes |

**When subagents are the right choice:**
- Isolated, parallelizable sub-tasks (e.g., research 5 APIs, run 5 test scenarios)
- Delegating a well-defined unit to a specialized agent definition
- Controlling context cost (subagents use ~420 tokens in parent context vs. 6,100 for full tools)
- Reviewer pattern: fresh context for review pass

**When agent teams are the right choice:**
- 3-5 teammates with distinct file ownership
- Tasks that require peer communication (not just reporting to parent)
- Interactive, in-session coordination where you want to watch all panes
- Budget allows 7x token overhead

**When dynamic workflows are the right choice:**
- Scale: more than ~10 parallel agents
- Conditional logic: if/while/for that Claude can't express naturally
- Need to save and resume a complex workflow across multiple days
- Already writing a JS orchestrator for other reasons (CI, scripting)

**When Agent SDK is the right choice:**
- Embedding Claude Code in your application (chat UI, IDE extension, SaaS)
- Hosting on your own infrastructure (cloud function, container)
- Need typed API, error handling, event stream
- SessionStore for multi-host deployments

**Cost optimization principles:**
1. Use Haiku as the evaluator model in subagent pipelines (cheap verification)
2. Use Sonnet for teammates (not Opus) in agent teams
3. Limit agent teams to 3-5 members; 5-6 tasks each
4. Dynamic workflows for scale; don't create 20 subagents manually
5. `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` only when team semantics are actually needed

**Common anti-patterns:**
- Creating an agent team for a task that one subagent could do → 7x unnecessary cost
- Manually orchestrating 20+ subagents in conversation → use a dynamic workflow
- Using dynamic workflows for a simple 2-step delegation → use a subagent
- Calling Agent SDK for a simple one-off script → use `claude -p`

## Limits

- **No nested subagents**: a subagent cannot spawn additional subagents. Workaround: use dynamic workflows for hierarchical orchestration.
- **No nested teams**: only one active team per leader.
- **Dynamic workflows**: session-tied resumption — if the session ends, resuming requires the workflow file.
- **Agent SDK**: `canUseTool` callback runs in the same process — expensive evaluations block the agent loop.

## Connections

- [[features/subagents]] — subagent configuration and inherits/doesn't-inherit table
- [[features/agent-teams]] — teams architecture, cost, activation
- [[features/dynamic-workflows]] — ultracode trigger, runtime limits, resumption
- [[features/agent-sdk]] — API surface, hosting, SessionStore
- [[features/routines]] — cloud-executed automation

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — dynamic workflows, agent teams, limits
- [[sources/2026-06-08_agent-sdk-core]] — SDK, subagent API, fork semantics
- [[sources/2026-06-08_extensions-ecosystem]] — subagent SKILL.md, AgentDefinition fields
