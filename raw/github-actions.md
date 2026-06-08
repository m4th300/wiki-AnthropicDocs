# Claude Code GitHub Actions

Source: https://code.claude.com/docs/fr/github-actions

## Vue d'ensemble

Claude Code GitHub Actions = intégration IA dans les workflows GitHub. Mention `@claude` dans PR/issue → Claude analyse, crée PRs, implémente, corrige.

Construit sur le Claude Agent SDK. Pour les revues automatiques sans déclencheur → voir [Code Review GitHub].

## Fonctionnalités

- Création instantanée de PR : décrire → Claude crée une PR complète
- Implémentation automatisée : issues → code fonctionnel
- Respecte CLAUDE.md et patterns de code existants
- Code reste sur les runners GitHub (sécurisé par défaut)

## Configuration rapide

```
/install-github-app    # dans Claude Code terminal
```

Nécessite : être admin du repo + accès API Claude direct (pas Bedrock/Vertex pour cette méthode).

## Configuration manuelle

1. Installer l'app Claude GitHub : https://github.com/apps/claude
   Permissions : Contents (R/W), Issues (R/W), Pull requests (R/W)
2. Ajouter `ANTHROPIC_API_KEY` aux secrets du repo
3. Copier le fichier workflow depuis `examples/claude.yml`

## Workflow basique

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # Responds to @claude mentions in comments
```

## Migration bêta → v1.0

| Ancien (bêta) | Nouveau (v1.0) |
|---------------|---------------|
| `mode` | Supprimé (détection auto) |
| `direct_prompt` | `prompt` |
| `custom_instructions` | `claude_args: --append-system-prompt` |
| `max_turns` | `claude_args: --max-turns` |
| `model` | `claude_args: --model` |

## Paramètres de l'action

| Paramètre | Description |
|-----------|-------------|
| `prompt` | Instructions Claude (texte ou nom de skill) |
| `claude_args` | Arguments CLI Claude Code |
| `plugin_marketplaces` | URL Git des marketplaces de plugins |
| `plugins` | Plugins à installer |
| `anthropic_api_key` | Clé API (requis pour API directe) |
| `trigger_phrase` | Déclencheur (défaut: "@claude") |
| `use_bedrock` | Utiliser Amazon Bedrock |
| `use_vertex` | Utiliser Google Vertex AI |

## Exemples d'usage dans les commentaires

```text
@claude implement this feature based on the issue description
@claude how should I implement user authentication for this endpoint?
@claude fix the TypeError in the user dashboard component
```

## Planification automatique

```yaml
name: Daily Report
on:
  schedule:
    - cron: "0 9 * * *"
jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Generate a summary of yesterday's commits and open issues"
          claude_args: "--model opus"
```

## Utilisation avec Bedrock/Vertex

### Pour AWS Bedrock
- GitHub OIDC Identity Provider configuré dans AWS
- Rôle IAM avec permissions `AmazonBedrockFullAccess`
- Secret `AWS_ROLE_TO_ASSUME`
- Format ID modèle Bedrock : `us.anthropic.claude-sonnet-4-6`

### Pour Google Vertex AI
- Workload Identity Federation configurée
- Compte de service avec rôle `Vertex AI User`
- Secrets `GCP_WORKLOAD_IDENTITY_PROVIDER` et `GCP_SERVICE_ACCOUNT`

## Optimisation des coûts

- GitHub Actions : consomme des minutes de runner
- API : coût en tokens selon longueur des prompts
- Utiliser `--max-turns` approprié
- Définir des timeouts au niveau du workflow
- Contrôles de concurrence pour limiter les exécutions parallèles

## Bonnes pratiques

- Toujours utiliser GitHub Secrets pour les clés API (jamais hardcodées)
- Créer un CLAUDE.md pour définir les normes de projet
- Examiner les suggestions Claude avant de merger
- Limiter les permissions de l'action au minimum nécessaire
