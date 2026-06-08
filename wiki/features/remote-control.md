---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 1
tags: [domain/claude-code, theme/integrations]
---

# Remote Control

## Description

Remote Control connects claude.ai/code or the Claude mobile app to a Claude Code session running on your local machine. Claude always executes locally — your filesystem, MCP servers, and credentials stay on your machine — while you interact with the session from anywhere. The connection uses HTTPS outbound only, with no open inbound ports.

## Status & Requirements

- **Status**: stable
- **Plans**: Pro, Max, Team, Enterprise (not API-key only)
- **Min version**: v2.1.51+
- **Team/Enterprise**: admin must enable Remote Control in admin console
- **Not compatible with**: Ultraplan

## Configuration

**Activation methods:**

```bash
# Server mode (recommended for multiple sessions, stays running)
claude remote-control --name "my-server" --capacity 32

# Interactive session with remote access
claude --remote-control
# or
claude --rc

# From inside an existing session
/remote-control
# or
/rc
```

**Server mode options:**

| Flag | Description |
|---|---|
| `--name <name>` | Display name for this server |
| `--spawn same-dir\|worktree\|session` | How to create sessions per connection |
| `--capacity N` | Max connections (default: 32) |
| `--verbose` | Enable verbose logging |
| `--sandbox` | Enable sandbox for each spawned session |

**Connecting from remote devices:**
- Open the session URL in a browser
- Scan the QR code with the Claude mobile app
- Find the session by name on `claude.ai/code`

**Session naming priority:**
1. `--name` flag or `/remote-control` argument
2. `/rename` command
3. Last significant message in history
4. Auto-generated name (e.g. `myhost-graceful-unicorn`)

**Enable for all sessions** (two ways):
```bash
/config → "Enable Remote Control for all sessions"
# or settings.json
```

**Push notifications** (v2.1.110+):
- Requires Claude mobile app, same account, notifications allowed
- Enable: `/config` → "Push when Claude decides"
- Claude decides when to notify (on completion, on permission prompt, etc.)

**Available commands from remote:**
`/compact`, `/clear`, `/context`, `/usage`, `/exit`, `/recap`, `/reload-plugins`

**Not available from remote** (require interactive selectors):
`/mcp`, `/plugin`, `/resume`

## Remote Control vs. Claude Code on the Web

| | Remote Control | Claude Code on the Web |
|---|---|---|
| Claude runs on | Your machine | Anthropic VM |
| Local MCP access | Yes | No |
| Local file access | Yes | No |
| Continues if machine off | No | Yes |

## Use Cases

- **Mobile supervision**: start a long task on your laptop, monitor progress from your phone
- **Review from tablet**: check Claude's work from a device without your dev environment
- **Multi-device workflow**: hand off between devices without interrupting the session
- **Remote server management**: `claude remote-control` on a server, connect from anywhere

## Tradeoffs

- **Machine dependency**: if your machine goes offline or sleeps, the session ends. For tasks that must survive machine shutdown, use Routines (cloud-executed) instead.
- **Network outage tolerance**: session expires after ~10 minutes of network outage.
- **One remote session per interactive process** (outside server mode): starting remote control in a terminal Claude Code session means one connection at a time.
- **Security**: the connection is HTTPS outbound through Anthropic's API — all traffic flows through Anthropic's infrastructure. Short-lived credentials per connection, limited to a single purpose.

## Connections

- [[features/routines]] — cloud alternative when machine needs to be off
- [[features/headless-mode]] — background execution without UI

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_claude-code-architecture]] — full remote control description, server mode, comparison with cloud
