# Créer des sous-agents personnalisés

Source: https://code.claude.com/docs/fr/sub-agents

## Ce que c'est

Les sous-agents = assistants IA spécialisés exécutés dans leur propre fenêtre de contexte avec une invite système personnalisée, un accès à des outils spécifiques et des permissions indépendantes.

Utiliser quand une tâche secondaire inonderait votre conversation principale avec des résultats de recherche, des journaux ou du contenu de fichiers que vous ne référencerez plus.

Différent de :
- [Agent view](/fr/agent-view) : pour exécuter de nombreuses sessions indépendantes en parallèle
- [Agent teams](/fr/agent-teams) : pour des sessions qui communiquent entre elles

## Sous-agents intégrés

- **Explore** : recherche dans la codebase
- **Plan** : planification et architecture
- **Usage général** : tâches générales déléguées

## Quand utiliser un sous-agent

- Tâche secondaire qui lirait beaucoup de fichiers (ex: recherche dans une grande codebase)
- Vérification du travail (contexte frais pour évaluer objectivement)
- Travail parallèle qui ne doit pas polluer le contexte principal
- Travailleurs spécialisés avec des outils et permissions différents

## Créer un sous-agent personnalisé

Créer un fichier `.claude/agents/<name>.md` (ou `~/.claude/agents/` pour portée utilisateur) :

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
model: opus
---
You are a senior security engineer. Review code for:
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication and authorization flaws
- Secrets or credentials in code
- Insecure data handling

Provide specific line references and suggested fixes.
```

Utiliser : "Utilisez un subagent pour réviser ce code pour les problèmes de sécurité."

## Champs frontmatter supportés

| Champ | Description |
|-------|-------------|
| `name` | Identifiant du sous-agent |
| `description` | Description (utilisée par Claude pour décider quand déléguer) |
| `tools` | Liste des outils autorisés (Read, Edit, Bash, Glob, Grep, etc.) |
| `model` | Modèle à utiliser (opus, sonnet, haiku ou ID complet) |
| `isolation` | `worktree` pour donner au sous-agent un git worktree isolé |
| `skills` | Skills à précharger dans le contexte du sous-agent |
| `effort` | Niveau d'effort (low, medium, high, max) |

## Portée des sous-agents

| Emplacement | Portée |
|------------|--------|
| `.claude/agents/` | Projet (partageable) |
| `~/.claude/agents/` | Utilisateur (tous projets) |
| Dans les paramètres gérés | Organisation |

Pour les plugins : les agents de plugin sont espacés par noms (`plugin-name:agent-name`).

## Ce qui se charge au démarrage d'un sous-agent

- L'invite système du sous-agent (pas l'invite système complète de Claude Code)
- CLAUDE.md et statut git
- Skills listées dans le champ `skills:` (entièrement préchargées)
- Contexte passé dans l'invite par l'agent principal

**Les sous-agents Explore et Plan intégrés omettent CLAUDE.md et le statut git.**

## Choisir un modèle pour le sous-agent

Pour les tâches simples : utiliser `haiku` ou `sonnet` pour réduire les coûts.
Pour les tâches complexes : `opus`.
Pour les sous-agents (via SDK) : `CLAUDE_CODE_SUBAGENT_MODEL` pour définir le modèle de tous les sous-agents.

## Permissions

Les sous-agents s'exécutent dans leur propre contexte de permission.
Définir des règles dans le frontmatter avec `tools:` pour restreindre les outils.
Ou utiliser les règles de permission du projet/utilisateur.

## Activer la mémoire persistante

Les sous-agents peuvent maintenir leur propre mémoire automatique.
Activer : `autoMemoryEnabled: true` dans la configuration du sous-agent.
Voir la documentation de configuration des sous-agents pour les détails.

## Exécuter en avant-plan ou arrière-plan

Par défaut, les sous-agents s'exécutent en arrière-plan.
Pour exécuter en avant-plan : préfixer la demande avec "run this subagent in the foreground" ou utiliser `--foreground`.
`Ctrl+X Ctrl+K` : arrêter tous les sous-agents en arrière-plan de la session.

## Forker la conversation courante

Un sous-agent "fork" hérite du contexte complet de la conversation parent.
Différent d'un sous-agent standard qui commence avec un contexte vide.
Utiliser pour les vérifications : le fork voit tout ce que la session principale a fait.

## Dans les plugins

Les sous-agents peuvent être inclus dans des plugins et distribués via des marketplaces.
Les agents de plugin sont espacés par noms : `/my-plugin:security-reviewer`.

## Dans le SDK Agent

```python
agents={"code-reviewer": AgentDefinition(
    description="Expert code reviewer",
    prompt="Analyze code quality and suggest improvements.",
    tools=["Read", "Glob", "Grep"],
)}
# Inclure "Agent" dans allowed_tools pour auto-approuver les invocations
```

## Cas d'usage courants

```text
use a subagent to investigate how our auth system handles token refresh
use a subagent to review this code for edge cases
use subagents to investigate each of these hypotheses in parallel
Use a subagent to review the rate limiter diff against PLAN.md
```
