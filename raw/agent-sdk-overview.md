# Agent SDK — Vue d'ensemble

Source: https://code.claude.com/docs/fr/agent-sdk/overview

**Note importante :** À partir du 15 juin 2026, l'utilisation de l'Agent SDK et de `claude -p` sur les plans abonnement tirera depuis un nouveau crédit mensuel Agent SDK, séparé des limites d'utilisation interactive.

## Ce que c'est

Une bibliothèque Python et TypeScript qui donne accès aux mêmes outils, boucle d'agent et gestion du contexte qui alimentent Claude Code — programmables dans vos propres applications.

## Démarrage rapide

### TypeScript
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Find and fix the bug in auth.ts",
  options: { allowedTools: ["Read", "Edit", "Bash"] }
})) {
  if ("result" in message) console.log(message.result);
}
```

### Python
```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="Find and fix the bug in auth.py",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"]),
    ):
        if hasattr(message, "result"):
            print(message.result)

asyncio.run(main())
```

## Installation

```bash
# TypeScript (inclut le binaire Claude Code natif)
npm install @anthropic-ai/claude-agent-sdk

# Python (nécessite Python 3.10+)
pip install claude-agent-sdk
```

## Authentification

```bash
export ANTHROPIC_API_KEY=your-api-key
```

Variables pour fournisseurs cloud :
- `CLAUDE_CODE_USE_BEDROCK=1` + credentials AWS
- `CLAUDE_CODE_USE_ANTHROPIC_AWS=1` + `ANTHROPIC_AWS_WORKSPACE_ID` + credentials AWS
- `CLAUDE_CODE_USE_VERTEX=1` + credentials Google Cloud
- `CLAUDE_CODE_USE_FOUNDRY=1` + credentials Azure

## Outils intégrés

| Outil | Ce qu'il fait |
|-------|--------------|
| `Read` | Lire des fichiers |
| `Write` | Créer de nouveaux fichiers |
| `Edit` | Modifier des fichiers existants |
| `Bash` | Exécuter des commandes terminal, git |
| `Monitor` | Surveiller un script en arrière-plan |
| `Glob` | Trouver des fichiers par pattern |
| `Grep` | Rechercher dans les fichiers avec regex |
| `WebSearch` | Rechercher sur le web |
| `WebFetch` | Récupérer le contenu d'une page web |
| `AskUserQuestion` | Poser des questions de clarification |

## Fonctionnalités

### Hooks (callbacks)
```python
async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get("tool_input", {}).get("file_path", "unknown")
    with open("./audit.log", "a") as f:
        f.write(f"{datetime.now()}: modified {file_path}\n")
    return {}

options=ClaudeAgentOptions(
    hooks={"PostToolUse": [HookMatcher(matcher="Edit|Write", hooks=[log_file_change])]}
)
```

### Sous-agents
```python
agents={"code-reviewer": AgentDefinition(
    description="Expert code reviewer",
    prompt="Analyze code quality and suggest improvements.",
    tools=["Read", "Glob", "Grep"],
)}
# Inclure "Agent" dans allowed_tools pour auto-approuver les invocations
```

### MCP
```python
mcp_servers={"playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}}
```

### Sessions
```python
# Capturer l'ID de session
if isinstance(message, SystemMessage) and message.subtype == "init":
    session_id = message.data["session_id"]

# Reprendre
options=ClaudeAgentOptions(resume=session_id)
```

### Skills et CLAUDE.md
Le SDK charge les configurations depuis `.claude/` et `~/.claude/` par défaut.

## Comparaison SDK Agent vs autres approches

| | SDK Agent | SDK Client Anthropic | Agents gérés |
|--|-----------|---------------------|-------------|
| Exécution des outils | Incluse (Claude gère) | Vous l'implémentez | Anthropic gère |
| Infrastructure | Votre process | Votre process | Infra Anthropic |
| Idéal pour | Prototypage, agents fichiers locaux | Contrôle total de la boucle | Production, sessions longues |

## Directives de marque

- ✅ "Claude Agent", "Claude", "{YourName} Powered by Claude"
- ❌ "Claude Code", "Claude Code Agent"
- Votre produit doit avoir sa propre marque

## Pages SDK détaillées

- agent-sdk/quickstart
- agent-sdk/python
- agent-sdk/typescript
- agent-sdk/agent-loop
- agent-sdk/features
- agent-sdk/custom-tools
- agent-sdk/cost-tracking
- agent-sdk/checkpointing
- agent-sdk/hooks
- agent-sdk/hosting
- agent-sdk/mcp
- agent-sdk/migration
- agent-sdk/system-prompts
- agent-sdk/observability
- agent-sdk/permissions
- agent-sdk/plugins
- agent-sdk/sessions
- agent-sdk/session-storage
- agent-sdk/skills
- agent-sdk/slash-commands
- agent-sdk/streaming
- agent-sdk/structured-outputs
- agent-sdk/subagents
- agent-sdk/tool-search
- agent-sdk/user-input
