# Plugins dans le SDK Agent

> Chargez des plugins personnalisés pour étendre Claude Code avec des skills, des agents, des hooks et des serveurs MCP

## Que sont les plugins ?

Packages d'extensions Claude Code incluant:
- **Skills** : capacités invoquées par le modèle (ou via `/skill-name`)
- **Agents** : sous-agents spécialisés
- **Hooks** : gestionnaires d'événements
- **Serveurs MCP** : intégrations via Model Context Protocol

Note: `.claude/commands/` est un format hérité. Utiliser `skills/` pour les nouveaux plugins.

## Chargement des plugins

```typescript
for await (const message of query({
  prompt: "Hello",
  options: {
    plugins: [
      { type: "local", path: "./my-plugin" },
      { type: "local", path: "/absolute/path/to/another-plugin" }
    ]
  }
}))
```

Le champ `type` doit être `"local"`. Pour les plugins distribués via marketplace/dépôt distant, les télécharger d'abord.

### Chemins

- **Relatifs** : résolus par rapport au cwd actuel
- **Absolus** : chemin complet du système de fichiers

Note: Le chemin doit pointer vers le répertoire racine du plugin (parent de `skills/`, `agents/`, etc.).

## Vérification de l'installation

```python
async for message in query(prompt="Hello", options=ClaudeAgentOptions(
    plugins=[{"type": "local", "path": "./my-plugin"}]
)):
    if isinstance(message, SystemMessage) and message.subtype == "init":
        print("Plugins:", message.data.get("plugins"))
        print("Skills:", message.data.get("skills"))
        # ["my-plugin:greet"]
        print("Commands:", message.data.get("slash_commands"))
        # ["compact", "context", "my-plugin:greet"]
```

## Utiliser les skills des plugins

Les skills des plugins sont espaces de noms avec le nom du plugin:

```python
async for message in query(
    prompt="/demo-plugin:greet",  # Namespace: plugin-name:skill-name
    options=ClaudeAgentOptions(
        plugins=[{"type": "local", "path": "./plugins/demo-plugin"}]
    ),
):
```

## Structure d'un plugin

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Manifeste optionnel
├── skills/                   # Agent Skills
│   └── my-skill/
│       └── SKILL.md
├── commands/                 # Héritage, utiliser skills/ à la place
│   └── custom-cmd.md
├── agents/                   # Agents personnalisés
│   └── specialist.md
├── hooks/                    # Gestionnaires d'événements
│   └── hooks.json
└── .mcp.json                # Serveurs MCP
```

Le manifeste `.claude-plugin/plugin.json` est optionnel. Sans lui, Claude Code découvre automatiquement les composants selon la disposition du répertoire.

## Cas d'usage courants

### Développement et test
```typescript
plugins: [{ type: "local", path: "./dev-plugins/my-plugin" }]
```

### Extensions spécifiques au projet
```typescript
plugins: [{ type: "local", path: "./project-plugins/team-workflows" }]
```

### Plusieurs sources
```typescript
plugins: [
  { type: "local", path: "./local-plugin" },
  { type: "local", path: "~/.claude/custom-plugins/shared-plugin" }
]
```

## Dépannage

- Plugin non chargé: vérifier le chemin (doit être le parent de `skills/`, `agents/`, etc.)
- Skills n'apparaissent pas: utiliser `/plugin-name:skill-name`, vérifier le message init
- Chemins relatifs: vérifier le répertoire de travail actuel
