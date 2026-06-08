# Log

<!-- Prepend : entrée la plus récente en haut. -->
<!-- Format : ## [YYYY-MM-DD] action | Titre -->
<!-- Actions valides : ingest, query, lint, update, init, schema -->

## [2026-06-08] ingest | Skill Creator + skills documentation
Mode : fast-track
Sources lues : raw/skill-creator/SKILL.md, raw/skill-creator/agents/grader.md, raw/skill-creator/agents/analyzer.md, raw/skill-creator/agents/comparator.md, raw/skill-creator/references/schemas.md, raw/skills.md, raw/agent-sdk-skills.md
Pages sources créées : sources/2026-06-08_skill-creator
Pages features créées : features/skills
Tensions détectées : aucune

## [2026-06-08] ingest | Fast-track ingest — all 101 raw files
Mode : fast-track
Sources lues : 101 fichiers `raw/*.md` (documentation officielle Claude Code)
Pages sources créées : sources/2026-06-08_agent-sdk-core, sources/2026-06-08_agent-sdk-extensions, sources/2026-06-08_claude-code-architecture, sources/2026-06-08_permissions-security, sources/2026-06-08_extensions-ecosystem, sources/2026-06-08_memory-settings-integrations, sources/2026-06-08_cli-surfaces-reference
Pages concepts mises à jour : concepts/agentic-loop, concepts/context-and-memory
Pages features mises à jour : features/agent-sdk, features/extension-model
Pages features créées : features/agent-teams, features/dynamic-workflows, features/headless-mode, features/hooks, features/mcp, features/prompt-caching, features/remote-control, features/routines, features/sandboxing, features/sessions, features/subagents
Pages frameworks mises à jour : frameworks/permission-architecture
Pages frameworks créées : frameworks/agentic-patterns, frameworks/settings-hierarchy
Pages recipes créées : recipes/github-actions-setup, recipes/multi-agent-coordination
Tensions détectées : aucune

## [2026-06-08] schema | Language convention + new page types
Added to CLAUDE.md: English language rule, `feature` and `recipe` page types with their folders (`wiki/features/`, `wiki/recipes/`), tags taxonomy. Removed `personne` type. Updated architecture diagram.

## [2026-06-08] init | Initialization — Claude Code research wiki
Langue : English
Types de pages : concept, feature, framework, recipe, source
Pages squelette créées : concepts/agentic-loop, concepts/context-and-memory, features/extension-model, features/agent-sdk, frameworks/permission-architecture
Templates créés : wiki/meta/feature.md, wiki/meta/recipe.md
Adaptations CLAUDE.md : oui — langue, types, taxonomie de tags, architecture
