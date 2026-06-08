---
type: source
created: 2026-06-08
updated: 2026-06-08
status: published
author: Anthropic
date_source: 2026-06-08
raw_file: raw/cli-reference.md, raw/commands.md, raw/keyboard-shortcuts.md, raw/output-styles.md, raw/error-reference.md, raw/troubleshooting.md, raw/troubleshoot-install.md, raw/quickstart.md, raw/setup.md, raw/desktop-quickstart.md, raw/desktop.md, raw/vs-code.md, raw/jetbrains.md, raw/chrome.md, raw/claude-code-on-the-web.md, raw/devcontainer.md, raw/slack.md, raw/third-party-integrations.md, raw/analytics.md, raw/monitoring-usage.md, raw/costs.md, raw/fast-mode.md, raw/computer-use.md, raw/code-review.md, raw/monorepos.md, raw/worktrees.md, raw/security-guidance.md
tags: [domain/claude-code, theme/integrations]
sources_count: 27
---

# CLI, Surfaces & Use Cases Reference

## Résumé

This cluster covers the full CLI surface of Claude Code (100+ flags, 80+ slash commands), all IDE and surface integrations (VS Code, JetBrains, Desktop app, Chrome, Web), observability and cost management, and advanced use cases (computer use, automated PR review, monorepo support, git worktrees). Key operational facts: typical enterprise cost is $13/developer/active day; fast mode enables up to 2.5x faster Opus at higher per-token cost; automated PR code review costs $15-25/review; computer use is macOS-only research preview.

## Points clés

**Key CLI flags**
- `claude -p "query"`: non-interactive (print mode); exits after response; from 2026-06-15 draws from separate credit pool on subscriptions
- `claude --bare`: skip all hooks, skills, plugins, MCP, memory, CLAUDE.md (sets `CLAUDE_CODE_SIMPLE`)
- `claude --worktree <name>` / `-w`: isolated git worktree at `.claude/worktrees/<name>/`
- `claude --from-pr <number|URL>`: resume sessions linked to a PR
- `claude --bg "task"`: start as background agent, return immediately
- `claude --remote "task"`: create cloud session on claude.ai
- `claude --teleport`: pull a web session into local terminal
- `claude --permission-mode <mode>`: start in specific mode
- `claude --model <alias>`: session-only model override
- `claude --effort <level>`: session effort level
- `claude --output-format json|stream-json`: structured output for `-p` mode
- `claude --json-schema '<schema>'`: validated JSON output

**Key slash commands**
- `/compact [instructions]`: summarize conversation; optional focus instructions
- `/context`: live context usage breakdown
- `/goal <condition>`: set loop exit condition
- `/effort [level|ultracode]`: set effort; `ultracode` = xhigh + workflow orchestration
- `/deep-research <question>`: web research workflow
- `/batch <instruction>`: parallel changes across 5-30 independent units in own worktrees
- `/code-review [level] [--fix] [--comment]`: review current diff
- `/rewind`: restore to earlier checkpoint
- `/bg`: detach to background, open agent view
- `/dispatch "task"`: launch new background session
- `/autofix-pr [prompt]`: cloud session that monitors branch PR and auto-pushes fixes
- `/ultraplan <prompt>`: plan in ultraplan session, execute remotely
- `/ultrareview [PR]`: deep multi-agent cloud code review

**VS Code extension**
- Min version: VS Code 1.98.0+; also works in Cursor, Kiro, Devin Desktop
- Toggle: Spark icon, Activity Bar, Cmd+Shift+P, or status bar `✱ Claude Code`
- `Option+K`/`Alt+K`: insert `@`-mention with path + line numbers
- Embedded MCP server: `mcp__ide__getDiagnostics`, `mcp__ide__executeCode`
- Runs on random port 127.0.0.1 only; random auth token per session
- Key setting: `claudeCode.useTerminal = true` for terminal mode

