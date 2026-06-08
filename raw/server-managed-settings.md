# Paramètres gérés par le serveur

Source: https://code.claude.com/docs/fr/server-managed-settings

## Ce que c'est

Configuration centralisée via une interface web sur Claude.ai.
Les clients Claude Code reçoivent ces paramètres automatiquement à l'authentification.

Disponible pour : Claude for Teams et Claude for Enterprise.
Nécessite : Claude Code v2.1.38+ (Teams) ou v2.1.30+ (Enterprise) + accès à `api.anthropic.com`.

## Quand utiliser vs Paramètres gérés par point de terminaison

| Approche | Idéal pour |
|---------|-----------|
| **Gérés par le serveur** (cette page) | Organisations sans MDM, utilisateurs sur appareils non gérés |
| **Gérés par point de terminaison** (plist/registre/fichier) | Organisations avec MDM ou gestion des points de terminaison |

## Configurer

1. claude.ai → **Admin Settings > Claude Code > Managed settings**
2. Ajouter la configuration JSON (tout paramètre disponible dans settings.json, sauf quelques exceptions)
3. Enregistrer → les clients reçoivent les paramètres au prochain démarrage

### Exemples

```json
{
  "permissions": {
    "deny": ["Bash(curl *)", "Read(./.env)"],
    "disableBypassPermissionsMode": "disable"
  },
  "allowManagedPermissionRulesOnly": true
}
```

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{"type": "command", "command": "/usr/local/bin/audit-edit.sh"}]
    }]
  }
}
```

```json
{
  "autoMode": {
    "environment": [
      "Source control: github.example.com/acme-corp",
      "Trusted cloud buckets: s3://acme-build-artifacts"
    ]
  }
}
```

## Précédence des paramètres

Paramètres gérés par serveur = priorité la plus haute (mais ne fusionnent PAS avec les paramètres gérés par point de terminaison).
La PREMIÈRE source non vide gagne : serveur > point de terminaison.

## Comportement de récupération

- Récupération au démarrage + interrogation toutes les heures
- Brève fenêtre non appliquée au premier lancement si pas de cache
- Paramètres en cache appliqués immédiatement aux démarrages suivants

## Appliquer un démarrage fermé par défaut (fail-closed)

```json
{ "forceRemoteSettingsRefresh": true }
```

Si la récupération échoue → l'interface se ferme plutôt que de continuer. Utile pour environnements haute sécurité.

## Boîtes de dialogue d'approbation de sécurité

Certains paramètres (commandes shell, variables d'env, hooks) → boîte de dialogue d'approbation utilisateur requise.
En mode non-interactif (`-p`) → ignorées, paramètres appliqués automatiquement.

## Limitations actuelles

- Paramètres uniformes pour tous les utilisateurs (pas de configuration par groupe)
- `managed-mcp.json` ne peut pas être distribué via paramètres gérés par serveur (utiliser `allowedMcpServers`/`deniedMcpServers`)
- Certains paramètres limités aux sources plist/registre (`policyHelper`, `wslInheritsWindowsSettings`) ne sont pas respectés

## Disponibilité

PAS disponible avec : Bedrock, Vertex AI, Foundry, points de terminaison `ANTHROPIC_BASE_URL` personnalisés.

## Journalisation d'audit

Événements d'audit disponibles via l'API de conformité ou export du journal d'audit.
