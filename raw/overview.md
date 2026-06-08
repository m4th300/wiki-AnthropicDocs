# Aperçu — Claude Code

Source: https://code.claude.com/docs/fr/overview

Claude Code est un outil de codage agentique qui lit votre base de code, modifie les fichiers, exécute des commandes et s'intègre à vos outils de développement. Disponible dans votre terminal, IDE, application de bureau et navigateur.

## Surfaces disponibles

- **Terminal (CLI)** : `curl -fsSL https://claude.ai/install.sh | bash` (macOS/Linux/WSL) ou `irm https://claude.ai/install.ps1 | iex` (Windows PowerShell)
- **VS Code** : extension "Claude Code" (Cmd+Shift+X)
- **Application de bureau** : macOS et Windows, onglet "Code"
- **Web** : claude.ai/code (aperçu de recherche, Pro/Max/Team/Enterprise)
- **JetBrains** : plugin sur la Marketplace JetBrains

## Ce que vous pouvez faire

- Automatiser les tâches répétitives (tests, lint, merge conflicts, dépendances)
- Créer des fonctionnalités et corriger des bugs en langage naturel
- Créer des commits et des PRs directement via git
- Connecter des outils externes via MCP (Model Context Protocol)
- Personnaliser avec CLAUDE.md, skills et hooks
- Exécuter des équipes d'agents (subagents, agent view, agent teams)
- Piping, scripts et automatisation CLI (mode non-interactif)
- Planifier des tâches récurrentes (Routines, tâches planifiées de bureau, /loop)
- Travailler de n'importe où (contrôle à distance, Slack, mobile)

## Tableau des intégrations

| Besoin | Solution |
|--------|----------|
| Continuer depuis téléphone | Contrôle à distance |
| Événements Telegram/Discord/iMessage | Canaux |
| Tâche locale → mobile | Web ou app Claude iOS |
| Tâches récurrentes | Routines ou Tâches planifiées de bureau |
| Automatiser PRs/issues | GitHub Actions ou GitLab CI/CD |
| Revue de code automatique sur PR | Code Review GitHub |
| Rapports de bugs Slack → PRs | Slack |
| Déboguer apps web | Chrome |
| Agents personnalisés | Agent SDK |

## Étapes suivantes

- Démarrage rapide → /fr/quickstart
- Stocker instructions/mémoires → /fr/memory
- Flux de travail courants → /fr/common-workflows
- Meilleures pratiques → /fr/best-practices
- Paramètres → /fr/settings
- Résolution des problèmes → /fr/troubleshooting
