# Skills (Étendre Claude)

Source: https://code.claude.com/docs/fr/skills

## Ce que c'est

Les skills étendent ce que Claude peut faire. Un fichier `SKILL.md` avec des instructions → Claude l'ajoute à sa boîte à outils.

- Claude utilise les skills quand c'est pertinent (automatiquement)
- Ou vous l'invoquez avec `/skill-name`
- Le corps d'une skill ne se charge que quand elle est utilisée (vs CLAUDE.md qui se charge toujours)

**Note :** Les commandes personnalisées (`.claude/commands/`) ont été fusionnées dans les skills.
`deploy.md` dans commands/ et `SKILL.md` dans skills/ créent tous deux `/deploy`.
Les fichiers commands/ continuent de fonctionner.

## Quand créer une skill

- Vous collez le même playbook dans le chat régulièrement
- Une section de CLAUDE.md est devenue une procédure multi-étapes
- Une tâche répétable peut être partagée avec l'équipe

## Structure d'une skill

```
.claude/skills/my-skill/
├── SKILL.md    # Fichier principal (requis)
└── helpers/    # Fichiers de support optionnels
```

### Format SKILL.md

```markdown
---
description: "Révision de code approfondie"
tools: [Read, Bash, Edit]
disable-model-invocation: false
---

# Code Review Skill

Instructions pour Claude sur comment effectuer une révision de code...
```

## Emplacements des skills

| Emplacement | Portée |
|------------|--------|
| `~/.claude/skills/` | Tous les projets |
| `.claude/skills/` | Projet (partageable) |
| `~/.claude/commands/` | Tous les projets (legacy) |
| `.claude/commands/` | Projet (legacy) |

## Skills groupées (bundled)

Incluses avec Claude Code :
- `/batch` : divise un grand changement en sous-agents parallèles
- `/code-review` | `/review` : révision du code
- `/debug` : débogage
- `/loop` : répète une invite à intervalles
- `/plan` : planification

Voir la [référence des commandes](/fr/commands) pour la liste complète.

## Contrôler l'invocation

### Via frontmatter

```markdown
---
disable-model-invocation: true   # Skill manuelle uniquement (pas auto-invoquée)
trigger: "code review"           # Mots déclencheurs pour invocation automatique
---
```

### Via settings.json

```json
{
  "skillOverrides": {
    "my-skill": {
      "disableModelInvocation": true
    }
  }
}
```

## Subagents dans les skills

Les skills peuvent lancer des subagents pour le travail délégué :

```markdown
---
subagent: true
model: claude-sonnet-4-6
tools: [Read, Bash]
---
# Research Agent
Explore the codebase and report findings...
```

## Plugins et marketplaces

Les skills peuvent être packagées dans des plugins et distribuées via des marketplaces.
Un plugin = ensemble de skills + hooks + subagents + serveurs MCP.

Installation : `bouton +` → Plugins (dans Desktop) ou via `/plugins`.
