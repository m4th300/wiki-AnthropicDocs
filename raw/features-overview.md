# Étendre Claude Code

Source: https://code.claude.com/docs/fr/features-overview

## Vue d'ensemble des extensions

| Extension | Ce qu'elle fait | Quand l'utiliser |
|-----------|----------------|-----------------|
| **CLAUDE.md** | Contexte persistant à chaque conversation | Conventions de projet, règles "toujours faire X" |
| **Skills** | Instructions, connaissances, workflows invocables | Contenu réutilisable, tâches répétables |
| **Code intelligence** | Navigation et diagnostics du serveur de langage | Langages typés, grands codebases |
| **MCP** | Connexion aux services externes | Données ou actions externes |
| **Subagents** | Contexte d'exécution isolé qui retourne des résultats | Isolation contexte, tâches parallèles |
| **Agent teams** | Coordonne plusieurs sessions Claude indépendantes | Recherche parallèle, hypothèses concurrentes |
| **Hooks** | Script/HTTP/prompt déclenché par des événements | Automatisation déterministe sur chaque événement |
| **Plugins** | Package de skills + hooks + agents + MCP | Distribution entre projets/équipes |

## Quand construire quoi (déclencheurs)

| Déclencheur | Ajouter |
|-------------|---------|
| Claude fait la même erreur une deuxième fois | CLAUDE.md |
| Vous tapez la même invite pour démarrer une tâche | Skill invocable par utilisateur |
| Vous collez le même playbook pour la 3ème fois | Skill |
| Vous copiez des données d'un onglet que Claude ne voit pas | Serveur MCP |
| Claude lit de nombreux fichiers pour trouver un symbole | Plugin code intelligence |
| Une tâche secondaire inonde votre conversation | Subagent |
| Quelque chose doit se produire à chaque fois sans demander | Hook |
| Un deuxième repo a besoin de la même config | Plugin |

## Comparaisons entre fonctionnalités similaires

### Skill vs Subagent

| Aspect | Skill | Subagent |
|--------|-------|---------|
| Ce que c'est | Instructions/connaissances réutilisables | Travailleur isolé |
| Impact context window | S'ajoute à votre fenêtre | Fenêtre séparée |
| Meilleur pour | Matériel de référence, workflows invocables | Tâches lisant de nombreux fichiers, parallélisation |

### CLAUDE.md vs Skills

| CLAUDE.md | Skill |
|-----------|-------|
| Se charge à chaque session | À la demande |
| Pas de workflows déclenchables | `/skill-name` déclenchable |
| Règles "toujours faire X" | Matériel de référence, procédures |
| Max 200 lignes recommandées | Contenu long OK (charge à la demande) |

### Hook vs Skill

| Aspect | Hook | Skill |
|--------|------|-------|
| S'exécute | Commande shell/HTTP/LLM | Instructions que Claude lit |
| Déclenché par | Événements fixes du cycle de vie | Vous ou Claude selon pertinence |
| Déterminisme | Toujours sur son événement | Claude interprète → peut varier |
| Meilleur pour | Linting, blocage, logging | Raisonnement, multi-étapes |

**Règle clé** : si ça DOIT se produire à chaque fois → hook. Si Claude doit DÉCIDER comment appliquer → skill.

### Subagent vs Agent team

| Aspect | Subagent | Agent team |
|--------|---------|------------|
| Communication | Rapporte au parent uniquement | Coéquipiers se messagent directement |
| Token cost | Inférieur (résultats résumés) | Supérieur (~7x) |
| Meilleur pour | Tâche ciblée, résultat seulement | Collaboration, hypothèses concurrentes |

Agent teams = expérimental, désactivé par défaut (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`).

## Coûts du contexte par fonctionnalité

| Fonctionnalité | Quand se charge | Coût contexte |
|----------------|----------------|---------------|
| CLAUDE.md | Début de session (complet) | Chaque requête |
| Skills | Descriptions au démarrage, contenu à l'usage | Faible jusqu'à usage |
| MCP | Noms d'outils au démarrage, schémas à la demande | Faible (avec Tool Search) |
| Code intelligence | Après modifications + à la demande | Faible, réduit les lectures |
| Subagents | Contexte frais au lancement | Isolé du parent |
| Hooks | Au déclenchement (externe) | Zéro sauf si sortie |

### Optimiser les coûts de contexte

- **Skills `disable-model-invocation: true`** : 0 coût jusqu'à invocation manuelle
- **MCP Tool Search** (par défaut) : seuls les noms d'outils au démarrage
- **`skillOverrides`** dans les paramètres : masquer des skills sans modifier leurs fichiers
