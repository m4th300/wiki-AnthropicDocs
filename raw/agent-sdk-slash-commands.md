# Slash Commands dans le SDK Agent

> Apprenez à utiliser les slash commands pour contrôler les sessions Claude Code via le SDK

Les slash commands contrôlent les sessions avec des commandes spéciales commençant par `/`. Envoyées via le SDK comme prompts normaux.

Seules les commandes fonctionnant sans terminal interactif peuvent être envoyées via le SDK.

## Découvrir les commandes disponibles

```typescript
for await (const message of query({ prompt: "Hello", options: { maxTurns: 1 } })) {
  if (message.type === "system" && message.subtype === "init") {
    console.log("Available slash commands:", message.slash_commands);
    // Exemple: ["clear", "compact", "context", "usage"]
  }
}
```

## Envoyer des slash commands

```python
async for message in query(prompt="/compact", options=ClaudeAgentOptions(max_turns=1)):
    if isinstance(message, ResultMessage):
        print("Command executed:", message.result)
```

## Commandes courantes

### `/compact` — Compacter l'historique

```python
async for message in query(prompt="/compact", options=ClaudeAgentOptions(max_turns=1)):
    if isinstance(message, SystemMessage) and message.subtype == "compact_boundary":
        print("Pre-compaction tokens:", message.data["compact_metadata"]["pre_tokens"])
```

### `/clear` — Réinitialiser le contexte

Réinitialise la conversation à un contexte vide. La conversation précédente reste sur disque et peut être reprise.

Note: `/clear` nécessite Claude Code v2.1.117+. Utile en mode d'entrée en continu. Pour les appels `query()` ponctuels, chaque appel démarre déjà avec contexte vide.

## Slash Commands personnalisées

Format hérité: fichiers markdown dans `.claude/commands/`
Format recommandé: `.claude/skills/<name>/SKILL.md` (supporte aussi l'invocation `/name`)

### Emplacements
- **Commandes projet** : `.claude/commands/` (héritées; préférer `.claude/skills/`)
- **Commandes personnelles** : `~/.claude/commands/`

### Format de fichier

Nom du fichier (sans `.md`) = nom de la commande.

```markdown
---
allowed-tools: Read, Grep, Glob
description: Run security vulnerability scan
model: claude-opus-4-7
---

Analyze the codebase for security vulnerabilities including:
- SQL injection risks
- XSS vulnerabilities
```

### Arguments et placeholders

```markdown
---
argument-hint: [issue-number] [priority]
description: Fix a GitHub issue
---

Fix issue #$0 with priority $1.
```

### Exécution de commandes bash dans les commandes

```markdown
---
allowed-tools: Bash(git add *), Bash(git status *), Bash(git commit *)
---

## Context
- Current status: !`git status`
- Current diff: !`git diff HEAD`

Create a git commit with appropriate message based on the changes.
```

### Références de fichiers

```markdown
---
description: Review configuration files
---

Review: @package.json and @tsconfig.json
```

## Utiliser les commandes personnalisées via le SDK

```python
async for message in query(
    prompt="/refactor src/auth/login.py",
    options=ClaudeAgentOptions(max_turns=3)
):
    if isinstance(message, AssistantMessage):
        for block in message.content:
            if hasattr(block, "text"):
                print(block.text)
```

Les commandes personnalisées apparaissent dans `slash_commands` du message `init`.
