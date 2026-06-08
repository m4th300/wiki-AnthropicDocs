# Agent Skills dans le SDK

> Étendez Claude avec des capacités spécialisées en utilisant Agent Skills dans le Claude Agent SDK

## Comment les Skills fonctionnent avec le SDK

1. **Définis comme artefacts du système de fichiers** : fichiers `SKILL.md` dans `.claude/skills/`
2. **Chargés à partir du système de fichiers** : via `settingSources`
3. **Découverts automatiquement** : au démarrage
4. **Invoqués par le modèle** : Claude choisit quand les utiliser
5. **Filtrés via l'option `skills`** : contrôle lesquels sont disponibles

Contrairement aux sous-agents, les Skills doivent être créés comme artefacts du système de fichiers (pas d'API programmatique).

## Emplacements

- **Project Skills** : `.claude/skills/` (partagés via git, chargés si `"project"` dans setting_sources)
- **User Skills** : `~/.claude/skills/` (tous les projets, chargés si `"user"` dans setting_sources)
- **Plugin Skills** : fournis avec les plugins installés

## Utilisation via le SDK

```python
options = ClaudeAgentOptions(
    cwd="/path/to/project",
    setting_sources=["user", "project"],  # Charger les skills du système de fichiers
    skills="all",                          # Activer tous les skills découverts
    allowed_tools=["Read", "Write", "Bash"],
)

async for message in query(
    prompt="Help me process this PDF document",
    options=options
):
    print(message)
```

## Option `skills`

- `"all"` : activer chaque skill découvert
- `["pdf", "docx"]` : activer uniquement ces skills spécifiques
- `[]` : désactiver tous les skills
- Omis : skills découverts activés (comportement CLI par défaut)

Noms: champ `name` dans `SKILL.md` ou nom du répertoire. Format `plugin:skill` pour les skills de plugins.

Note: `skills` est un filtre de contexte, pas un sandbox. Les skills non listés sont masqués au modèle mais leurs fichiers restent accessibles via Read/Bash.

## Découverte des skills disponibles

```python
async for message in query(
    prompt="What Skills are available?",
    options=ClaudeAgentOptions(setting_sources=["user", "project"], skills="all")
):
    print(message)
```

## Restrictions d'outils (SDK)

Le frontmatter `allowed-tools` dans SKILL.md ne s'applique PAS via le SDK. Contrôler via `allowedTools` dans la configuration de requête.

## Dépannage

### Skills non trouvés
- Vérifier que `settingSources` inclut `"user"` et/ou `"project"`
- Vérifier que `cwd` pointe vers le bon répertoire (ou un parent jusqu'à la racine du repo)
- `ls .claude/skills/*/SKILL.md` et `ls ~/.claude/skills/*/SKILL.md`

### Skill non utilisé
- Vérifier l'option `skills` (doit inclure le nom du skill)
- Améliorer la description dans SKILL.md pour correspondre à plus de requêtes
