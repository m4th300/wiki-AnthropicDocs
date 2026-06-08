# Claude Code sur Amazon Bedrock

Source: https://code.claude.com/docs/fr/amazon-bedrock

## Démarrage rapide (assistant `/setup-bedrock`)

```bash
claude   # → "3rd-party platform" → "Amazon Bedrock"
```
L'assistant détecte vos profils AWS, vérifie les modèles disponibles, et épingle les versions. Résultat sauvegardé dans `~/.claude/settings.json`.

Rouvrir l'assistant : `/setup-bedrock`

## Prérequis

- Compte AWS avec Bedrock activé
- Accès aux modèles Claude Anthropic activé (formulaire de cas d'usage soumis une fois)
- AWS CLI (optionnel)
- Permissions IAM appropriées

## Configuration manuelle

### 1. Soumettre le formulaire de cas d'usage

Dans la console Bedrock → catalogue de modèles → sélectionner un modèle Anthropic → soumettre formulaire.
Une fois par compte AWS. Via API pour AWS Organizations : `PutUseCaseForModelAccess`.

### 2. Configurer les identifiants AWS

Option A : `aws configure`
Option B : Variables d'environnement (clé d'accès)
```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```
Option C : Profil SSO
```bash
aws sso login --profile=<profile>
export AWS_PROFILE=your-profile-name
```
Option D : `aws login`
Option E : Clés API Bedrock
```bash
export AWS_BEARER_TOKEN_BEDROCK=your-bedrock-api-key
```

### 3. Activer Bedrock dans Claude Code

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1  # requis
```

Notes :
- `AWS_REGION` est requis (ne lit pas `.aws/config`)
- `/logout` indisponible avec Bedrock (authentification AWS gère)
- WebSearch non disponible sur Bedrock

### 4. Épingler les versions de modèle (important pour déploiements)

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='us.anthropic.claude-opus-4-8'
export ANTHROPIC_DEFAULT_SONNET_MODEL='us.anthropic.claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'
```

Modèles par défaut sans épinglage :
- Principal : `us.anthropic.claude-sonnet-4-5-20250929-v1:0`
- Petit/rapide : identique au principal (Haiku peut ne pas être activé partout)

### Actualisation automatique des identifiants

```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "env": {"AWS_PROFILE": "myprofile"}
}
```

Alternative pour identifiants inter-comptes (exécuté à chaque rechargement) :
```json
{
  "awsCredentialExport": "script-returning-json-credentials"
}
```

Sortie attendue de `awsCredentialExport` :
```json
{"Credentials": {"AccessKeyId": "...", "SecretAccessKey": "...", "SessionToken": "..."}}
```

## Politique IAM requise

```json
{
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "bedrock:InvokeModel",
      "bedrock:InvokeModelWithResponseStream",
      "bedrock:ListInferenceProfiles",
      "bedrock:GetInferenceProfile"
    ],
    "Resource": ["arn:aws:bedrock:*:*:inference-profile/*", "...foundation-model/*"]
  }]
}
```

`bedrock:GetInferenceProfile` permet à Claude Code de résoudre les ARN d'inference profile.

## Contexte 1M tokens sur Bedrock

Ajouter `[1m]` à l'ID du modèle :
```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8[1m]'
```

Note : disponibilité varie selon la région.

## Niveaux de service

```bash
export ANTHROPIC_BEDROCK_SERVICE_TIER=priority  # default, flex, priority
```

## Garde-fous AWS

```json
{
  "env": {
    "ANTHROPIC_CUSTOM_HEADERS": "X-Amzn-Bedrock-GuardrailIdentifier: your-id\nX-Amzn-Bedrock-GuardrailVersion: 1"
  }
}
```

## Point de terminaison Mantle

Bedrock Mantle = point de terminaison Bedrock utilisant la forme API Anthropic native.
```bash
export CLAUDE_CODE_USE_MANTLE=1
export AWS_REGION=us-east-1
```

ID de modèles Mantle : préfixe `anthropic.` (ex: `anthropic.claude-haiku-4-5`)

Pour utiliser Mantle et Invoke en parallèle :
```bash
export CLAUDE_CODE_USE_BEDROCK=1
export CLAUDE_CODE_USE_MANTLE=1
```

```json
{ "availableModels": ["opus", "sonnet", "haiku", "anthropic.claude-haiku-4-5"] }
```

## Mapper les versions aux profiles d'inférence

```json
{
  "modelOverrides": {
    "claude-opus-4-7": "arn:aws:bedrock:us-east-2:123:application-inference-profile/opus-47-prod",
    "claude-opus-4-6": "arn:aws:bedrock:us-east-2:123:application-inference-profile/opus-46-prod"
  }
}
```

## Vérifications au démarrage

Claude Code v2.1.94+ vérifie les modèles accessibles au démarrage.
- Si version plus ancienne épinglée et plus récente disponible → invite à mettre à jour
- Si version par défaut non disponible → revient à version précédente pour cette session

## Prompt caching sur Bedrock

Disponibilité varie selon le modèle et la région.
- TTL par défaut : 5 minutes
- TTL 1 heure : `ENABLE_PROMPT_CACHING_1H=1`

## Dépannage

**Boucle d'authentification SSO** : supprimer `awsAuthRefresh` si VPN d'entreprise interfère.
**Problèmes de région** : `aws bedrock list-inference-profiles --region your-region`
**Erreur "on-demand throughput isn't supported"** : utiliser un ID de profil d'inférence.
**Erreur Mantle 400** : les profils d'inférence comme `us.anthropic.claude-*` ne sont pas valides sur Mantle.
