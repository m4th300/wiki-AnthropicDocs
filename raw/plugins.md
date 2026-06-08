# Créer des plugins

Source: https://code.claude.com/docs/fr/plugins

## Plugins vs Configuration autonome

| Approche | Noms des skills | Idéal pour |
|---------|----------------|-----------|
| Autonome (`.claude/`) | `/hello` | Personnel, spécifique à un projet, expérimentation |
| Plugins | `/plugin-name:hello` | Partage équipe, distribution, multi-projets |

## Structure d'un plugin

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # Manifeste (nom, version, description)
├── skills/
│   └── my-skill/
│       └── SKILL.md
├── agents/              # Définitions d'agents
├── hooks/
│   └── hooks.json       # Gestionnaires d'événements
├── .mcp.json            # Serveurs MCP
├── .lsp.json            # Serveurs LSP
├── monitors/
│   └── monitors.json    # Moniteurs en arrière-plan
├── commands/            # Format legacy (utiliser skills/ pour les nouveaux)
├── bin/                 # Exécutables ajoutés au PATH
└── settings.json        # Paramètres par défaut
```

⚠️ Ne PAS mettre `commands/`, `agents/`, `skills/`, `hooks/` DANS `.claude-plugin/`. Seulement `plugin.json` va dans `.claude-plugin/`.

## Manifeste (plugin.json)

```json
{
  "name": "my-plugin",
  "description": "A greeting plugin",
  "version": "1.0.0",
  "author": {"name": "Your Name"}
}
```

- `name` = espace de noms des skills (`/my-plugin:hello`)
- `version` : sans version → SHA du commit utilisé

## Skills dans un plugin

```markdown
---
description: Greet the user with a personalized message
disable-model-invocation: true
---

Greet the user named "$ARGUMENTS" warmly.
```

## Tester localement

```bash
claude --plugin-dir ./my-plugin
claude --plugin-dir ./my-plugin.zip         # Archive zip (v2.1.128+)
claude --plugin-url https://example.com/my-plugin.zip
```

Plusieurs plugins :
```bash
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
```

`/reload-plugins` pour récupérer les modifications sans redémarrer.

## Développer dans le répertoire de skills

```bash
claude plugin init my-tool    # Crée ~/.claude/skills/my-tool/
```

Se charge automatiquement en tant que `my-tool@skills-dir` sans marketplace.

## Hooks dans un plugin

`hooks/hooks.json` :
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{"type": "command", "command": "npm run lint:fix"}]
    }]
  }
}
```

## Serveurs MCP dans un plugin

`.mcp.json` à la racine du plugin (même format que le `.mcp.json` de projet).

## Moniteurs en arrière-plan

`monitors/monitors.json` :
```json
[{"name": "error-log", "command": "tail -F ./logs/error.log", "description": "Error log"}]
```

## Paramètres par défaut via settings.json

```json
{"agent": "security-reviewer"}
```

## Partager et distribuer

1. Ajouter un README.md
2. Choisir stratégie de versioning
3. Héberger dans un repo Git ou marketplace
4. Soumettre à la marketplace communautaire : claude.ai/settings/plugins/submit

### Marketplaces publiques Anthropic

- `claude-plugins-official` : plugins maintenus par Anthropic (disponible automatiquement)
- `claude-community` : soumissions tierces après examen

### Valider

```bash
claude plugin validate
```

## Convertir la config autonome en plugin

```bash
cp -r .claude/commands my-plugin/
cp -r .claude/agents my-plugin/
cp -r .claude/skills my-plugin/
# Créer .claude-plugin/plugin.json
```

Après migration : supprimer les fichiers originaux de `.claude/` pour éviter les doublons.