**Desktop app** (macOS + Windows)
- Three tabs: Chat (general, no file access), Cowork (cloud VM), Code (local file access)
- Environments: Local (your machine), Remote (Anthropic cloud), SSH (remote machine)
- Key features: sessions sidebar with parallel sessions, diff view, PR tracking with auto-fix/merge, scheduled tasks, side chat, computer use
- CLI vs Desktop: Desktop adds GUI for plugins, scheduled tasks, PR tracking, graphical diff; CLI has full commands

**Claude Code on the Web** (research preview)
- VM: 4 vCPU, 16 GB RAM, 30 GB disk, Ubuntu 24.04
- Not available: user-level `~/.claude/CLAUDE.md`, locally-added MCP servers, interactive auth
- GitHub auth: either GitHub App (onboarding) or `/web-setup` (sync local `gh` token)
- Move to terminal: `claude --teleport` (requires clean git state, branch pushed to remote)
- Setup scripts: run at session start as root; cached ~7 days; keep under 5 minutes
- Network levels: None / Trusted / Full / Custom

**Cost management**
- Enterprise average: ~$13/developer/active day; $150-250/developer/month
- Agent teams: ~7x tokens vs. standard session
- Rate limits scale inversely with team size (200-300K TPM for 1-5 users; 10-15K TPM for 500+)
- Context management: `/clear` between unrelated tasks; `/compact <focus>` with instructions; skills > CLAUDE.md for specialized content
- Extended thinking: can cost 10K+ thinking tokens per request; lower with `/effort low` or `MAX_THINKING_TOKENS=8000`

**Fast mode** (research preview, v2.1.36+)
- Up to 2.5x faster Opus at higher per-token cost (same quality)
- Opus 4.8: $10/$50 MTok (input/output); Opus 4.7/4.6: $30/$150 MTok
- Billed via usage credits (not plan); not on Bedrock/Vertex/Foundry
- Cache invalidation warning: activating mid-session invalidates existing cache; activate at session start
- Toggle: `/fast` or `"fastMode": true` in settings.json

**Computer use** (research preview, v2.1.85+, macOS only, Pro/Max only)
- Enable: `/mcp` → computer-use; grant Accessibility + Screen Recording permissions
- Machine-wide lock: one session at a time; other apps hidden during operation
- Per-app approval required each session
- Claude's tool preference: MCP server > Bash > Chrome > computer use (last resort)

**Automated code review** (Team/Enterprise, ~$15-25/review)
- Setup: admin installs Claude GitHub App, selects repos, sets behavior (once / after every push / manual)
- Manual trigger: `@claude review` in PR comment (top-level only)
- Severity: 🔴 Important / 🟡 Nit / 🟣 Pre-existing
- `REVIEW.md` at repo root: highest-priority instructions for review agents
- Neutral conclusion: never blocks merge

**Monorepos**
- `claudeMdExcludes`: skip specific CLAUDE.md files
- `permissions.additionalDirectories`: grant access to directories outside cwd
- `worktree.sparsePaths`: write only listed directories to disk for large monorepos
- Per-directory skills loaded on demand

**OpenTelemetry monitoring**
- `CLAUDE_CODE_ENABLE_TELEMETRY=1` + OTEL env vars
- Metrics: `claude_code.session.count`, `claude_code.cost.usage`, `claude_code.token.usage`, `claude_code.lines_of_code.count`, `claude_code.pull_request.count`
- Events: `claude_code.user_prompt`, `claude_code.tool_decision`, `claude_code.permission_mode_changed`, etc.
- Content logging: `OTEL_LOG_USER_PROMPTS=1`, `OTEL_LOG_TOOL_DETAILS=1`

## Liens wiki

- [[features/headless-mode]] — non-interactive mode details
- [[features/computer-use]] — computer use feature
- [[features/remote-control]] — Remote Control and cloud sessions
- [[frameworks/ci-cd-integration]] — GitHub Actions, GitLab, headless patterns

## Tensions

Aucune tension identifiée.

## Questions soulevées

- How does `/autofix-pr` interact with branch protection rules and required reviewers?
- Can `computer-use` be used in headless/non-interactive mode for automated UI testing pipelines?
