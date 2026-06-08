# Présentation du SDK Agent

> Créez des agents IA de production avec Claude Code en tant que bibliothèque

Note: Starting June 15, 2026, Agent SDK and `claude -p` usage on subscription plans will draw from a new monthly Agent SDK credit, separate from your interactive usage limits.

Créez des agents IA qui lisent autonomement les fichiers, exécutent des commandes, recherchent sur le web, modifient le code, et bien plus. Le SDK Agent vous offre les mêmes outils, boucle d'agent et gestion du contexte qui alimentent Claude Code, programmables en Python et TypeScript.

## Installation

```python
pip install claude-agent-sdk
```

```typescript
npm install @anthropic-ai/claude-agent-sdk
```

## Exemple minimal

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="Find and fix the bug in auth.py",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"]),
    ):
        print(message)

asyncio.run(main())
```

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Find and fix the bug in auth.ts",
  options: { allowedTools: ["Read", "Edit", "Bash"] }
})) {
  console.log(message);
}
```

## Outils intégrés

| Outil | Ce qu'il fait |
|-------|---------------|
| Read | Lire n'importe quel fichier du répertoire de travail |
| Write | Créer de nouveaux fichiers |
| Edit | Effectuer des modifications précises aux fichiers existants |
| Bash | Exécuter des commandes de terminal, des scripts, des opérations git |
| Monitor | Surveiller un script en arrière-plan et réagir à chaque ligne de sortie |
| Glob | Trouver des fichiers par motif |
| Grep | Rechercher le contenu des fichiers avec regex |
| WebSearch | Rechercher sur le web |
| WebFetch | Récupérer et analyser le contenu des pages web |
| AskUserQuestion | Poser à l'utilisateur des questions de clarification |

## Capacités principales

- **Hooks** : PreToolUse, PostToolUse, Stop, SessionStart, SessionEnd, UserPromptSubmit
- **Sous-agents** : Générer des agents spécialisés pour des sous-tâches
- **MCP** : Connecter des systèmes externes via Model Context Protocol
- **Permissions** : Contrôler quels outils l'agent peut utiliser
- **Sessions** : Maintenir le contexte sur plusieurs échanges

## Fonctionnalités de Claude Code disponibles via le SDK

| Fonctionnalité | Description | Emplacement |
|----------------|-------------|-------------|
| Skills | Capacités spécialisées | `.claude/skills/*/SKILL.md` |
| Commands | Commandes personnalisées (héritage) | `.claude/commands/*.md` |
| Memory | Contexte du projet | `CLAUDE.md` |
| Plugins | Étendre avec skills, agents, hooks, MCP | Via l'option `plugins` |

## Comparaison avec d'autres outils Claude

### SDK Agent vs SDK Client Anthropic
- SDK Client : vous implémentez la boucle d'outils
- SDK Agent : Claude gère les outils de façon autonome

### SDK Agent vs CLI Claude Code
- CLI : développement interactif, tâches ponctuelles
- SDK : pipelines CI/CD, applications personnalisées, automatisation

### SDK Agent vs Agents gérés
| | SDK Agent | Agents gérés |
|--|-----------|--------------|
| S'exécute dans | Votre processus | Infrastructure Anthropic |
| Interface | Python/TypeScript | API REST |
| Idéal pour | Prototypage local | Production sans infrastructure |

## Authentification

- API Anthropic: `ANTHROPIC_API_KEY`
- Amazon Bedrock: `CLAUDE_CODE_USE_BEDROCK=1`
- Google Vertex AI: `CLAUDE_CODE_USE_VERTEX=1`
- Microsoft Azure: `CLAUDE_CODE_USE_FOUNDRY=1`

## Directives de marque

- Autorisé: "Claude Agent", "Claude", "{Name} Powered by Claude"
- Non autorisé: "Claude Code" ou "Claude Code Agent"
