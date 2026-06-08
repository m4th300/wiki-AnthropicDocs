# Context

*Updated via `/update` at the end of significant sessions.*

## Domain

A research environment specialized in Claude Code and its ecosystem — Claude API, Agent SDK, integrations, and agentic patterns. Covers how Claude Code works internally, all extension mechanisms, and concrete implementation patterns, with the goal of understanding, implementing, and continuously improving Claude usage across all surfaces.

## Current situation

First ingest complete. All 101 raw documentation files from the official Claude Code documentation have been compiled into the wiki. The wiki now has substantive coverage of the entire Claude Code surface area:

- **7 source pages** covering ~101 raw files grouped by topic cluster
- **2 concept pages** (agentic loop, context & memory) — fully filled
- **13 feature pages** covering the major Claude Code features
- **3 framework pages** (permission architecture, settings hierarchy, agentic patterns)
- **2 recipe pages** (GitHub Actions setup, multi-agent coordination)

The wiki is now ready for query mode and targeted ingest of new sources as the documentation evolves.

## Active projects

*(to be completed as sessions progress)*

## Active threads

- **Agent SDK**: new Python/TypeScript SDK exposing Claude Code programmatically. Key areas to watch: `SessionStore` adapters (S3, Redis, PostgreSQL), file checkpointing evolution, permission modes in hosted environments.
- **Experimental features**: Agent Teams, Dynamic Workflows (ultracode), Routines — all experimental/research preview. Follow release notes for stability changes.
- **MCP ecosystem**: managed MCP for enterprise, deferred tool schemas (tool search). Watch for OAuth improvements and STDIO-in-cloud limitations resolution.

## Open questions

- What is the exact token budget breakdown for the context window in practice? (System prompt size varies by tool count — no exact figure in docs.)
- Are there undocumented limits on subagent nesting beyond "no nested subagents"?
- How does the Haiku evaluator model in the agentic loop interact with cost calculation — is it always Haiku or configurable?
