# Index

<!-- Maintenu par le LLM. Une entrée par page. Format : - [[chemin/page]] — description en une ligne -->
<!-- Classé par catégorie, alphabétique dans chaque catégorie. -->

## Méta
- [[contexte]] — current domain state and active threads

## Concepts
- [[concepts/agentic-loop]] — the core gather→act→verify execution cycle, loop controls, SDK stream
- [[concepts/context-and-memory]] — context window composition, compaction, CLAUDE.md scoping, auto-memory

## Features
- [[features/agent-sdk]] — Python/TypeScript SDK exposing Claude Code's agentic loop programmatically
- [[features/agent-teams]] — experimental peer-to-peer multi-agent coordination (v2.1.32+, opt-in)
- [[features/dynamic-workflows]] — JS-orchestrated workflows scaling up to 1,000 agents (ultracode)
- [[features/extension-model]] — comparative overview of all 6 extension mechanisms (hooks, MCP, skills, plugins, subagents, CLAUDE.md)
- [[features/headless-mode]] — non-interactive `-p` mode for CI/scripting with structured output
- [[features/hooks]] — lifecycle event handlers for observability, access control, and automation
- [[features/mcp]] — Model Context Protocol: connecting Claude to external tools and services
- [[features/prompt-caching]] — 3-layer cache reducing token costs by ~90% on repeated context
- [[features/remote-control]] — control a local Claude session from claude.ai or mobile app
- [[features/routines]] — cloud-executed scheduled/triggered automation (research preview)
- [[features/sandboxing]] — OS-level filesystem and network isolation for Bash commands
- [[features/sessions]] — session storage, continuation, branching, and multi-host persistence
- [[features/skills]] — reusable SKILL.md workflows with progressive disclosure and description optimization

## Frameworks
- [[frameworks/agentic-patterns]] — decision framework: subagents vs. teams vs. workflows vs. SDK
- [[frameworks/permission-architecture]] — the 5-level allow/deny/ask permission system and 6 modes
- [[frameworks/settings-hierarchy]] — 5-scope settings resolution with merge rules and enterprise controls

## Recipes
- [[recipes/github-actions-setup]] — step-by-step GitHub Actions integration with the official action
- [[recipes/multi-agent-coordination]] — concrete patterns for subagents, teams, dynamic workflows, and SDK

## Sources
- [[sources/2026-06-08_agent-sdk-core]] — Agent SDK core: query(), sessions, hosting, subagents, system prompts
- [[sources/2026-06-08_skill-creator]] — Skill Creator: full skill dev loop (draft, eval, grade, compare, optimize description)
- [[sources/2026-06-08_agent-sdk-extensions]] — Agent SDK extensions: custom tools, MCP, hooks, permissions, plugins, skills
- [[sources/2026-06-08_claude-code-architecture]] — Claude Code architecture: loop, modes, context, teams, workflows, routines
- [[sources/2026-06-08_cli-surfaces-reference]] — CLI surfaces: commands, shortcuts, IDE integrations, Web, costs, computer use
- [[sources/2026-06-08_extensions-ecosystem]] — Extensions: hooks config, MCP add/types, plugins, skills, prompt library
- [[sources/2026-06-08_memory-settings-integrations]] — Settings, memory, caching, cloud integrations, GitHub Actions, channels
- [[sources/2026-06-08_permissions-security]] — Permissions, modes, sandbox, data policies, authentication
