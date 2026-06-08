---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 2
tags: [domain/claude-code, theme/automation]
---

# Headless Mode (Non-Interactive Mode)

## Description

Non-interactive mode (`-p` / `--print`) executes a Claude Code prompt and exits without opening an interactive terminal session. It enables Claude Code integration into CI/CD pipelines, shell scripts, and programmatic workflows. Bare mode (`--bare`) is the stricter variant that also disables all hooks, skills, plugins, MCP, auto memory, and CLAUDE.md — guaranteeing identical behavior across machines. From June 15, 2026, using `claude -p` on subscription plans draws from a separate monthly credit pool.

## Status & Requirements

- **Status**: stable
- **Platforms**: All (sandboxing not supported on Windows native; use WSL2 for sandbox)
- **Note**: "Headless mode" is the deprecated term; current terminology is "non-interactive mode"

## Configuration

**Basic usage:**
```bash
# Simple query
claude -p "Fix the failing tests in src/auth/"

# Continue last session
claude -c -p "Now add error handling"

# Resume specific session
claude -r "session-id" "continue the refactoring"

# Output format
claude -p "Summarize open issues" --output-format json
claude -p "Analyze the build" --output-format stream-json   # newline-delimited events
```

**Key flags for non-interactive use:**

| Flag | Description |
|---|---|
| `-p` / `--print` | Enable non-interactive mode |
| `--output-format text\|json\|stream-json` | Output format (default: text) |
| `--max-turns N` | Hard limit on tool-use turns |
| `--max-budget-usd N` | Spending cap in USD |
| `--permission-mode <mode>` | Starting permission mode |
| `--allowedTools "Read,Bash(npm test)"` | Pre-approve specific tools |
| `--bare` | Skip all extensions (hooks, skills, plugins, MCP, memory) |
| `--no-session-persistence` | Disable session JSONL writing |
| `--model <alias>` | Model for this invocation |

**Bare mode** — for reproducible CI:
```bash
claude -p "Review the PR changes" --bare --permission-mode dontAsk
```
- Ignores: hooks, skills, plugins, MCP, auto memory, CLAUDE.md
- Sets `CLAUDE_CODE_SIMPLE=1` in the subprocess environment
- Uses `ANTHROPIC_API_KEY` or `CLAUDE_CODE_OAUTH_TOKEN` (no OAuth from browser)
- Will become the default for `-p` in a future version

**Piping:**
```bash
git log --oneline -20 | claude -p "Summarize these commits"
tail -200 app.log | claude -p "Find anomalies"
git diff main --name-only | claude -p "Review these changes for security issues"
```
Pipe limit: 10 MB.

**Structured output:**
```bash
claude -p "Analyze this file" --json-schema '{"type":"object","properties":{"summary":{"type":"string"},"risk":{"type":"string","enum":["low","medium","high"]}}}'
```

**Stream-JSON events** (for programmatic processing):
```bash
claude -p "query" --output-format stream-json | while IFS= read -r line; do
    echo "$line" | python3 -c "import sys,json; m=json.load(sys.stdin); print(m.get('type','?'), m.get('content','')[:50] if m.get('content') else '')"
done
```

**Permission handling in CI:**
- `--permission-mode bypassPermissions`: skip all prompts (container/VM only)
- `--permission-mode dontAsk`: deny everything not in `--allowedTools`
- `--permission-prompt-tool <mcp-tool>`: delegate permission prompts to an MCP tool
- Pre-approve: `--allowedTools "Read,Bash(npm test),Bash(npm run lint)"`

**Authentication in non-interactive mode:**
- Interactive OAuth does NOT work in `--bare` mode
- Use `ANTHROPIC_API_KEY` or `CLAUDE_CODE_OAUTH_TOKEN` (`claude setup-token` generates one)
- `CLAUDE_CODE_OAUTH_TOKEN` is ignored in bare mode — use `ANTHROPIC_API_KEY`

## Use Cases

- **CI code review**: `git diff main...HEAD | claude -p "Review for security issues"`
- **Automated test repair**: `claude -p "Fix the failing test in $(npm test 2>&1 | grep FAIL | head -1)"`
- **PR automation**: trigger on PR open → `claude -p "Analyze and suggest fixes"`
- **Scheduled analysis**: nightly report generation, dependency audit
- **Shell script integration**: combine with standard Unix tools (grep, awk, xargs)

## Tradeoffs

- **`--bare` removes project context**: no CLAUDE.md means no project conventions — Claude may make decisions inconsistent with your codebase. Consider passing key instructions via `--append-system-prompt-file`.
- **Separate credit pool** (from 2026-06-15): subscription users who use `-p` heavily should monitor this separate pool.
- **No session interactivity**: cannot interrupt mid-execution, no checkpointing/rewind.
- **vs SDK**: for complex multi-turn automation, the Agent SDK gives more control (streaming, session management, hooks, callbacks). Use `-p` for simple scripting; SDK for production automation.

## Connections

- [[concepts/agentic-loop]] — the same loop runs in non-interactive mode, just without a human in the loop
- [[frameworks/permission-architecture]] — `dontAsk` and `bypassPermissions` modes are key for CI
- [[features/agent-sdk]] — the SDK alternative for production automation
- [[frameworks/ci-cd-integration]] — patterns for GitHub Actions and GitLab CI

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — headless, bare mode, output formats, piping
- [[sources/2026-06-08_cli-surfaces-reference]] — CLI flags, structured output, authentication in CI
