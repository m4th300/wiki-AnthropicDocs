---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 1
tags: [domain/claude-code, theme/security]
---

# Sandboxing

## Description

Claude Code's built-in sandbox provides OS-level filesystem and network isolation for Bash subprocess commands. It uses Seatbelt on macOS and bubblewrap on Linux/WSL2 to confine what shell commands can read, write, and connect to — without wrapping the entire Claude Code process. Critically: the sandbox covers only Bash commands. Built-in tools (Read, Edit, Write), MCP servers, and hooks run on the host and are not sandboxed.

## Status & Requirements

- **Status**: stable (built-in sandbox); `@anthropic-ai/sandbox-runtime` is research beta
- **Platforms**: macOS, Linux, WSL2 only. Windows native: NOT supported.
- **Linux/WSL2 dependencies**: `sudo apt-get install bubblewrap socat`

## Configuration

**Enable in-session:** `/sandbox`

**Full configuration schema** (`settings.json`):
```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": ["~/.kube", "/tmp/build"],
      "denyRead":   ["~/"],
      "allowRead":  ["."],
      "denyWrite":  ["~/Documents"]
    },
    "network": {
      "allowedDomains": ["api.github.com", "*.npmjs.org"],
      "deniedDomains": ["example.com"]
    }
  }
}
```

**Path prefix syntax** (different from permission rule syntax):

| Prefix | Meaning |
|---|---|
| `/` | Absolute from filesystem root |
| `~/` | From home directory |
| `./` or no prefix | Relative to project root |

**Defaults:**
- Write: only in current working directory
- Read: entire computer (except some denied directories)
- Network: no domains pre-approved; first connection to new domain prompts user

**Custom proxy ports:**
```json
{
  "sandbox": {
    "network": { "httpProxyPort": 8080, "socksProxyPort": 8081 }
  }
}
```

**Organization-managed keys** (in managed settings):
- `failIfUnavailable: true` → hard error if sandbox unavailable on machine
- `allowUnsandboxedCommands: false` → disable "retry without sandbox" option
- `excludedCommands: ["docker *", "terraform *"]` → commands that skip the sandbox
- `allowManagedReadPathsOnly: true` → only managed `allowRead` entries honored
- `allowManagedDomainsOnly: true` → same for network domains

**Sandbox runtime** (`@anthropic-ai/sandbox-runtime`, research beta): wraps the entire Claude Code process, not just Bash.
```bash
npx @anthropic-ai/sandbox-runtime claude
```
Config: `~/.srt-settings.json` with `writePaths` and `allowedDomains`.

**Sandbox environment comparison:**

| Approach | What is isolated | Docker required |
|---|---|---|
| Built-in Bash sandbox | Bash commands only | No |
| Sandbox runtime | Entire Claude Code process | No |
| Dev container | Full dev environment | Yes |
| VM | Full OS | No |
| Claude Code on the web | Full OS, hosted by Anthropic | No |

## Use Cases

- **Restricting network access**: prevent Claude's shell commands from calling arbitrary external services
- **Read protection**: deny reads of `~/.aws`, `~/.ssh` in sandboxed sessions
- **Write containment**: ensure shell commands write only to the project directory
- **Enterprise compliance**: `failIfUnavailable: true` + `allowManagedDomainsOnly: true` for strict org policy

## Tradeoffs

- **Bash-only coverage**: Read, Write, Edit, and MCP server tool calls are NOT sandboxed. Attacker-controlled MCP servers or hooks run on the host regardless.
- **TLS not terminated at proxy**: HTTPS content is not inspected; domain fronting is possible on broad domains
- **Unix sockets**: allowing `/var/run/docker.sock` grants full host access
- **Environment variables inherited**: credentials in env vars pass to subprocesses. Scrub with `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB`.
- **No Windows support**: native Windows users must use WSL2 for sandbox functionality

## Connections

- [[frameworks/permission-architecture]] — permission rules are a complementary control layer
- [[features/headless-mode]] — `--dangerously-skip-permissions` + container is the recommended CI pattern

## Tensions

Aucune tension identifiée. Note the important scope limitation: sandbox ≠ full Claude Code isolation. For complete isolation, use a container/VM or `@anthropic-ai/sandbox-runtime`.

## Sources

- [[sources/2026-06-08_permissions-security]] — sandbox config, default behaviors, organization controls, security limitations
