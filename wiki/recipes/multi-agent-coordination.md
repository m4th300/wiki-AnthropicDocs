---
type: recipe
created: 2026-06-08
updated: 2026-06-08
status: current
sources_count: 3
tags: [domain/claude-code, theme/automation]
---

# Multi-Agent Coordination

## Description

Concrete patterns for orchestrating multiple Claude Code agents. Covers the four major mechanisms (subagents, agent teams, dynamic workflows, Agent SDK) with actual implementation patterns and anti-patterns. Each pattern includes a cost estimate, complexity rating, and resumability note.

## Pattern 1 — Subagent Fan-Out (Recommended starting point)

**Use when:** 3-10 parallel, independent sub-tasks.

**Natural language invocation:**
```
"Create subagents to: (1) investigate the auth module for security issues, 
(2) review the payment module for compliance gaps, 
(3) audit the logging system for PII leaks. 
Each agent should work independently and report back."
```

**SKILL.md for reusable subagents** (`wiki/meta/reviewer.md`):
```markdown
---
agent: security-reviewer
description: Reviews code for security vulnerabilities. Outputs a structured JSON report.
model: claude-sonnet-4-6
allowed-tools: [Read, Glob, Grep, WebSearch]
---
# Security Reviewer
Focus on: OWASP Top 10, injection vectors, auth bypass, PII exposure.
Output format: {"severity": "high|medium|low", "findings": [...]}
```

**Agent SDK — parallel fan-out:**
```python
import asyncio
from claude_code_sdk import query, ClaudeAgentOptions

async def parallel_review(modules: list[str]):
    tasks = [
        query(
            prompt=f"Security review of {module}",
            options=ClaudeAgentOptions(
                allowed_tools=["Read", "Glob", "Grep"],
                max_turns=10
            )
        )
        for module in modules
    ]
    results = await asyncio.gather(*[collect(t) for t in tasks])
    return results

async def collect(stream):
    async for msg in stream:
        if hasattr(msg, 'result'):
            return msg.result
```

**Cost:** ~1 full session per subagent, no overhead. 5 agents = ~5x base cost.

**Resumability:** None — if parent session ends, work is lost.

---

## Pattern 2 — Writer/Reviewer (Context isolation)

**Use when:** Need fresh perspective for review; reviewer should not be influenced by the writer's reasoning.

**Steps:**
1. Session A (Writer): implement the feature → `/rename writer-session`
2. Fork: `claude -c --fork-session` → creates Session B with same context
3. Session B (Reviewer): "Review the implementation in Session A fresh. Do not look at your own previous reasoning."
4. Session A: apply corrections from reviewer's output

**Why fork instead of new session:** Reviewer has the codebase context but not the writer's justification chain.

**Agent SDK variant:**
```python
# Writer
async for msg in query("Implement the auth module", options=opts):
    if isinstance(msg, ResultMessage):
        writer_session_id = msg.session_id

# Reviewer — fork from writer, different prompt
async for msg in query(
    "Review the auth module implementation for security issues",
    options=ClaudeAgentOptions(resume=writer_session_id, fork_session=True)
):
    pass
```

---

## Pattern 3 — Agent Team (Peer coordination)

**Use when:** 3-5 parallel agents that need to communicate with each other (not just report to parent).

**Activation required:**
```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

**Natural language invocation:**
```
"Create a team of 3 agents to refactor the payment module:
- Agent 1: database layer (models/, migrations/)
- Agent 2: API layer (routes/, controllers/)  
- Agent 3: test layer (tests/payment/)
Each agent owns their files. Coordinate via the shared task list."
```

**Key rules for teams:**
- Give each agent **exclusive file ownership** — no two agents write to the same file
- Set 5-6 tasks per agent max
- Use `Shift+Down` to navigate between agents in the terminal
- `Ctrl+X Ctrl+K` to stop all agents

**Cost:** ~7x a single session total. Use Sonnet for teammates to reduce cost.

---

## Pattern 4 — Dynamic Workflow (Large scale)

**Use when:** More than 10 parallel agents, or need scripting logic.

**Trigger:**
```
"ultracode: Run a comprehensive security audit across all 47 microservices. 
For each service, check OWASP Top 10, generate a severity-ranked report, 
and create a GitHub issue for any high-severity finding."
```

**Save for reuse:** `/save security-audit-workflow` — saves the orchestration script.

**Resume:** `claude --resume security-audit-workflow` + `/effort ultracode`

**Limits:** 16 concurrent agents, 1,000 total per workflow.

---

## Pattern 5 — SDK Multi-Host (Production deployment)

**Use when:** Embedding in a production service; sessions must persist across container restarts.

```python
from claude_code_sdk import query, ClaudeAgentOptions
from claude_code_sdk.session_stores import S3SessionStore

store = S3SessionStore(bucket="my-sessions", prefix="prod/")

async def handle_request(session_id: str | None, prompt: str):
    opts = ClaudeAgentOptions(
        resume=session_id,      # None = new session
        session_store=store,
        max_turns=20,
        allowed_tools=["Read", "Edit", "Bash"]
    )
    async for msg in query(prompt=prompt, options=opts):
        if isinstance(msg, ResultMessage):
            return {
                "result": msg.result,
                "session_id": msg.session_id  # return to client for next turn
            }
```

---

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| 20 sequential subagents | Slow, blocks parent | Fan-out in parallel |
| Agent team for one delegatable task | 7x unnecessary cost | Use one subagent |
| Subagents with shared file writes | Conflicts | Give each agent exclusive ownership |
| Large workflow without saving | Can't resume | `/save workflow-name` before starting |
| SDK agent spawning nested agents | Not supported | Use dynamic workflows |
| Agent team without per-agent file ownership | Merge conflicts | Define file territory before spawning |

## Decision Checklist

```
1. How many parallel sub-tasks?
   1: just do it inline
   2-10: subagents
   >10: dynamic workflow

2. Do agents need to talk to each other?
   No: subagents
   Yes: agent team (experimental) or dynamic workflow with message file

3. Does it need to run without a session?
   Yes: Routines or GitHub Actions
   No: above patterns apply

4. Is this embedded in production code?
   Yes: Agent SDK with SessionStore
   No: interactive patterns above
```

## Connections

- [[frameworks/agentic-patterns]] — full decision framework and comparison table
- [[features/subagents]] — subagent definition and inheritance
- [[features/agent-teams]] — teams activation and architecture
- [[features/dynamic-workflows]] — ultracode, save, resume
- [[features/agent-sdk]] — SDK API, SessionStore, hosting

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — agent teams, dynamic workflows, limits
- [[sources/2026-06-08_agent-sdk-core]] — SDK query, fork, SessionStore patterns
- [[sources/2026-06-08_extensions-ecosystem]] — subagent SKILL.md, AgentDefinition fields
