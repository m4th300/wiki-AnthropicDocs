# Claude Code GitLab CI/CD

Source: https://code.claude.com/docs/fr/gitlab-ci-cd

Note : En bêta. Maintenu par GitLab.

## Vue d'ensemble

Intégration de Claude Code dans les pipelines GitLab CI/CD. Basé sur le Claude Code CLI et Agent SDK.

## Configuration rapide

```yaml
stages:
  - ai

claude:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  variables:
    GIT_STRATEGY: fetch
  before_script:
    - apk update && apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
  script:
    - /bin/gitlab-mcp-server || true
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Review this MR and implement the requested changes'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
```

Ajouter `ANTHROPIC_API_KEY` comme variable CI/CD masquée.

## Variables d'entrée

- `AI_FLOW_INPUT` : le prompt/instruction
- `AI_FLOW_CONTEXT` : contexte supplémentaire (ex: URL de la MR)
- `AI_FLOW_EVENT` : type d'événement déclencheur

## Fournisseurs cloud

### Amazon Bedrock (OIDC)

Prérequis :
- Compte AWS avec Bedrock activé
- GitLab configuré comme fournisseur OIDC dans AWS IAM
- Rôle IAM avec permissions Bedrock

Variables CI/CD : `AWS_ROLE_TO_ASSUME`, `AWS_REGION`

```yaml
before_script:
  - ...
  - aws sts assume-role-with-web-identity ...
  - export AWS_ACCESS_KEY_ID="..." AWS_SECRET_ACCESS_KEY="..." AWS_SESSION_TOKEN="..."
script:
  - claude -p "${AI_FLOW_INPUT}" --permission-mode acceptEdits ...
variables:
  AWS_REGION: "us-west-2"
```

### Google Vertex AI (Workload Identity Federation)

Prérequis :
- Projet GCP avec API Vertex AI activée
- Workload Identity Federation configurée
- Compte de service avec permissions Vertex AI

Variables CI/CD : `GCP_WORKLOAD_IDENTITY_PROVIDER`, `GCP_SERVICE_ACCOUNT`, `CLOUD_ML_REGION`

## Cas d'usage

```text
@claude implement this feature based on the issue description
@claude suggest a concrete approach to cache the results of this API call
@claude fix the TypeError in the user dashboard component
```

## Bonnes pratiques

- CLAUDE.md à la racine pour les conventions de projet
- Ne jamais committer les clés API (utiliser les variables CI/CD masquées)
- Utiliser OIDC/WIF plutôt que des clés statiques
- Définir des délais d'expiration de tâche raisonnables

## Sécurité

- Chaque tâche s'exécute dans un conteneur isolé
- Modifications passent par des MR (examinateurs voient chaque diff)
- Les règles de protection de branche s'appliquent
- Accès réseau restreint

## Dépannage

**Claude ne répond pas aux @claude** : vérifier le déclenchement du pipeline + variables CI/CD.
**Impossible d'écrire des commentaires** : vérifier les permissions de `CI_JOB_TOKEN` + outil `mcp__gitlab` activé.
**Erreurs d'authentification** : vérifier la configuration OIDC/WIF pour Bedrock/Vertex.
