# Claude Code sur Google Vertex AI

Source: https://code.claude.com/docs/fr/google-vertex-ai

## Démarrage rapide (assistant `/setup-vertex`)

```bash
claude   # → "plateforme tierce" → "Google Vertex AI"
```
L'assistant détecte votre projet GCP, vérifie les modèles disponibles, épingle les versions.

Nécessite Claude Code v2.1.98+. Rouvrir l'assistant : `/setup-vertex`

## Prérequis

- Compte GCP avec facturation activée
- API Vertex AI activée (`gcloud services enable aiplatform.googleapis.com`)
- Accès aux modèles Claude dans Model Garden (peut prendre 24-48h)
- Google Cloud SDK installé et configuré

## Configuration manuelle

### Activer l'API Vertex AI

```bash
gcloud config set project YOUR-PROJECT-ID
gcloud services enable aiplatform.googleapis.com
```

### Demander l'accès aux modèles

Dans [Vertex AI Model Garden](https://console.cloud.google.com/vertex-ai/model-garden) → rechercher "Claude" → demander l'accès.

### Configurer les identifiants

```bash
gcloud auth application-default login
```

Note : Claude Code v2.1.121+ supporte Workload Identity Federation basée sur certificat X.509 via `GOOGLE_APPLICATION_CREDENTIALS`.

### Activer Vertex AI dans Claude Code

```bash
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=global      # ou us-east5, europe-west1, etc.
export ANTHROPIC_VERTEX_PROJECT_ID=YOUR-PROJECT-ID
```

Variables optionnelles :
```bash
# Remplacer l'URL du point de terminaison
export ANTHROPIC_VERTEX_BASE_URL=https://aiplatform.googleapis.com

# Désactiver la mise en cache
export DISABLE_PROMPT_CACHING=1

# TTL de cache 1h
export ENABLE_PROMPT_CACHING_1H=1

# Région spécifique pour certains modèles
export VERTEX_REGION_CLAUDE_HAIKU_4_5=us-east5
export VERTEX_REGION_CLAUDE_4_6_SONNET=europe-west1
```

Note : `/logout` indisponible avec Vertex (authentification Google Cloud gère).

### Épingler les versions de modèle (important)

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5@20251001'
```

Défauts sans épinglage :
- Principal : `claude-sonnet-4-5@20250929`
- Petit/rapide : identique au principal

### Actualisation automatique des identifiants

```json
{
  "gcpAuthRefresh": "gcloud auth application-default login",
  "env": {"ANTHROPIC_VERTEX_PROJECT_ID": "your-project-id"}
}
```

## Régions et points de terminaison

Support : `global`, multi-régions (`eu`, `us`), ou régions spécifiques (`us-east5`).

Note importante : la disponibilité des modèles varie par région. Certains modèles supportent seulement les points de terminaison globaux.

## Configuration IAM

Rôle requis : `roles/aiplatform.user`
- `aiplatform.endpoints.predict` : invocation de modèle et comptage des jetons

## MCP Tool Search sur Vertex

Désactivé par défaut sur Vertex. Activer pour Sonnet 4.5+ et Opus 4.5+ :
```bash
export ENABLE_TOOL_SEARCH=true
```

Note : les modèles antérieurs sur Vertex AI ne supportent pas l'en-tête bêta requis.

## Contexte 1M tokens

Ajouter `[1m]` à l'ID du modèle. L'assistant de configuration propose l'option directement.

## Dépannage

**"Could not load default credentials"** :
- `gcloud auth application-default login`
- Ou définir `GOOGLE_APPLICATION_CREDENTIALS`

**Erreur 404 "modèle non trouvé"** :
- Confirmer que le modèle est activé dans Model Garden
- Vérifier disponibilité dans la région spécifiée
- Pour `global` : vérifier support dans Model Garden sous "Fonctionnalités prises en charge"

**Erreur 429** :
- Basculer vers `CLOUD_ML_REGION=global` pour meilleure disponibilité
