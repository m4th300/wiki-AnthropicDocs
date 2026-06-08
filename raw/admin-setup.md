# Configurer Claude Code pour votre organisation

Source: https://code.claude.com/docs/fr/admin-setup

## Carte de décision pour les administrateurs

| Décision | Référence |
|---------|---------|
| Choisir le fournisseur d'API | Authentification, Bedrock, Vertex AI, Foundry |
| Livraison des paramètres gérés | Paramètres gérés par serveur, Fichiers de paramètres |
| Ce qu'il faut appliquer | Permissions, Sandboxing |
| Visibilité de l'utilisation | Analytique, Surveillance, Coûts |
| Gestion des données | Utilisation des données, Sécurité |

## Choisir le fournisseur d'API

| Fournisseur | Choisir quand |
|------------|-------------|
| Claude for Teams/Enterprise | Recommandation par défaut - sous un seul abonnement par siège |
| Claude Console | Orienté API, facturation à l'usage |
| Amazon Bedrock | Contrôles de conformité et facturation AWS existants |
| Google Vertex AI | Contrôles de conformité et facturation GCP existants |
| Microsoft Foundry | Contrôles de conformité et facturation Azure existants |

Note : Claude Code sur le web, Routines, Code Review, Remote Control, Chrome → nécessitent un compte Claude.ai (pas disponibles via clés API Console ou credentials cloud seuls).

## Comment les paramètres atteignent les appareils

| Mécanisme | Livraison | Priorité | Plateformes |
|---------|---------|---------|-----------|
| Géré par serveur | Console d'administration claude.ai | Très élevée | Tous |
| Politique plist/registre | macOS: plist, Windows: HKLM | Élevée | macOS, Windows |
| Géré basé sur fichier | `/Library/Application Support/ClaudeCode/managed-settings.json`, `/etc/claude-code/managed-settings.json`, `C:\Program Files\ClaudeCode\managed-settings.json` | Moyenne | Tous |
| Registre utilisateur Windows | HKCU | Très basse | Windows uniquement |

Paramètres gérés par serveur = nécessite Claude for Teams/Enterprise.
Pour WSL héritant des paramètres Windows : `wslInheritsWindowsSettings: true`.

## Ce qu'il faut appliquer

| Contrôle | Paramètres clés |
|---------|----------------|
| Règles de permission | `permissions.allow`, `permissions.deny` |
| Verrouillage des permissions | `allowManagedPermissionRulesOnly`, `permissions.disableBypassPermissionsMode` |
| Sandboxing | `sandbox.enabled`, `sandbox.network.allowedDomains` |
| CLAUDE.md de politique | Fichier au chemin de la politique gérée |
| Contrôle serveur MCP | `allowedMcpServers`, `deniedMcpServers`, `allowManagedMcpServersOnly` |
| Contrôle marketplace plugins | `strictKnownMarketplaces`, `blockedMarketplaces` |
| Verrouillage personnalisation | `strictPluginOnlyCustomization` |
| Restrictions hooks | `allowManagedHooksOnly`, `allowedHttpHookUrls` |
| Désactiver agent view | `disableAgentView` |
| Version minimale | `minimumVersion` |

Note : Refuser WebFetch bloque l'outil fetch de Claude, mais si Bash est autorisé, curl/wget peuvent toujours accéder à n'importe quelle URL. Sandboxing ferme cette lacune.

## Visibilité de l'utilisation

| Capacité | Disponibilité | Par où commencer |
|---------|-------------|-----------------|
| Surveillance OpenTelemetry | Tous les fournisseurs | `/fr/monitoring-usage` |
| Tableau de bord analytique | Anthropic uniquement | `/fr/analytics` |
| Suivi des coûts | Anthropic uniquement | `/fr/costs` |

## Gestion des données

| Sujet | Reference |
|-------|---------|
| Politique d'utilisation des données | `/fr/data-usage` |
| Zero Data Retention | `/fr/zero-data-retention` |
| Architecture de sécurité | `/fr/security` |

## Vérifier la configuration

Demander à un développeur d'exécuter `/status` → chercher la ligne "Enterprise managed settings" avec la source entre parenthèses : `(remote)`, `(plist)`, `(HKLM)`, `(HKCU)` ou `(file)`.

## Ressources pour les développeurs

- [Démarrage rapide](/fr/quickstart)
- [Flux de travail courants](/fr/common-workflows)
- [Claude 101](https://anthropic.skilljar.com/claude-101)
- [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)

## Dépannage courant

- Boucles de connexion : `/logout` puis `/login`
- Option d'auth enterprise manquante : `claude update`
- "You haven't been added to your organization yet" : mettre à jour le siège dans la console d'administration
