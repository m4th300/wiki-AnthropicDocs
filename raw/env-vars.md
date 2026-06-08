# Variables d'environnement - Complete Reference

## Documentation Index
Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt

---

## Overview
Variables d'environnement pour contrôler le comportement de Claude Code, incluant la sélection du modèle, l'authentification, le routage des requêtes et les bascules de fonctionnalités.

Beaucoup des mêmes comportements peuvent être configurés via:
- Fichier de paramètres (`settings.json`)
- Drapeau CLI
- Commande en session comme `/model`

---

## Setting Environment Variables

### In Your Shell

#### macOS, Linux, WSL
```bash
export API_TIMEOUT_MS="1200000"
claude
```

Pour la définir pour chaque session, ajoutez la ligne `export` à `~/.bashrc`, `~/.zshrc` ou le fichier de profil de votre shell.

#### Windows PowerShell
```powershell
$env:API_TIMEOUT_MS = "1200000"
claude
```

Pour la définir pour chaque session :
```powershell
[Environment]::SetEnvironmentVariable("API_TIMEOUT_MS", "1200000", "User")
```
Puis ouvrez un nouveau terminal.

#### Windows CMD
```batch
set API_TIMEOUT_MS=1200000
claude
```

Pour la définir pour chaque session : `setx API_TIMEOUT_MS "1200000"` et ouvrez un nouveau terminal.

### In Settings Files

Ajoutez les variables sous la clé `env` dans un fichier `settings.json`:

```json
{
  "env": {
    "API_TIMEOUT_MS": "1200000",
    "BASH_DEFAULT_TIMEOUT_MS": "300000"
  }
}
```

#### Settings Files Scope

| Fichier | S'applique à |
| :--- | :--- |
| `~/.claude/settings.json` | Vous, dans chaque projet |
| `.claude/settings.json` | Tous ceux qui travaillent dans le projet, archivé dans le contrôle de source |
| `.claude/settings.local.json` | Vous, dans ce projet uniquement, non archivé |
| Paramètres gérés | Tous les membres de votre organisation, déployés par un administrateur |

---

## Precedence Rules

- **Variable d'environnement** > Paramètre de configuration
- `ANTHROPIC_MODEL` remplace le paramètre `model`
- `CLAUDE_CODE_AUTO_CONNECT_IDE` remplace `autoConnectIde`
- `--model` et `/model` remplacent `ANTHROPIC_MODEL`
- `CLAUDE_CODE_EFFORT_LEVEL` remplace `/effort`

Claude Code lit les variables au démarrage; les modifications prennent effet au prochain lancement.

---

## Complete Variables List

### Authentication & API Configuration

| Variable | Purpose |
| :--- | :--- |
| `ANTHROPIC_API_KEY` | Clé API envoyée en tant qu'en-tête `X-Api-Key`. Utilisée à la place de votre abonnement Claude Pro, Max, Team ou Enterprise. En mode non interactif (`-p`), la clé est toujours utilisée. Pour utiliser votre abonnement à la place: `unset ANTHROPIC_API_KEY` |
| `ANTHROPIC_AUTH_TOKEN` | Valeur personnalisée pour l'en-tête `Authorization` (préfixée par `Bearer `) |
| `ANTHROPIC_WORKSPACE_ID` | ID d'espace de travail pour la fédération d'identité de charge de travail |

### AWS & Bedrock

| Variable | Purpose |
| :--- | :--- |
| `ANTHROPIC_AWS_API_KEY` | Clé API de l'espace de travail pour Claude Platform on AWS. Envoyée comme `x-api-key`, prend priorité sur AWS SigV4 |
| `ANTHROPIC_AWS_BASE_URL` | Remplacer l'URL du point de terminaison Claude Platform on AWS. Par défaut: `https://aws-external-anthropic.{AWS_REGION}.api.aws` |
| `ANTHROPIC_AWS_WORKSPACE_ID` | Requis pour Claude Platform on AWS. Envoyé comme en-tête `anthropic-workspace-id` |
| `ANTHROPIC_BEDROCK_BASE_URL` | Remplacer l'URL du point de terminaison Bedrock. |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | Remplacer l'URL du point de terminaison Bedrock Mantle |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Bedrock niveau de service: `default`, `flex` ou `priority`. Envoyé comme en-tête `X-Amzn-Bedrock-Service-Tier` |
| `AWS_BEARER_TOKEN_BEDROCK` | Clé API Bedrock pour l'authentification |

