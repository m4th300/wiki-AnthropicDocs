---
type: source
created: 2026-06-08
updated: 2026-06-08
status: published
author: Anthropic
date_source: 2026-06-08
raw_file: raw/overview.md, raw/how-claude-code-works.md, raw/goals.md, raw/features-overview.md, raw/auto-mode.md, raw/headless.md, raw/interactive-mode.md, raw/context-window.md, raw/glossary.md, raw/sub-agents.md, raw/agent-teams.md, raw/dynamic-workflows.md, raw/routines.md, raw/scheduled-tasks.md, raw/remote-control.md, raw/checkpointing.md, raw/common-workflows.md, raw/best-practices.md, raw/agent-view.md
tags: [domain/claude-code, theme/automation]
sources_count: 19
---

# Claude Code — Architecture & Agentic Patterns

## Résumé

Claude Code is an agentic coding tool (not a chat interface): the harness provides tools, permission enforcement, context management, and execution runtime; the model is Claude. The agentic loop runs three phases — gather context → act → verify — and can chain dozens of actions before returning control. Claude Code ships across CLI, VS Code, JetBrains, Desktop, and Web surfaces, all running the same engine. Key agentic patterns (subagents, agent teams, dynamic workflows, routines) address different scales of parallelism and autonomy.

## Points clés

**Agentic loop**
- Three phases: gather context (read files, search, fetch) → act (edit, run, commit) → verify (test, check output) → repeat
- Loop terminates when Claude judges work done, or user interrupts with `Esc`
- `/goal` (v2.1.139+): Haiku evaluates a user-defined completion condition after each turn; loop continues until satisfied
- Checkpoints created before each file edit; `Esc+Esc` or `/rewind` to restore (does NOT cover Bash)

**Operating modes**
- Interactive: full conversational terminal session; `Shift+Tab` cycles permission modes
- Non-interactive (`-p`/`--print`): executes and exits; output: `text`/`json`/`stream-json`; from 2026-06-15 draws from separate credit pool
- Bare (`--bare`): ignores hooks, skills, plugins, MCP, auto memory, CLAUDE.md; for reproducible CI
- Auto mode: background classifier examines every tool call; blocks by default: `curl | bash`, production deploys, bulk cloud deletions, IAM grants; falls back after 3 consecutive or 20 total blocks
- Plan mode: read-only; Claude proposes without modifying

**Context window composition** (approximate startup token counts)
- System prompt: ~4,200 | Auto memory: ~680 | Project CLAUDE.md: ~1,800 | Environment: ~280
- After compaction: CLAUDE.md and auto memory re-injected; path-scoped rules NOT re-injected (lost until matching file is read again)
- Subagent context economy: typically 6,100 tokens read by subagent → 420 tokens returned to parent

**Subagents**
- Built-in: Explore (codebase search), Plan (architecture), General-purpose
- Custom: `.claude/agents/<name>.md` (project) or `~/.claude/agents/<name>.md` (user); frontmatter: `name`, `description`, `tools`, `model`, `isolation`, `skills`, `effort`
- `isolation: worktree`: subagent gets its own git worktree, auto-deleted if no changes
- Fork subagent: inherits full parent context; useful for verification
- Background by default; foreground with `--foreground` or "run in the foreground"
- `Ctrl+X Ctrl+K` stops all background subagents

**Agent teams** (experimental, `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, v2.1.32+)
- Architecture: team leader + teammates + shared task list + mailbox (peer-to-peer messaging)
- Cost: ~7x tokens vs. standard session (each teammate = full Claude instance)
- Display modes: `in-process`, `tmux`, `auto`
- Optimal: 3-5 teammates, 5-6 tasks each, no overlapping file ownership
- NOT resumable (no `/resume` for in-process teammates)
- Limitations: one team at a time, no nested teams, requires tmux or iTerm2 for split panes

**Dynamic workflows** (ultracode)
- A JS script that orchestrates subagents at scale; Claude writes the script, runtime executes it
- Trigger: `ultracode:` keyword in prompt, `/effort ultracode`, or `/deep-research`
- Limits: 16 simultaneous agents, 1,000 total per run
- Results are cached on resume within the same session; exits reset to zero
- Save: `/workflows` → `s`; saved to `.claude/workflows/`; run as `/<name>`

**Routines** (cloud-executed automation)
- Run on Anthropic infrastructure even when machine is off
- Triggers: scheduled (min 1h interval), API (`POST /v1/claude_code/routines/{id}/fire`), GitHub events (PR/Release)
- Manage: `claude.ai/code/routines` or `/schedule` CLI commands
- Branch permissions: by default only `claude/*` branches; enable unrestricted for others
- Connectors: claude.ai connectors only; local MCP servers NOT available

**Scheduled tasks & `/loop`**
- `/loop 5m <prompt>`: runs prompt on interval while session is open
- `/loop <prompt>` (no interval): Claude self-paces based on observed state
- `/loop` (no args): runs built-in maintenance (continue work → handle MR → cleanup)
- Customize: `.claude/loop.md` or `~/.claude/loop.md`
- Max 50 tasks per session; 7-day auto-expiry; disable: `CLAUDE_CODE_DISABLE_CRON=1`

**Remote control**
- Connects claude.ai/code or Claude mobile to a local session (code always executes locally)
- Activation: `claude remote-control` (server mode), `claude --remote-control`, or `/rc`
- `--spawn same-dir|worktree|session` controls session isolation per remote connection
- Security: HTTPS outbound only, no open inbound ports

**Best practices (non-obvious)**
- Provide executable verification in every session (tests, build checks, screenshot comparison)
- Explore → Plan → Code → Validate workflow (4 phases)
- After 2 failed corrections: `/clear` + better prompt (not more corrections)
- CLAUDE.md: keep under 200 lines; if Claude ignores it, the file is too long
- Subagents for investigation (they explore in their own context, return only summary)
- `/btw <question>` for quick questions that shouldn't persist in context

## Liens wiki

- [[concepts/agentic-loop]] — the core loop described here in depth
- [[concepts/context-and-memory]] — context window composition and compaction
- [[features/subagents]] — subagent feature detail
- [[features/agent-teams]] — experimental agent teams
- [[features/dynamic-workflows]] — ultracode and workflow scripts
- [[features/routines]] — cloud-executed automation
- [[features/headless-mode]] — non-interactive and bare mode
- [[frameworks/agentic-patterns]] — when to use which multi-agent pattern

## Tensions

Aucune tension identifiée.

## Questions soulevées

- What's the performance ceiling for dynamic workflows at 1,000 agents? Are there practical latency/cost limits before that?
- How does the `/goal` evaluator (Haiku) handle ambiguous completion conditions — does it err toward continuing or stopping?
