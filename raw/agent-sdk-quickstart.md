# Agent SDK — Démarrage rapide

Source: https://code.claude.com/docs/fr/agent-sdk/quickstart

## Configuration

```bash
mkdir my-agent && cd my-agent

# TypeScript
npm install @anthropic-ai/claude-agent-sdk

# Python (uv)
uv init && uv add claude-agent-sdk

# Python (pip)
python3 -m venv .venv && source .venv/bin/activate
pip install claude-agent-sdk
```

Clé API dans `.env` :
```
ANTHROPIC_API_KEY=your-api-key
```

## Exemple minimal (Python)

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ResultMessage

async def main():
    async for message in query(
        prompt="Review utils.py for bugs that would cause crashes. Fix any issues you find.",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit", "Glob"],
            permission_mode="acceptEdits",
        ),
    ):
        if isinstance(message, AssistantMessage):
            for block in message.content:
                if hasattr(block, "text"):
                    print(block.text)
                elif hasattr(block, "name"):
                    print(f"Tool: {block.name}")
        elif isinstance(message, ResultMessage):
            print(f"Done: {message.subtype}")

asyncio.run(main())
```

## Exemple minimal (TypeScript)

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Review utils.py for bugs that would cause crashes. Fix any issues you find.",
  options: {
    allowedTools: ["Read", "Edit", "Glob"],
    permissionMode: "acceptEdits"
  }
})) {
  if (message.type === "assistant" && message.message?.content) {
    for (const block of message.message.content) {
      if ("text" in block) console.log(block.text);
      else if ("name" in block) console.log(`Tool: ${block.name}`);
    }
  } else if (message.type === "result") {
    console.log(`Done: ${message.subtype}`);
  }
}
```

## Modes de permission SDK

| Mode | Comportement | Cas d'usage |
|------|-------------|-----------|
| `acceptEdits` | Approuve auto les modifications de fichiers + commandes FS | Développement de confiance |
| `dontAsk` | Refuse tout hors `allowedTools` | Agents sans tête verrouillés |
| `auto` (TS uniquement) | Classificateur approuve/refuse chaque appel | Autonome avec garde-fous |
| `bypassPermissions` | Exécute tout sans invite | CI en bac à sable |
| `default` | Nécessite callback `canUseTool` | Flux d'approbation personnalisés |

## Outils par combinaison

| Outils | Capacités |
|--------|---------|
| `Read`, `Glob`, `Grep` | Analyse en lecture seule |
| `Read`, `Edit`, `Glob` | Analyser et modifier |
| `Read`, `Edit`, `Bash`, `Glob`, `Grep` | Automatisation complète |

## Personnaliser

```python
# Ajouter WebSearch
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Edit", "Glob", "WebSearch"],
    permission_mode="acceptEdits"
)

# Invite système personnalisée
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Edit", "Glob"],
    permission_mode="acceptEdits",
    system_prompt="You are a senior Python developer. Always follow PEP 8."
)

# Avec Bash (tests auto-correctifs)
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Edit", "Glob", "Bash"],
    permission_mode="acceptEdits"
)
# Prompt: "Write unit tests for utils.py, run them, and fix any failures"
```

## Dépannage

**"API key not found"** : vérifier `ANTHROPIC_API_KEY` dans `.env` ou shell.
**Erreur `thinking.type.enabled`** : upgrader vers SDK Agent v0.2.111+ pour Opus 4.7.