### Google Vertex AI

| Variable | Purpose |
| :--- | :--- |
| `ANTHROPIC_VERTEX_BASE_URL` | Remplacer l'URL du point de terminaison Vertex AI |
| `ANTHROPIC_VERTEX_PROJECT_ID` | ID de projet GCP pour requêtes Vertex AI |

### Microsoft Foundry

| Variable | Purpose |
| :--- | :--- |
| `ANTHROPIC_FOUNDRY_API_KEY` | Clé API pour authentification Microsoft Foundry |
| `ANTHROPIC_FOUNDRY_BASE_URL` | URL de base complète pour la ressource Foundry (ex: `https://my-resource.services.ai.azure.com/anthropic`) |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Nom de la ressource Foundry (ex: `my-resource`). Requis si `ANTHROPIC_FOUNDRY_BASE_URL` n'est pas défini |

### API Routing & Proxies

| Variable | Purpose |
| :--- | :--- |
| `ANTHROPIC_BASE_URL` | Remplacer le point de terminaison de l'API pour acheminer via proxy ou passerelle. Quand défini sur hôte non-first-party, la recherche d'outils MCP est désactivée par défaut |
| `ANTHROPIC_CUSTOM_HEADERS` | En-têtes personnalisés à ajouter aux requêtes (format `Name: Value`, séparés par sauts de ligne pour multiples) |
| `ANTHROPIC_BETAS` | Liste séparée par virgules de valeurs d'en-tête `anthropic-beta` supplémentaires |
| `CLAUDE_CODE_EXTRA_BODY` | Objet JSON à fusionner au niveau supérieur de chaque corps de requête API |

### Model Configuration

| Variable | Purpose |
| :--- | :--- |
| `ANTHROPIC_MODEL` | Nom du paramètre de modèle à utiliser |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Modèle Haiku par défaut |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Modèle Sonnet par défaut |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Modèle Opus par défaut |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | Nom d'affichage pour le modèle Haiku |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | Description pour le modèle Haiku |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | Capacités supportées par le modèle Haiku |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | Nom d'affichage pour le modèle Sonnet |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | Description pour le modèle Sonnet |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | Capacités supportées par le modèle Sonnet |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | Nom d'affichage pour le modèle Opus |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | Description pour le modèle Opus |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | Capacités supportées par le modèle Opus |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | ID de modèle à ajouter comme entrée personnalisée dans sélecteur `/model` |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | Nom d'affichage pour l'entrée de modèle personnalisé |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | Description pour l'entrée de modèle personnalisé |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_SUPPORTED_CAPABILITIES` | Capacités pour l'entrée de modèle personnalisé |
| `ANTHROPIC_SMALL_FAST_MODEL` | [DÉPRÉCIÉ] Nom du modèle de classe Haiku pour tâches en arrière-plan |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | Remplacer la région AWS pour le modèle de classe Haiku sur Bedrock |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | Définissez sur `1` pour empêcher le remappage automatique d'Opus 4.0 et 4.1 |

### Effort & Thinking Configuration

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_EFFORT_LEVEL` | Niveau d'effort: `low`, `medium`, `high`, `xhigh`, `max` ou `auto`. Prend priorité sur `/effort` et paramètre `effortLevel` |
| `CLAUDE_CODE_DISABLE_THINKING` | Définissez sur `1` pour forcer la désactivation de la réflexion étendue |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | Définissez sur `1` pour désactiver le raisonnement adaptatif sur Opus 4.6 et Sonnet 4.6 |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | Remplacer la limite de tokens par défaut pour les lectures de fichiers |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS` | Remplacer la taille de la fenêtre de contexte que Claude Code assume. S'applique seulement quand `DISABLE_COMPACT` est aussi défini |

### Timeouts & Performance

| Variable | Purpose |
| :--- | :--- |
| `API_TIMEOUT_MS` | Délai d'expiration pour requêtes API en millisecondes (défaut: 600 000 ou 10 minutes; maximum: 2 147 483 647) |
| `BASH_DEFAULT_TIMEOUT_MS` | Délai d'expiration par défaut pour commandes bash longues (défaut: 120 000 ou 2 minutes) |
| `BASH_MAX_TIMEOUT_MS` | Délai d'expiration maximal que le modèle peut définir pour commandes bash longues (défaut: 600 000 ou 10 minutes) |
| `BASH_MAX_OUTPUT_LENGTH` | Nombre maximum de caractères dans sorties bash avant que la sortie complète soit enregistrée dans un fichier |

### File Operations

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_GLOB_HIDDEN` | Définissez sur `false` pour exclure les fichiers pointés des résultats Glob |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | Définissez sur `false` pour faire respecter à Glob les modèles `.gitignore` |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Délai d'expiration pour la découverte de fichiers Glob (défaut: 20s, 60s sur WSL) |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | Définissez sur `1` pour désactiver le checkpointing de fichier (la commande `/rewind` ne pourra pas restaurer) |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | Définissez sur `1` pour désactiver le traitement des pièces jointes (`@` envoyé comme texte brut) |

