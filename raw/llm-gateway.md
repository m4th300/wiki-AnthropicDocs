# Configuration de la passerelle LLM

Source: https://code.claude.com/docs/fr/llm-gateway

## Exigences pour une passerelle LLM

La passerelle doit exposer au moins un des formats API :

1. **Anthropic Messages** : `/v1/messages`, `/v1/messages/count_tokens`
   - Doit transférer les en-têtes : `anthropic-beta`, `anthropic-version`

2. **Bedrock InvokeModel** : `/invoke`, `/invoke-with-response-stream`
   - Doit préserver les champs du corps : `anthropic_beta`, `anthropic_version`

3. **Vertex rawPredict** : `:rawPredict`, `:streamRawPredict`, `/count-tokens:rawPredict`
   - Doit transférer : `anthropic-beta`, `anthropic-version`

Note : lors d'utilisation du format Anthropic Messages avec Bedrock ou Vertex → définir `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`.

## En-têtes de requête Claude Code

| En-tête | Description |
|---------|-------------|
| `X-Claude-Code-Session-Id` | ID de session unique |
| `X-Claude-Code-Agent-Id` | ID du sous-agent ou coéquipier |
| `X-Claude-Code-Parent-Agent-Id` | ID de l'agent parent |

Pour omettre le bloc d'attribution : `CLAUDE_CODE_ATTRIBUTION_HEADER=0`

## Découverte des modèles via passerelle

Activer : `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1` (nécessite v2.1.129+)

Claude Code interroge `/v1/models` de la passerelle au démarrage et ajoute les modèles au sélecteur `/model`. Seulement les modèles dont l'ID commence par `claude` ou `anthropic` sont ajoutés.

## Configuration LiteLLM

⚠️ Les versions LiteLLM 1.82.7 et 1.82.8 ont été compromises avec un malware. Éviter ces versions.

### Point de terminaison unifié (recommandé)

```bash
export ANTHROPIC_BASE_URL=https://litellm-server:4000
```

### Authentification

**Clé API statique** :
```bash
export ANTHROPIC_AUTH_TOKEN=sk-litellm-static-key
```

**Clé API dynamique (script)** :
```json
{
  "apiKeyHelper": "~/bin/get-litellm-key.sh",
  "env": {"CLAUDE_CODE_API_KEY_HELPER_TTL_MS": "3600000"}
}
```

### Transmission directe par fournisseur

**API Claude via LiteLLM** :
```bash
export ANTHROPIC_BASE_URL=https://litellm-server:4000/anthropic
```

**Amazon Bedrock via LiteLLM** :
```bash
export ANTHROPIC_BEDROCK_BASE_URL=https://litellm-server:4000/bedrock
export CLAUDE_CODE_SKIP_BEDROCK_AUTH=1
export CLAUDE_CODE_USE_BEDROCK=1
```

**Google Vertex AI via LiteLLM** :
```bash
export ANTHROPIC_VERTEX_BASE_URL=https://litellm-server:4000/vertex_ai/v1
export ANTHROPIC_VERTEX_PROJECT_ID=your-gcp-project-id
export CLAUDE_CODE_SKIP_VERTEX_AUTH=1
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=us-east5
```

**Claude Platform sur AWS via passerelle** :
```bash
export ANTHROPIC_AWS_BASE_URL=https://litellm-server:4000/anthropic-aws
export ANTHROPIC_AWS_WORKSPACE_ID=wrkspc_01ABCDEFGHIJKLMN
export CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH=1
export CLAUDE_CODE_USE_ANTHROPIC_AWS=1
```
