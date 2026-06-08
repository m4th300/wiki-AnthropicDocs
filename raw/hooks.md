# Référence des hooks

Source: https://code.claude.com/docs/fr/hooks (référence complète)

## Vue d'ensemble

Les hooks sont des commandes shell, endpoints HTTP ou prompts LLM qui s'exécutent automatiquement à des points spécifiques du cycle de vie de Claude Code.

## Événements du cycle de vie

| Événement | Déclenchement |
|-----------|--------------|
| `SessionStart` | Démarrage ou reprise de session |
| `Setup` | Avec `--init-only`, `--init` ou `--maintenance` |
| `UserPromptSubmit` | À la soumission d'un prompt |
| `UserPromptExpansion` | Lors de l'expansion d'une commande |
| `PreToolUse` | Avant l'exécution d'un appel d'outil |
| `PermissionRequest` | À l'affichage d'un dialogue de permission |
| `PostToolUse` | Après succès d'un appel d'outil |
| `PostToolUseFailure` | Après l'échec d'un appel d'outil |
| `Stop` | À la fin de la réponse de Claude |
| `StopFailure` | Si la réponse se termine par erreur |
| `SessionEnd` | À la fin de la session |
| `FileChanged` | Quand un fichier surveillé change |
| `CwdChanged` | Quand le répertoire de travail change |
| `ConfigChange` | Quand la configuration change |
| `WorktreeCreate/Remove` | À la création/suppression d'un worktree |
| `InstructionsLoaded` | Quand CLAUDE.md ou rules se charge |
| `PreCompact / PostCompact` | Avant/après compaction du contexte |
| `SubagentStart / SubagentStop` | Au démarrage/arrêt d'un subagent |

## Configuration

### Emplacements

| Emplacement | Portée |
|------------|--------|
| `~/.claude/settings.json` | Tous les projets |
| `.claude/settings.json` | Projet (partageable) |
| `.claude/settings.local.json` | Projet (local) |

### Structure

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/validate-bash.sh"
          }
        ]
      }
    ]
  }
}
```

## Patterns de matcher

| Valeur | Comportement |
|--------|-------------|
| `"*"`, `""`, ou omis | Correspond à tous |
| Lettres/chiffres/`_`/`\|` | Chaîne exacte ou liste `Bash\|Edit` |
| Autres caractères | Regex JavaScript `^Notebook` |

Outils MCP : `mcp__<server>__<tool>` → ex: `mcp__memory__create_entities`, `mcp__memory__.*`

## Types de gestionnaires

### 1. Commande (shell)
```json
{
  "type": "command",
  "command": "/path/to/script.sh",
  "args": [],           // Forme exec (pas de shell)
  "timeout": 600,
  "statusMessage": "Validating...",
  "async": false,
  "shell": "bash"
}
```

**Forme shell** (sans `args`) : chaîne passée au shell.

### 2. HTTP
```json
{
  "type": "http",
  "url": "http://localhost:8080/hooks/pre-tool-use",
  "timeout": 30,
  "headers": {"Authorization": "Bearer $MY_TOKEN"},
  "allowedEnvVars": ["MY_TOKEN"]
}
```

### 3. MCP
```json
{
  "type": "mcp_tool",
  "server": "my_server",
  "tool": "security_scan",
  "input": {"file_path": "${tool_input.file_path}"}
}
```

### 4. Prompt LLM
```json
{
  "type": "prompt",
  "prompt": "Is this code change safe? Analyze: $ARGUMENTS",
  "model": "claude-opus-4-1",
  "timeout": 30
}
```

### 5. Agent
```json
{
  "type": "agent",
  "prompt": "Review the changes: $ARGUMENTS",
  "timeout": 60
}
```

## Codes de sortie et contrôle

- **Exit 0** : Succès. Claude analyse stdout pour JSON
- **Exit 2** : Erreur bloquante. stderr devient message d'erreur
- **Autre** : Erreur non-bloquante. Exécution continue

### Format JSON de sortie

```json
{
  "continue": true,
  "decision": "block",
  "reason": "Reason for blocking",
  "suppressOutput": false,
  "systemMessage": "Warning message",
  "additionalContext": "Context for Claude",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Blocked by policy"
  }
}
```

## Entrée des hooks (champs communs)

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "hook_event_name": "PreToolUse",
  "permission_mode": "default",
  "effort": {"level": "medium"}
}
```

## Contrôle de permission (PreToolUse)

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",     // allow | deny | ask | defer
    "permissionDecisionReason": "Destructive command blocked",
    "modifiedInput": {"command": "safer-command"},
    "additionalContext": "Security policy violation"
  }
}
```

## Exemple complet : bloquer rm -rf

```bash
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command' < /dev/stdin)

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    "hookSpecificOutput": {
      "hookEventName": "PreToolUse",
      "permissionDecision": "deny",
      "permissionDecisionReason": "rm -rf blocked by policy"
    }
  }'
  exit 0
fi

exit 0
```

## SessionStart : injection de contexte

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Branch: main\nIssues: #123, #456",
    "sessionTitle": "my-session",
    "watchPaths": ["/path/to/watch"],
    "reloadSkills": true,
    "initialUserMessage": "First prompt text"
  }
}
```

## Désactiver tous les hooks

```json
{
  "disableAllHooks": true
}
```

## Menu `/hooks`

Inspecter tous les hooks configurés en lecture seule.
