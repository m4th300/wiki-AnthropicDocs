# Comment fonctionne Claude Code

Source: https://code.claude.com/docs/fr/how-claude-code-works

## La boucle agentive

Trois phases : **rassembler le contexte** → **agir** → **vérifier les résultats** → répéter.
Claude enchaîne des dizaines d'actions, se corrige en cours de route. Vous pouvez interrompre à tout moment.

## Composants

- **Modèles** : Claude (Sonnet par défaut, Opus pour raisonnement complexe). `/model` pour changer.
- **Outils** : ce qui rend Claude agentique — sans outils, Claude ne peut que répondre avec du texte.

## Catégories d'outils

| Catégorie | Ce que Claude peut faire |
|-----------|--------------------------|
| Opérations sur les fichiers | Lire, éditer, créer, renommer |
| Recherche | Trouver des fichiers par motif, regex, explorer |
| Exécution | Commandes shell, serveurs, tests, git |
| Web | Rechercher, récupérer documentation |
| Intelligence du code | Erreurs de type, définitions, références (nécessite plugins) |

## Ce que Claude peut accéder

- Votre projet (répertoire courant + sous-répertoires)
- Votre terminal (toute commande CLI)
- État git (branche, modifications, historique)
- CLAUDE.md (instructions persistantes)
- Mémoire automatique (~/.claude/projects/, 200 premières lignes de MEMORY.md)
- Extensions configurées (MCP, skills, subagents, Chrome)

## Environnements d'exécution

| Environnement | Où le code s'exécute | Cas d'usage |
|---------------|---------------------|-------------|
| Local | Votre machine | Par défaut, accès complet |
| Cloud | VMs Anthropic | Déléguer des tâches, repos non locaux |
| Contrôle à distance | Votre machine depuis navigateur | Interface web + local |

## Sessions

- Sauvegardées localement sous `~/.claude/projects/` (JSONL)
- Snapshots de fichiers avant chaque édition (checkpoints)
- Sessions indépendantes (contexte frais à chaque démarrage)
- Reprise : `claude --continue` ou `claude --resume`
- Fork : `--fork-session` ou `/branch`

## Fenêtre de contexte

Contient : historique de conversation, fichiers, sorties de commande, CLAUDE.md, auto memory, skills, instructions système.
- `/context` pour voir l'utilisation
- Compaction automatique quand la limite approche (efface sorties anciennes, résume)
- CLAUDE.md + auto memory survivent à la compaction

## Modes de permission (Shift+Tab pour changer)

- **Par défaut** : demande avant éditions et commandes shell
- **Auto-accepter les éditions** : édite sans demander, demande pour commandes
- **Plan Mode** : lecture seule, propose un plan avant exécution
- **Mode Auto** : classificateur en arrière-plan, aperçu de recherche

## Conseils d'utilisation efficace

- C'est conversationnel : commencez, puis itérez
- Interrompre avec `Esc` ou taper une correction sans attendre
- Être spécifique dès le départ (fichiers, contraintes, exemples)
- Donner quelque chose à vérifier (tests, captures d'écran)
- Utiliser Plan Mode pour explorer avant d'implémenter
- Déléguer le "comment", pas le "quoi"
