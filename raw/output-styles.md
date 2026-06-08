# Styles de sortie

Source: https://code.claude.com/docs/fr/output-styles

## Ce que c'est

Les styles de sortie modifient l'invite système pour définir le rôle, le ton et le format de réponse.
Utile quand vous voulez un comportement différent à chaque tour sans répéter vos instructions.

## Styles intégrés

| Style | Comportement |
|-------|-------------|
| **Default** | L'invite système existante de Claude Code pour l'ingénierie logicielle |
| **Proactive** | S'exécute immédiatement, fait des hypothèses raisonnables, préfère l'action à la planification |
| **Explanatory** | Fournit des "Insights" éducatifs entre les tâches de codage |
| **Learning** | Mode d'apprentissage : ajoute des marqueurs `TODO(human)` pour que vous implementiez des parties |

Note : Proactive ≠ Mode auto. Proactive = instruction comportementale. Mode auto = classificateur d'approbation.

## Changer de style

```text
/config    # → Output style → choisir dans le menu
```

Ou directement dans un fichier de paramètres :
```json
{ "outputStyle": "Explanatory" }
```

Important : Le style fait partie de l'invite système → les changements prennent effet après `/clear` ou nouvelle session.

## Créer un style personnalisé

Créer un fichier Markdown dans :
- `~/.claude/output-styles/` (niveau utilisateur)
- `.claude/output-styles/` (niveau projet)
- Répertoire managed settings + `/output-styles/` (politique gérée)

```markdown
---
name: Diagrams first
description: Lead every explanation with a diagram
keep-coding-instructions: true
---

When explaining code, start with a Mermaid diagram, then explain in prose.

## Diagram conventions
Use `flowchart TD` for control flow and `sequenceDiagram` for request paths.
```

### Frontmatter

| Champ | Description | Par défaut |
|-------|-------------|---------|
| `name` | Nom du style (sinon = nom du fichier) | Nom du fichier |
| `description` | Affiché dans `/config` | Aucun |
| `keep-coding-instructions` | Conserver les instructions d'ingénierie logicielle intégrées | `false` |
| `force-for-plugin` | Appliquer automatiquement quand le plugin est activé | `false` |

Note : **Sans `keep-coding-instructions: true`** → les instructions d'ingénierie logicielle sont SUPPRIMÉES.
Utiliser sans `keep-coding-instructions` quand Claude ne fait pas d'ingénierie logicielle du tout.

## Comparaison avec les fonctionnalités connexes

| Fonctionnalité | Fonctionnement | Quand utiliser |
|----------------|---------------|----------------|
| **Styles de sortie** | Modifie l'invite système | Rôle/ton/format différent à chaque tour |
| **CLAUDE.md** | Ajoute un message utilisateur | Conventions projet, contexte codebase |
| `--append-system-prompt` | Ajoute à l'invite système | Ajout ponctuel pour une seule invocation |
| **Agents** | Propre invite système, modèle, outils | Assistant à portée séparée |
| **Skills** | Chargées à la demande | Workflow réutilisable |

## Plugins et styles de sortie

Les plugins peuvent inclure des styles de sortie dans `output-styles/`.
`force-for-plugin: true` applique le style automatiquement quand le plugin est activé.