### UI & Display Configuration

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | Définissez sur `1` pour désactiver le rendu en plein écran. Prend priorité sur `CLAUDE_CODE_NO_FLICKER` et paramètre `tui` |
| `CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT` | Définissez sur `1` pour repeindre l'écran entier à chaque image |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL` | Définissez sur `1` pour désactiver le défilement virtuel (affiche chaque message dans la transcription) |
| `CLAUDE_CODE_DISABLE_MOUSE` | Définissez sur `1` pour désactiver le suivi de la souris |
| `CLAUDE_CODE_HIDE_CWD` | Définissez sur `1` pour masquer le répertoire de travail dans le logo de démarrage |
| `CLAUDE_CODE_ACCESSIBILITY` | Définissez sur `1` pour garder le curseur du terminal natif visible |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | Définissez sur `1` pour désactiver les mises à jour automatiques du titre du terminal |
| `CLAUDE_CODE_FORCE_SYNC_OUTPUT` | Définissez sur `1` pour forcer l'activation du mode privé DEC 2026 synchronized output |

### IDE Integration

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | Remplacer la connexion IDE automatique. `false` pour empêcher, `true` pour forcer |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | Ignorer l'installation automatique des extensions IDE |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | Définissez sur `1` pour ignorer la validation des entrées du fichier de verrouillage IDE |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | Remplacer l'adresse d'hôte utilisée pour se connecter à l'extension IDE |

### Memory & Context Management

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | `1` pour désactiver la mémoire automatique, `0` pour la forcer même en mode `--bare` |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | Définissez sur `1` pour empêcher le chargement de tous les fichiers CLAUDE.md |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | Définissez sur `1` pour charger les fichiers de mémoire depuis les répertoires spécifiés avec `--add-dir` |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | Capacité de contexte en tokens utilisée pour les calculs de compactage automatique. Par défaut: fenêtre de contexte du modèle (200K ou 1M) |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Pourcentage de capacité de contexte (1-100) auquel le compactage automatique se déclenche. Par défaut ~95% |

### Git & Workflows

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | Définissez sur `1` pour supprimer les instructions de flux de travail de commit et PR intégrées |
| `CLAUDE_CODE_DISABLE_WORKFLOWS` | Définissez sur `1` pour désactiver les workflows |

### Agent & Background Tasks

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW` | Définissez sur `1` pour désactiver les agents en arrière-plan et la vue des agents |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Définissez sur `1` pour désactiver toutes les fonctionnalités de tâche en arrière-plan |
| `CLAUDE_CODE_AUTO_BACKGROUND_TASKS` | Définissez sur `1` pour forcer l'activation de la mise en arrière-plan automatique des tâches |
| `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` | Délai d'expiration de blocage pour les subagents en arrière-plan. Par défaut `600000` (10 minutes) |
| `CLAUDE_CODE_FORK_SUBAGENT` | Définissez sur `1` pour que les subagents forked soient le défaut |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | Définissez sur `1` pour désactiver tous les types de subagent intégrés (mode non interactif `-p` seulement) |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | Définissez sur `1` pour ignorer le préfixe `mcp__<server>__` sur les noms d'outils MCP |

### Tasks & Scheduling

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_DISABLE_CRON` | Définissez sur `1` pour désactiver les tâches planifiées |
| `CLAUDE_CODE_ENABLE_TASKS` | Contrôle si les sessions utilisent les outils Task structurés ou l'outil `TodoWrite` hérité. Définissez sur `0` pour revenir à `TodoWrite` |

