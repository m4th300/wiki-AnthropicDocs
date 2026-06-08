---
type: feature
created: 2026-06-08
updated: 2026-06-08
status: current
status_feature: stable
sources_count: 3
tags: [domain/claude-code, theme/extensions]
---

# Skills

## Description

Skills are reusable Claude Code workflows stored as `SKILL.md` files. A skill bundles instructions, optional scripts, and reference files into a single loadable unit. Claude invokes skills automatically when the description matches the task, or explicitly via `/skill-name`. Unlike CLAUDE.md (always loaded), a skill's body only loads on activation — making them context-efficient for large playbooks.

Note: legacy `.claude/commands/` files continue to work and are merged into skills. A `deploy.md` in `commands/` and a `SKILL.md` in `skills/` both create a `/deploy` command.

## Status & Requirements

- **Status**: stable
- **Platforms**: All

## Configuration

### Structure

```
.claude/skills/my-skill/
├── SKILL.md             # required
├── scripts/             # executable helpers (run without loading)
├── references/          # docs loaded into context as needed
└── assets/              # templates, icons, fonts used in output
```

### SKILL.md frontmatter

```markdown
---
name: my-skill                    # required — identifier and slash command name
description: "When to invoke..."  # required — primary triggering mechanism
tools: [Read, Bash, Edit]         # optional — inherits parent tools if omitted
disable-model-invocation: false   # optional — true = manual slash command only
trigger: "deploy"                 # optional — additional trigger keywords
---
```

**Key frontmatter fields:**

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Identifier used for `/name` invocation and `skills` SDK option |
| `description` | **Yes — critical** | Claude reads this to decide when to invoke. Include both WHAT and WHEN. |
| `tools` | No | List of allowed tools; inherits parent tools if omitted |
| `disable-model-invocation` | No | `true` = only invocable via `/skill-name`, never auto |
| `trigger` | No | Additional keywords that trigger the skill |
| `subagent` | No | `true` = skill runs in its own context |
| `model` | No | Model override for this skill |

**Description writing principle:** Claude tends to "undertrigger" — be explicit and "pushy". Don't just say what the skill does; say *when* to use it, including paraphrases and edge cases.

### Progressive disclosure (3-level loading system)

| Level | What loads | Size guidance |
|---|---|---|
| Metadata | `name` + `description` | Always in context (~100 words) |
| SKILL.md body | Full instructions | On activation (target <500 lines) |
| Bundled resources | `scripts/`, `references/`, `assets/` | On demand (scripts execute without loading) |

If SKILL.md approaches 500 lines, add a layer of hierarchy with pointers to reference files. Large reference files (>300 lines) should include a table of contents.

### Emplacements / scopes

| Location | Scope |
|---|---|
| `~/.claude/skills/` | All projects (user) |
| `.claude/skills/` | Project only (shareable via git) |
| `~/.claude/commands/` | All projects (legacy, still works) |
| `.claude/commands/` | Project (legacy, still works) |

### Bundled skills (shipped with Claude Code)

| Skill | Slash command | Description |
|---|---|---|
| batch | `/batch` | Splits a large change into parallel subagents |
| code-review | `/code-review`, `/review` | Code review workflow |
| debug | `/debug` | Debugging assistance |
| loop | `/loop` | Repeats a prompt at intervals |
| plan | `/plan` | Planning and architecture |

Full list: see `wiki/sources/2026-06-08_cli-surfaces-reference`.

### Invocation control

**Via settings.json:**
```json
{
  "skillOverrides": {
    "my-skill": {
      "disableModelInvocation": true
    }
  }
}
```

### Subagents in skills

```markdown
---
subagent: true
model: claude-sonnet-4-6
tools: [Read, Bash]
---
# Research Agent
Explore the codebase and report findings...
```

### SDK usage

```python
options = ClaudeAgentOptions(
    cwd="/path/to/project",
    setting_sources=["user", "project"],  # load skills from filesystem
    skills="all",                          # enable all discovered skills
    allowed_tools=["Read", "Write", "Bash"],
)
```

**`skills` option values:**
- `"all"` — enable all discovered skills
- `["pdf", "docx"]` — enable only these skills (by name or directory)
- `[]` — disable all skills
- Omitted — CLI default behavior (discovered skills enabled)

**Important SDK constraint**: `allowed-tools` in `SKILL.md` frontmatter is **ignored** in the SDK. Tool control is via `allowedTools` in the query options — not in the skill file [[sources/2026-06-08_agent-sdk-extensions]].

`skills` is a context filter, not a sandbox: unlisted skills are hidden from the model but their files remain accessible via `Read`/`Bash`.

### Troubleshooting skill discovery

```bash
# Verify files exist
ls .claude/skills/*/SKILL.md
ls ~/.claude/skills/*/SKILL.md

# Check setting_sources includes "user" and/or "project"
# Check cwd points to the correct directory or a repo ancestor
```

## Use Cases

- **Repeatable playbooks**: workflows you paste into chat regularly → package as a skill
- **Team-shared procedures**: deploy, review, migration workflows committed to `.claude/skills/`
- **Multi-step procedures**: a CLAUDE.md section that grew into a multi-step process
- **Context-efficient specialization**: skill body loads only when needed (vs always-loaded CLAUDE.md)
- **Bundled tooling**: scripts/ folder pre-packages helper scripts that all future invocations can use without re-writing them

## Tradeoffs

**vs CLAUDE.md**: CLAUDE.md loads every turn (always in context); skills load only on activation. Use CLAUDE.md for invariants; use skills for procedures.

**vs Hooks**: hooks trigger on lifecycle events (pre/post tool); skills trigger on task description match. Hooks are reactive; skills are instructional.

**vs Subagents**: subagents run in their own context window; skills run in the parent context. Use subagents for isolated delegation; skills for adding behavior to the current session.

**Auto-trigger reliability**: Claude may "undertrigger" — fail to invoke a skill when it would be useful. Mitigation: write "pushy" descriptions that name edge cases and alternative phrasings. Use `disable-model-invocation: false` and test with `/skill-name` to verify the skill works before relying on auto-trigger.

## Connections

- [[features/extension-model]] — skills in the full extension mechanism comparison
- [[features/subagents]] — subagents vs. skills for delegating work
- [[features/hooks]] — hooks vs. skills for adding behavior
- [[concepts/context-and-memory]] — skills and CLAUDE.md loading into context
- [[features/agent-sdk]] — `skills` and `setting_sources` SDK options

## Tensions

Aucune tension identifiée.

## Sources

- [[sources/2026-06-08_extensions-ecosystem]] — skills structure, frontmatter, bundled list, subagents in skills
- [[sources/2026-06-08_agent-sdk-extensions]] — SDK `skills` option, `allowed-tools` ignored constraint, setting_sources
- [[sources/2026-06-08_skill-creator]] — progressive disclosure anatomy, description writing principles, skill creation workflow
