# Référence des hooks Claude Code

> Les hooks sont des commandes shell, des points de terminaison HTTP ou des invites LLM définis par l'utilisateur qui s'exécutent automatiquement à des points spécifiques du cycle de vie de Claude Code. Ils permettent l'automatisation des politiques de sécurité, la configuration de l'environnement, la validation et l'intégration avec des systèmes externes.

**Note:** La page `hooks-reference` (https://code.claude.com/docs/fr/hooks-reference) renvoie HTTP 404. Le contenu ci-dessous provient de la page `hooks` (https://code.claude.com/docs/fr/hooks), qui constitue la référence complète des hooks.

## Cycle de vie des hooks

Les hooks se déclenchent à des points spécifiques d'une session Claude Code :

- **Niveau session** (une fois par session) : `SessionStart`, `SessionEnd`
- **Niveau tour** (une fois par invite utilisateur) : `UserPromptSubmit`, `Stop`, `StopFailure`
- **Niveau outil** (pendant la boucle agentique) : `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`

## Structure de configuration

Les hooks sont définis en JSON avec trois niveaux d'imbrication :

```
Événement → Groupe de matchers → Gestionnaire(s) de hook
```

### Emplacements et portée des hooks

| Emplacement | Portée | Partageable |
|-------------|--------|-------------|
| `~/.claude/settings.json` | Tous les projets | Non |
| `.claude/settings.json` | Un seul projet | Oui |
| `.claude/settings.local.json` | Un seul projet | Non |
| Paramètres de politique gérée | À l'échelle de l'organisation | Oui |
| `hooks/hooks.json` du plugin | Quand le plugin est actif | Oui |
| Frontmatter de skill/agent | Pendant la durée de vie du composant | Oui |

## Modèles de matchers

Le champ `matcher` filtre quand les hooks s'exécutent :

| Motif | Type | Exemples |
|-------|------|----------|
| `"*"`, `""`, ou omis | Tout correspondre | Se déclenche à chaque occurrence |
| Lettres, chiffres, `_`, `\|` | Chaîne exacte ou liste | `Bash`, `Edit\|Write` |
| Autres caractères | Regex | `^Notebook`, `mcp__memory__.*` |

### Matchers spécifiques aux événements

| Événement | Correspond à | Exemples |
|-----------|-------------|----------|
| Événements d'outils (`PreToolUse`, `PostToolUse`, etc.) | Nom de l'outil | `Bash`, `mcp__memory__.*` |
| `SessionStart` | Source de session | `startup`, `resume`, `clear`, `compact` |
| `Setup` | Flag CLI | `init`, `maintenance` |
| `SessionEnd` | Raison de sortie | `clear`, `resume`, `logout`, `other` |
| `Notification` | Type de notification | `permission_prompt`, `auth_success` |
| `SubagentStart`/`SubagentStop` | Type d'agent | `general-purpose`, `Explore`, noms personnalisés |
| `ConfigChange` | Source de config | `user_settings`, `project_settings`, `policy_settings` |
| `FileChanged` | Noms de fichiers | `.envrc\|.env` |
| `StopFailure` | Type d'erreur | `rate_limit`, `authentication_failed`, `server_error` |

## Types de gestionnaires de hooks

### 1. Hooks de commande (`type: "command"`)

Exécutent des commandes shell avec entrée JSON sur stdin.

**Champs :**
```json
{
  "type": "command",
  "command": "script.sh",
  "args": ["arg1", "arg2"],
  "timeout": 600,
  "async": false,
  "asyncRewake": false,
  "shell": "bash",
  "statusMessage": "Validation en cours...",
  "if": "Bash(rm *)",
  "once": false
}
```

**Forme exec vs forme shell :**
- **Avec `args`** (Exec) : Exécution directe, sans interprétation shell
- **Sans `args`** (Shell) : Passé au shell avec tokenisation et pipes

```json
{
  "command": "node",
  "args": ["${CLAUDE_PLUGIN_ROOT}/scripts/format.js", "--fix"]
}
```

### 2. Hooks HTTP (`type: "http"`)

Envoient du JSON à un point de terminaison HTTP via POST.

**Champs :**
```json
{
  "type": "http",
  "url": "http://localhost:8080/hooks/pre-tool-use",
  "timeout": 600,
  "headers": {
    "Authorization": "Bearer $MY_TOKEN"
  },
  "allowedEnvVars": ["MY_TOKEN"],
  "statusMessage": "Validation...",
  "if": "Bash(git *)"
}
```

### 3. Hooks d'outil MCP (`type: "mcp_tool"`)

Appellent des outils sur des serveurs MCP connectés.

**Champs :**
```json
{
  "type": "mcp_tool",
  "server": "my_server",
  "tool": "security_scan",
  "input": {
    "file_path": "${tool_input.file_path}"
  },
  "timeout": 600
}
```

### 4. Hooks de prompt (`type: "prompt"`)

Envoient une évaluation LLM à tour unique.

**Champs :**
```json
{
  "type": "prompt",
  "prompt": "Should this file be edited? Analyze: $ARGUMENTS",
  "model": "claude-opus",
  "timeout": 30
}
```

### 5. Hooks d'agent (`type: "agent"`)

Créent des sous-agents avec accès aux outils (expérimental).

**Champs :**
```json
{
  "type": "agent",
  "prompt": "Validate this operation: $ARGUMENTS",
  "timeout": 60
}
```

## Champs communs des hooks

| Champ | Requis | Description |
|-------|--------|-------------|
| `type` | Oui | `"command"`, `"http"`, `"mcp_tool"`, `"prompt"`, ou `"agent"` |
| `if` | Non | Syntaxe de règle de permission : `"Bash(git *)"`, `"Edit(*.ts)"` |
| `timeout` | Non | Secondes avant annulation (défauts : 600 pour command/http/mcp_tool, 30 pour prompt, 60 pour agent) |
| `statusMessage` | Non | Message de spinner personnalisé pendant l'exécution |
| `once` | Non | Si `true`, s'exécute une fois par session puis est supprimé (skills/agents uniquement) |

## Entrée/Sortie des hooks

### Champs d'entrée communs

Tous les hooks reçoivent :

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "effort": { "level": "medium" },
  "agent_id": "optional-subagent-id",
  "agent_type": "Explore"
}
```

### Codes de sortie

- **0** : Succès (analyser le JSON de stdout)
- **2** : Erreur bloquante (lire stderr comme message d'erreur)
- **Autre** : Erreur non bloquante (continuer l'exécution)

### Schéma de sortie JSON

```json
{
  "continue": true,
  "stopReason": "message optionnel",
  "suppressOutput": false,
  "systemMessage": "texte d'avertissement",
  "terminalSequence": "séquence d'échappement",
  "decision": "block",
  "reason": "pourquoi bloqué",
  "additionalContext": "contexte pour Claude",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "texte de raison"
  }
}
```

### Séquences de terminal

Limitées aux séquences OSC sur liste blanche :
- OSC 0, 1, 2 : Titres de fenêtre
- OSC 9 : Notifications (iTerm2, Windows Terminal, WezTerm)
- OSC 99 : Notifications Kitty
- OSC 777 : Notifications urxvt/Ghostty
- BEL : Sonnerie

```bash
seq=$(printf '\033]777;notify;%s;%s\007' "$title" "$body")
jq -nc --arg seq "$seq" '{terminalSequence: $seq}'
```

## Référence des événements de hook

### SessionStart

**Se déclenche :** Nouvelle session, reprise, effacement ou compaction
**Matchers :** `startup`, `resume`, `clear`, `compact`

**Entrée :**
```json
{
  "source": "startup",
  "model": "claude-sonnet-4-6",
  "session_title": "titre-optionnel"
}
```

**Contrôle de décision :**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Branche actuelle : main",
    "sessionTitle": "mon-projet",
    "watchPaths": ["/chemin/absolu"],
    "reloadSkills": true,
    "initialUserMessage": "Commencer par cette invite"
  }
}
```

