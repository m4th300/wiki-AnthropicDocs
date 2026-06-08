# Application de bureau Claude Code

Source: https://code.claude.com/docs/fr/desktop

Référence complète pour l'onglet Code de l'application Claude Desktop.

## Installation

- macOS : Universal (Intel + Apple Silicon)
- Windows : x64 ou ARM64
- Linux : non disponible (utiliser la CLI)

Prérequis Windows : Git for Windows (redémarrer l'app après installation).

## Onglets de l'application

- **Chat** : conversation générale sans accès aux fichiers
- **Cowork** : agent autonome en arrière-plan (VM cloud)
- **Code** : assistant de codage interactif avec accès aux fichiers locaux

## Types d'environnement

- **Local** : votre machine, accès direct aux fichiers
- **Remote** : infrastructure cloud Anthropic (continue même si app fermée)
- **SSH** : machine distante via SSH (Claude Code installé automatiquement)

## Sessions parallèles

La barre latérale permet de gérer plusieurs sessions simultanées.
Chaque session = son propre git worktree isolé.

Volet tâches : voir les sous-agents et commandes en arrière-plan.

## Modes de permission

- **Demander les permissions** : défaut, approuver chaque modification
- **Auto-accepter les modifications** : édite sans demander
- **Plan mode** : cartographie l'approche sans toucher aux fichiers

## Vue de différence

Affiche les modifications file par file. Commentaires en ligne sur les lignes spécifiques.

## Fonctionnalités Desktop

- **Preview** : voir l'app en cours d'exécution (menu déroulant Preview)
- **Suivi PR** : surveille CI, corrige/fusionne automatiquement quand vérifications passent
- **Tâches planifiées** : runs récurrents automatiques (review quotidien, audit hebdo, etc.)
- **Chat latéral** : question sans dérailler la session principale
- **Sessions distantes** : lancer des tâches longues sur le cloud

## Plugins

Bouton `+` → Plugins pour parcourir et installer depuis les marketplaces.

## Dispatch (depuis le téléphone)

Envoyer une tâche depuis l'application mobile Claude → génère une session Desktop.

## Sessions from Dispatch

Sessions créées depuis Dispatch apparaissent dans la barre latérale Desktop.

## Suivi de PR

Après ouverture d'une PR : Claude surveille les CI checks.
Options : corriger les échecs automatiquement, fusionner quand tout passe.

## Paramètres Desktop

Configuration partagée avec la CLI : CLAUDE.md, MCP, hooks, skills, paramètres.

## Venant de la CLI

Toutes les fonctionnalités CLI disponibles via terminal intégré.
`/desktop` pour déplacer une session terminal vers l'application Desktop.

## Comparaison CLI vs Desktop

| Fonctionnalité | CLI | Desktop |
|----------------|-----|---------|
| Toutes les commandes | ✓ | Via terminal intégré |
| Sessions parallèles | Worktrees manuels | Interface graphique |
| Vue de différence | Terminal | Graphique |
| Plugins | `/plugin` | Interface graphique |
| Suivi PR | Manuel | Automatique |
| Tâches planifiées | `/loop` | Interface graphique |

## Workspace arrangement

Glisser-déposer les volets (chat, diff, terminal, fichier, preview) dans la disposition souhaitée.
Terminal : `Ctrl+\`` pour ouvrir.

## Utilisation de l'ordinateur

Claude peut utiliser votre ordinateur (applications, fenêtres, clics) via Computer Use.
Option dans les paramètres Desktop.
