# Authentification

Source: https://code.claude.com/docs/fr/authentication

## Méthodes d'authentification

### Pour les individus
- Abonnement Claude Pro ou Max (connexion claude.ai)
- Claude for Teams ou Enterprise (compte claude.ai invité par l'admin)
- Claude Console (identifiants Console)
- Fournisseurs cloud (Bedrock, Vertex AI, Foundry) → variables d'environnement uniquement

### Pour les équipes
- **Claude for Teams** : libre-service, collaboration, facturation centralisée. Idéal pour petites équipes.
- **Claude for Enterprise** : ajoute SSO, capture de domaine, permissions basées sur les rôles, politiques gérées.
- **Claude Console** : facturation API → inviter via Settings → Members → Invite + rôle (Claude Code ou Developer)
- **Fournisseurs cloud** : Bedrock, Vertex AI, Foundry

## Stockage des identifiants

- **macOS** : Keychain macOS chiffré
- **Linux** : `~/.claude/.credentials.json` (mode 0600)
- **Windows** : `%USERPROFILE%\.claude\.credentials.json` (contrôles d'accès du profil utilisateur)
- Si `CLAUDE_CONFIG_DIR` défini : `.credentials.json` dans ce répertoire

## Ordre de priorité d'authentification

1. Identifiants fournisseur cloud (`CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_USE_FOUNDRY`)
2. `ANTHROPIC_AUTH_TOKEN` (en-tête `Authorization: Bearer`, pour passerelles LLM)
3. `ANTHROPIC_API_KEY` (en-tête `X-Api-Key`, pour API Anthropic directe)
4. Sortie de `apiKeyHelper` (script shell pour identifiants dynamiques)
5. `CLAUDE_CODE_OAUTH_TOKEN` (jeton longue durée de `claude setup-token`)
6. Identifiants OAuth d'abonnement de `/login` (défaut pour Pro/Max/Team/Enterprise)

**Important** : si `ANTHROPIC_API_KEY` défini avec un abonnement actif → la clé API prend priorité une fois approuvée. Peut causer des échecs si la clé appartient à une organisation désactivée.

Désactiver : `unset ANTHROPIC_API_KEY`
Vérifier méthode active : `/status`

## `apiKeyHelper`

Script shell exécuté pour obtenir des identifiants dynamiques.
```json
{ "apiKeyHelper": "/path/to/script.sh" }
```
- Appelé après 5 min ou sur HTTP 401
- Personnaliser l'intervalle : `CLAUDE_CODE_API_KEY_HELPER_TTL_MS`
- Si > 10 secondes : avertissement dans la barre d'invite

Note : `apiKeyHelper`, `ANTHROPIC_API_KEY` et `ANTHROPIC_AUTH_TOKEN` s'appliquent **uniquement aux sessions CLI**. Claude Desktop et les sessions distantes utilisent OAuth exclusivement.

## Jeton de longue durée (pour CI/scripts)

```bash
claude setup-token
# Affiche le jeton → copier
export CLAUDE_CODE_OAUTH_TOKEN=your-token
```

- Authentifie avec votre abonnement Claude
- Nécessite Pro, Max, Team ou Enterprise
- Limité à l'inférence uniquement (pas Remote Control)
- Le mode bare (`--bare`) ne lit pas `CLAUDE_CODE_OAUTH_TOKEN` → utiliser `ANTHROPIC_API_KEY` à la place

## Commandes utiles

```bash
claude          # Premier lancement → authentification navigateur
/login          # Se connecter ou changer de compte
/logout         # Se déconnecter
claude auth login  # Alternative si navigateur ne s'ouvre pas
claude setup-token # Générer jeton longue durée
```

Si navigateur affiche un code au lieu de rediriger : coller dans terminal à l'invite "Paste code here if prompted".
Commun dans WSL2, SSH, conteneurs.
