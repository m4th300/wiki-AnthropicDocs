---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 2
tags: [domain/claude-code]
---

# Sessions

## Description

A session is a recorded conversation tied to a project directory, stored as a JSONL transcript on local disk. Sessions enable conversation continuity — resuming work after a break, branching to explore an alternative approach, or forking to a reviewer. They capture conversation history, not filesystem state (for filesystem snapshots, use checkpointing).

## Status & Requirements

- **Status**: stable
- **Platforms**: All

## Configuration

**Storage location:**
```
~/.claude/projects/<encoded-cwd>/<session-id>.jsonl
```
`<encoded-cwd>` is derived from the working directory path. Override with `CLAUDE_CONFIG_DIR`.

**Session commands:**

| Command | Effect |
|---|---|
| `claude --continue` / `-c` | Resume last session in current directory |
| `claude --resume` | Open interactive session selector |
| `claude --resume <name>` | Resume by name directly |
| `claude --from-pr <number\|URL>` | Resume session linked to a PR |
| `/resume` | Switch sessions from within an active session |
| `/branch <name>` or `claude -c --fork-session` | Fork current session |
| `/rename <name>` | Name current session |

**Retention:** 30 days by default; configure with `cleanupPeriodDays` in settings.json.

**Disable transcript writing:**
- All sessions: `CLAUDE_CODE_SKIP_PROMPT_HISTORY=1`
- Non-interactive only: `claude -p --no-session-persistence`

**Branch/fork semantics:**
- Fork creates a copy — changes in the fork don't affect the original
- "allow for this session" permissions are NOT carried over to the fork
- Use for: exploring alternative implementations, what-if scenarios

**Sessions in the Agent SDK:**

| Strategy | Python | TypeScript | Use case |
|---|---|---|---|
| Auto (last session) | `continue_conversation=True` | `continue: true` | Multi-turn in same process |
| By ID | `resume=session_id` | `resume: sessionId` | Resume specific past session |
| Fork | `fork_session=True` + `resume` | `forkSession()` | Explore alternative without losing original |
| Stateless | N/A | `persistSession: false` | Pure stateless, nothing on disk |

**SessionStore** (SDK — external storage for multi-host deployments):
- Interface: `append(key, entries)` (required), `load(key)` (required), `listSessions()` (optional)
- Built-in: `InMemorySessionStore` (testing/dev)
- Reference adapters: `S3SessionStore`, `RedisSessionStore`, `PostgresSessionStore` (in SDK repo)
- Architecture: double-write — subprocess writes to local disk first, SDK mirrors to store
- Compatible with: `query()`, `startup()`, `listSessions()`, `getSessionMessages()`, `renameSession()`, `tagSession()`, `deleteSession()`, `forkSession()`
- Incompatible with: `persistSession: false` and `enableFileCheckpointing`

**Capturing session ID:**
```python
async for message in query(prompt="Fix the bug", ...):
    if isinstance(message, ResultMessage):
        session_id = message.session_id  # save this for later resume
```

## Use Cases

- **Multi-day work**: resume exactly where you left off, including conversation context
- **PR-linked sessions**: `claude --from-pr 1234` — resume the session that worked on that PR
- **Parallel exploration**: fork session → try approach A while original pursues approach B
- **Writer/Reviewer pattern**: Session A (Writer) → export context → Session B (Reviewer with fresh context) → Session A (apply corrections)
- **Multi-host automation**: `SessionStore` enables cloud agents to resume sessions across container restarts

## Tradeoffs

- **Sessions store conversation, not filesystem state**: if you rewind a session, files are NOT reverted (use `/rewind` / checkpointing for that)
- **Named sessions vs IDs**: names are display-only; session IDs are UUIDs. `--resume <name>` searches by name, not ID.
- **`SessionStore` double-write overhead**: adds latency proportional to storage backend; `mirror_error` events on failure don't abort the session but may lose remote copies

## Connections

- [[concepts/context-and-memory]] — sessions are the persistence layer for context
- [[features/agent-sdk]] — `SessionStore` interface for multi-host deployments

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_memory-settings-integrations]] — storage path, retention, branching
- [[sources/2026-06-08_agent-sdk-core]] — `SessionStore`, fork, continue, stateless sessions