**Environnement :** `$CLAUDE_ENV_FILE` disponible pour persister les variables

### Setup

**Se déclenche :** `--init-only`, `--init` ou `--maintenance` en mode `-p`
**Matchers :** `init`, `maintenance`

**Utilisation :** Préparation unique en CI ou scripts

### UserPromptSubmit

**Se déclenche :** Avant que Claude traite l'invite utilisateur
**Matchers :** Aucun (se déclenche toujours)

**Entrée :**
```json
{
  "prompt": "texte d'entrée utilisateur"
}
```

**Décision :** `"decision": "block"` pour rejeter

**Timeout par défaut :** 30 secondes

### UserPromptExpansion

**Se déclenche :** Une commande tapée par l'utilisateur se développe en invite
**Matchers :** Noms de skill/commande

**Entrée :**
```json
{
  "command": "nom-du-skill",
  "expansion": "texte d'invite développé"
}
```

**Décision :** `"decision": "block"` pour bloquer le développement

### PreToolUse

**Se déclenche :** Avant l'exécution de l'outil
**Matchers :** Noms d'outils (ex. `Bash`, `mcp__memory__.*`)

**Entrée :**
```json
{
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
```

**Contrôle de décision :**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny|allow|ask|defer",
    "permissionDecisionReason": "texte de raison",
    "updatedToolInput": { "command": "commande modifiée" }
  }
}
```

### PostToolUse

**Se déclenche :** Après l'exécution réussie de l'outil
**Matchers :** Noms d'outils

**Entrée :** Même que `PreToolUse` plus `tool_output`

**Décision :** `"decision": "block"` pour arrêter le traitement

### PostToolUseFailure

**Se déclenche :** Après l'échec de l'exécution de l'outil
**Matchers :** Noms d'outils

### PostToolBatch

**Se déclenche :** Après un lot d'appels d'outils parallèles
**Matchers :** Aucun

**Décision :** `"decision": "block"` pour arrêter avant le prochain appel au modèle

### Stop

**Se déclenche :** Quand Claude finit de répondre
**Matchers :** Aucun

**Décision :** `"decision": "block"` pour continuer la conversation

### StopFailure

**Se déclenche :** Le tour se termine en raison d'une erreur API
**Matchers :** Type d'erreur (`rate_limit`, `authentication_failed`, etc.)

**Note :** La sortie et le code de sortie sont ignorés

### PermissionRequest

**Se déclenche :** La boîte de dialogue de permission apparaît
**Matchers :** Noms d'outils

**Contrôle de décision :**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow|deny",
      "updatedInput": { "command": "modifié" }
    }
  }
}
```