### Telemetry & Monitoring

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | Définissez sur `1` pour activer la collecte de données OpenTelemetry |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Équivalent de définir `DISABLE_AUTOUPDATER`, `DISABLE_FEEDBACK_COMMAND`, `DISABLE_ERROR_REPORTING` et `DISABLE_TELEMETRY` |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | Définissez sur `1` pour désactiver les sondages de qualité de session |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` | Définissez sur `1` pour acheminer le sondage vers votre collecteur OpenTelemetry |

### Debugging

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | Remplacer le chemin d'accès au fichier journal de débogage (chemin de fichier, pas répertoire). Nécessite `--debug` ou variable `DEBUG` pour activer la journalisation |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | Niveau de journal minimum: `verbose`, `debug` (par défaut), `info`, `warn`, `error` |
| `CLAUDECODE` | Défini sur `1` dans les sous-processus générés par Claude Code (outils Bash, sessions tmux, commandes hook, etc.) |

### Advanced & Experimental

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | Définissez sur `1` pour désactiver le mode rapide |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | `0` pour forcer les récapitulatifs désactivés, `1` pour forcer activés. Prend priorité sur `/config` |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | Définissez sur `false` pour désactiver les suggestions d'invite |
| `CLAUDE_CODE_ENABLE_AUTO_MODE` | Définissez sur `1` pour rendre le mode auto disponible sur Bedrock, Vertex AI et Foundry. Nécessite v2.1.158+ |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | Définissez sur `1` pour remplir le sélecteur `/model` depuis `/v1/models` de votre passerelle |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | Définissez sur `1` pour désactiver la prise en charge de la fenêtre de contexte 1M |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | Définissez sur `1` pour ignorer l'ajout automatique de la place de marché officielle |
| `CLAUDE_CODE_DISABLE_POLICY_SKILLS` | Définissez sur `1` pour ignorer le chargement des skills depuis le répertoire des skills gérés au niveau du système |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | Contrôle si les entrées d'appel d'outil se transmettent en continu. Activé par défaut sur l'API Anthropic. `0` pour refuser, `1` pour forcer |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | Définissez sur `1` pour supprimer les en-têtes `anthropic-beta` spécifiques et les champs de schéma d'outil bêta |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | Définissez sur `1` pour désactiver le secours non-streaming quand une requête streaming échoue |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | Définissez sur `0` pour omettre le bloc d'attribution du début de l'invite système |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | Définissez sur `1` pour activer les équipes d'agents (expérimental, désactivé par défaut) |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | Temps en ms à attendre après que la boucle de requête devienne inactive avant de quitter |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | Intervalle en ms auquel les identifiants doivent être actualisés (lors de l'utilisation de `apiKeyHelper`) |

### Platform-Specific

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_GIT_BASH_PATH` | Windows uniquement: chemin vers l'exécutable Git Bash (`bash.exe`) |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Retourner au répertoire de travail d'origine après chaque commande Bash ou PowerShell |
| `CCR_FORCE_BUNDLE` | Définissez sur `1` pour forcer `claude --remote` à regrouper et télécharger votre référentiel local |

### TLS & Certificates

| Variable | Purpose |
| :--- | :--- |
| `CLAUDE_CODE_CERT_STORE` | Liste séparée par des virgules de sources de certificats CA. `bundled` (Mozilla CA) ou `system`. Par défaut `bundled,system` |
| `CLAUDE_CODE_CLIENT_CERT` | Chemin vers le fichier de certificat client pour l'authentification mTLS |
| `CLAUDE_CODE_CLIENT_KEY` | Chemin vers le fichier de clé privée client pour l'authentification mTLS |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | Phrase de passe pour `CLAUDE_CODE_CLIENT_KEY` chiffré (facultatif) |

---

## Notes

- Claude Code reads environment variables at startup; changes take effect on next launch
- Environment variables from shell are session-specific; use settings files for persistence
- For detailed configuration of model behavior, see [Model Configuration](/fr/model-config)
- For settings file precedence and global configuration, see [Settings](/fr/settings)
