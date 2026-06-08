---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: experimental
sources_count: 1
tags: [domain/claude-code, theme/automation]
---

# Routines

## Description

Routines are saved Claude Code configurations (prompt + repositories + connectors) that execute automatically on Anthropic's cloud infrastructure. Unlike `/loop` (which requires an open session) or Desktop scheduled tasks (which require the machine to be on), Routines run even when your machine is off — making them the appropriate mechanism for reliable scheduled automation. Available on Pro, Max, Team, and Enterprise plans (requires Claude Code on the web to be enabled). Status: research preview.

## Status & Requirements

- **Status**: experimental (research preview)
- **Plans**: Pro, Max, Team, Enterprise (with Claude Code on the web enabled)
- **Platforms**: Cloud-executed; manage via `claude.ai/code/routines` or `/schedule` CLI

## Configuration

**Trigger types:**

**Scheduled** — cron-style recurring execution:
```bash
/schedule daily PR review at 9am
/schedule weekly dependency audit on Mondays
/schedule update "run every 6 hours"
```
- Minimum interval: 1 hour
- Custom cron via `/schedule update`
- One-shot runs: don't count toward daily cap

**API** — HTTP webhook trigger:
```http
POST https://api.anthropic.com/v1/claude_code/routines/{routine-id}/fire
Authorization: Bearer <token>
```
Response includes `claude_code_session_id` and session URL.

**GitHub** — event-driven trigger:
- Events: Pull request (opened, closed, assigned, labeled, synchronized) and Release
- Filters: Author, Title, Body, Base branch, Head branch, Labels, Is draft, Is merged
- Operators: equals, contains, starts with, is one of, not one of, matches regex

One routine can combine multiple trigger types.

**Management commands:**
```bash
/schedule daily PR review at 9am     # create
/schedule list                         # list all routines
/schedule update                       # modify existing
/schedule run                          # run immediately
```
Web UI: `claude.ai/code/routines`

**Environment:**
- Each execution clones the repository at the default branch
- Default network access: Trusted (common package registries, cloud APIs)
- Additional access: configure in environment settings

**Connectors:**
- All claude.ai account connectors included by default
- MCP servers added locally via CLI (`claude mcp add`) are NOT available in Routines
- Add MCP servers for routines at `claude.ai/customize/connectors` or declare in `.mcp.json` in the repo

**Branch permissions:**
- Default: Claude can only push to `claude/*` branches
- Enable unrestricted pushes: "Allow unrestricted branch pushes" setting

**Usage:**
- Counts against subscription usage (same as interactive sessions)
- Daily cap on executions per account; one-shot runs are exempt
- Can exceed cap with usage credits (billed)

## Use Cases

- **Nightly code review**: summarize the day's merged PRs, flag potential issues
- **Daily standups**: generate a status summary from open issues and recent commits
- **Weekly dependency audit**: check for outdated packages, security advisories
- **PR automation**: auto-label, auto-assign, auto-comment on new PRs matching filters
- **Release automation**: trigger post-release tasks when a GitHub Release is published
- **Monitoring alerts**: fire via API when an external alert system detects an issue

## Tradeoffs

- **vs `/loop`**: `/loop` needs an open session; Routines run when your machine is off. Use `/loop` for in-session interactive automation; Routines for reliable scheduled/triggered work.
- **vs Desktop scheduled tasks**: Desktop tasks require the machine to be on; Routines do not.
- **vs GitHub Actions**: GitHub Actions run on your infrastructure with full access to local tools; Routines run on Anthropic's cloud with access only to committed code and configured connectors. Routines are simpler to set up; Actions are more powerful for complex CI/CD.
- **Local MCP servers not available**: MCP servers configured locally are not accessible in Routines — only `.mcp.json` in the repo or `claude.ai/customize/connectors`. This is a significant constraint for teams that rely heavily on local MCP tools.
- **Research preview limitations**: feature may change; not suitable for mission-critical automation yet.

## Connections

- [[features/dynamic-workflows]] — in-session orchestration alternative for single-run tasks
- [[features/headless-mode]] — non-interactive alternative for local CI/scripting
- [[frameworks/ci-cd-integration]] — Routines vs GitHub Actions vs headless mode decision guide

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — trigger types, environment, connectors, branch permissions, usage limits