### PermissionDenied

**Se déclenche :** L'appel d'outil est refusé par le classificateur de mode auto
**Matchers :** Noms d'outils

**Décision :**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionDenied",
    "retry": true
  }
}
```

### Notification

**Se déclenche :** Claude Code envoie une notification
**Matchers :** `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, `elicitation_complete`, `elicitation_response`

### MessageDisplay

**Se déclenche :** Pendant l'affichage du message de l'assistant
**Matchers :** Aucun (affichage uniquement)

**Décision :**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "MessageDisplay",
    "displayContent": "texte modifié à afficher"
  }
}
```

### SubagentStart

**Se déclenche :** Sous-agent créé
**Matchers :** Type d'agent

### SubagentStop

**Se déclenche :** Sous-agent terminé
**Matchers :** Type d'agent

### TaskCreated

**Se déclenche :** Création de tâche via `TaskCreate`
**Matchers :** Aucun

### TaskCompleted

**Se déclenche :** Tâche marquée comme terminée
**Matchers :** Aucun

### TeammateIdle

**Se déclenche :** Un coéquipier de l'équipe d'agents est sur le point d'être inactif
**Matchers :** Aucun

### CwdChanged

**Se déclenche :** Le répertoire de travail change
**Matchers :** Aucun (se déclenche toujours)

**Environnement :** `$CLAUDE_ENV_FILE` disponible

### FileChanged

**Se déclenche :** Un fichier surveillé change sur le disque
**Matchers :** Noms de fichiers littéraux (`.envrc|.env`)

### ConfigChange

**Se déclenche :** Un fichier de configuration change
**Matchers :** `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills`

**Décision :** `"decision": "block"` pour empêcher le changement

### InstructionsLoaded

**Se déclenche :** CLAUDE.md ou `.claude/rules/*.md` chargé
**Matchers :** `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact`

### PreCompact

**Se déclenche :** Avant la compaction du contexte
**Matchers :** `manual`, `auto`

**Décision :** `"decision": "block"` pour empêcher la compaction

### PostCompact

**Se déclenche :** Après la compaction terminée
**Matchers :** `manual`, `auto`

### Elicitation

**Se déclenche :** Un serveur MCP demande une saisie utilisateur
**Matchers :** Noms de serveurs MCP

**Contrôle de décision :**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "Elicitation",
    "action": "accept|decline|cancel",
    "content": { "field_name": "value" }
  }
}
```

### ElicitationResult

**Se déclenche :** L'utilisateur répond à l'élicitation MCP
**Matchers :** Noms de serveurs MCP

### WorktreeCreate

**Se déclenche :** Création de worktree via `--worktree` ou isolation
**Matchers :** Aucun

**Sortie :** Le hook de commande imprime le chemin sur stdout ; le hook HTTP retourne `hookSpecificOutput.worktreePath`

### WorktreeRemove

**Se déclenche :** Suppression du worktree à la sortie ou fin du sous-agent
**Matchers :** Aucun

### SessionEnd

**Se déclenche :** La session se termine
**Matchers :** Raison de sortie (`clear`, `resume`, `logout`, `other`)

## Espaces réservés de chemin

Utiliser dans les arguments de commande et les URL :

```json
{
  "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/script.sh",
  "args": ["${CLAUDE_PLUGIN_ROOT}/bin/tool"]
}
```

- `${CLAUDE_PROJECT_DIR}` : Racine du projet
- `${CLAUDE_PLUGIN_ROOT}` : Répertoire d'installation du plugin
- `${CLAUDE_PLUGIN_DATA}` : Répertoire de données persistantes du plugin

## Hooks dans les skills et agents

Définir dans le frontmatter YAML :

```yaml
---
name: my-skill
description: Description
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./validate.sh"
---
```

## Exemples de configuration

### Hook de commande basique

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/check-safety.sh",
            "args": []
          }
        ]
      }
    ]
  }
}
```

### Hook HTTP avec authentification

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/hooks/validation",
            "headers": {
              "Authorization": "Bearer $API_TOKEN"
            },
            "allowedEnvVars": ["API_TOKEN"],
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Hook d'outil MCP

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "mcp_tool",
            "server": "security",
            "tool": "scan_file",
            "input": {
              "path": "${tool_input.file_path}"
            }
          }
        ]
      }
    ]
  }
}
```

### Hook asynchrone avec Rewake

```json
{
  "type": "command",
  "command": "background-check.sh",
  "async": true,
  "asyncRewake": true,
  "timeout": 300
}
```

## Désactiver les hooks

```json
{
  "disableAllHooks": true
}
```

Supprimer les entrées de hook individuelles de la configuration JSON pour les effacer.

## Comportement du code de sortie 2

| Événement | Comportement |
|-----------|-------------|
| `PreToolUse` | Bloque l'appel d'outil |
| `PermissionRequest` | Refuse la permission |
| `UserPromptSubmit` | Rejette l'invite |
| `UserPromptExpansion` | Bloque le développement |
| `Stop` | Empêche l'arrêt, continue |
| `SubagentStop` | Empêche l'arrêt du sous-agent |
| `ConfigChange` | Bloque le changement de configuration |
| `PreCompact` | Bloque la compaction |
| Autres événements | Erreur non bloquante |

## Le menu `/hooks`

Voir les hooks configurés dans le navigateur en lecture seule :
- Affiche les 5 types de hooks avec les sources
- Affiche l'événement, le matcher et les détails
- Pas d'édition directe (modifier le JSON directement)

## Fonctionnalités avancées

### Exécution asynchrone

```json
{
  "async": true,
  "asyncRewake": true
}
```

S'exécute en arrière-plan, réveille optionnellement Claude sur le code de sortie 2.

### Limites de sortie

Les sorties de chaîne de hook sont limitées à 10 000 caractères. L'excédent est écrit dans un fichier avec référence de chemin.

### Syntaxe des règles de permission

Utiliser dans le champ `if` :
- `"Bash(git *)"` - correspond aux sous-commandes git
- `"Edit(*.ts)"` - correspond aux fichiers TypeScript
- La correspondance du nom d'outil suit les mêmes règles que le matcher

### Environnement de variable

`$CLAUDE_EFFORT` disponible dans les hooks de commande où le niveau d'effort s'applique.
