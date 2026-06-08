# Claude Code sur Microsoft Foundry

Source: https://code.claude.com/docs/fr/microsoft-foundry

## Configuration

### 1. Provisionner la ressource Foundry

Dans le [portail Microsoft Foundry](https://ai.azure.com/) :
1. Créer une nouvelle ressource
2. Créer des déploiements pour Claude Opus, Sonnet, Haiku

### 2. Configurer les identifiants Azure

**Option A : Clé API**
```bash
export ANTHROPIC_FOUNDRY_API_KEY=your-azure-api-key
```
Obtenir depuis : ressource Azure → Points de terminaison et clés → Clé API

**Option B : Microsoft Entra ID (DefaultAzureCredential)**
```bash
az login
```
Utilisé automatiquement si `ANTHROPIC_FOUNDRY_API_KEY` n'est pas défini.

Note : `/logout` indisponible avec Foundry.

### 3. Activer Foundry dans Claude Code

```bash
export CLAUDE_CODE_USE_FOUNDRY=1
export ANTHROPIC_FOUNDRY_RESOURCE={resource}
# Ou URL complète :
# export ANTHROPIC_FOUNDRY_BASE_URL=https://{resource}.services.ai.azure.com/anthropic
```

### 4. Épingler les versions de modèle (IMPORTANT)

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5'
```

Sans épinglage → alias se résolvent à la dernière version, peut ne pas être disponible dans votre compte Foundry.
Défauts sans épinglage :
- Principal : `claude-sonnet-4-5` (Foundry n'a pas d'assistant de configuration interactif)

### Prompt caching

Activé automatiquement.
TTL 1h : `ENABLE_PROMPT_CACHING_1H=1`

### 5. Exécuter Claude Code

```bash
claude
```

Pas d'assistant de configuration interactif pour Foundry. Les variables d'environnement sont le seul chemin.

## Configuration Azure RBAC

Rôles requis :
- `Azure AI User` ou `Cognitive Services User` (inclut toutes les permissions nécessaires)

Rôle personnalisé minimal :
```json
{
  "permissions": [
    {"dataActions": ["Microsoft.CognitiveServices/accounts/providers/*"]}
  ]
}
```

## Dépannage

**"ChainedTokenCredential authentication failed"** : configurer Entra ID ou définir `ANTHROPIC_FOUNDRY_API_KEY`.
