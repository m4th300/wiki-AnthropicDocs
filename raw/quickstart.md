# Démarrage rapide — Claude Code

Source: https://code.claude.com/docs/fr/quickstart

## Prérequis

- Terminal ouvert
- Un projet de code
- Abonnement Claude (Pro, Max, Team, Enterprise), compte Console, ou fournisseur cloud

## Étape 1 : Installer

macOS/Linux/WSL : `curl -fsSL https://claude.ai/install.sh | bash`
Windows PowerShell : `irm https://claude.ai/install.ps1 | iex`
Windows CMD : `curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd`
Homebrew : `brew install --cask claude-code`
WinGet : `winget install Anthropic.ClaudeCode`

## Étape 2 : Se connecter

```bash
claude
# Suit les invites → authentification navigateur
# Changer de compte plus tard : /login
```

Types de comptes acceptés : Claude Pro/Max/Team/Enterprise, Claude Console, Amazon Bedrock, Google Vertex AI, Microsoft Foundry.

## Étape 3 : Démarrer une session

```bash
cd /path/to/your/project
claude
```

`/help` pour les commandes, `/resume` pour continuer une conversation.

## Étapes 4-8 : Utilisation de base

```text
what does this project do?          # Explorer la base de code
add a hello world function          # Modifier du code
commit my changes                   # Git
add input validation to the form    # Fonctionnalités
refactor the auth module            # Refactoring
write unit tests for X              # Tests
update the README                   # Documentation
review my changes                   # Revue de code
```

## Commandes essentielles

| Commande | Usage |
|----------|-------|
| `claude` | Mode interactif |
| `claude "task"` | Tâche unique |
| `claude -p "query"` | Requête puis quitte |
| `claude -c` | Continuer dernière conversation |
| `claude -r` | Reprendre conversation précédente |
| `/clear` | Effacer historique |
| `/help` | Aide |
| `exit` ou Ctrl+D | Quitter |

## Conseils débutants

- Être spécifique : "fix the login bug where users see a blank screen after wrong credentials"
- Utiliser des étapes : "1. create DB table 2. create API endpoint 3. build webpage"
- Laisser Claude explorer d'abord : "analyze the database schema"
- Raccourcis : `/` pour commandes, Tab pour complétion, ↑ pour historique, Shift+Tab pour modes
