# Hooks SDK Agent — Intercepter et contrôler le comportement des agents

> Interceptez et personnalisez le comportement des agents aux points d'exécution clés avec des hooks

## Fonctionnement

1. Un événement se déclenche (PreToolUse, PostToolUse, Stop, etc.)
2. Le SDK collecte les hooks enregistrés pour ce type d'événement
3. Les matchers filtrent les hooks qui s'exécutent
4. Les fonctions de rappel s'exécutent
5. Le rappel retourne une décision (allow, deny, modify)

## Hooks disponibles

| Événement | Python | TypeScript | Déclencheur | Cas d'usage |
|-----------|--------|------------|-------------|-------------|
| `PreToolUse` | Oui | Oui | Avant appel d'outil | Bloquer commandes dangereuses |
| `PostToolUse` | Oui | Oui | Après résultat d'outil | Auditer les changements |
| `PostToolUseFailure` | Oui | Oui | Échec d'outil | Gérer les erreurs |
| `PostToolBatch` | Non | Oui | Lot complet d'outils | Injecter conventions |
| `UserPromptSubmit` | Oui | Oui | Soumission d'invite | Injecter contexte |
| `MessageDisplay` | Non | Oui | Message assistant terminé | Reformater texte affiché |
| `Stop` | Oui | Oui | Arrêt de l'agent | Sauvegarder état de session |
| `SubagentStart` | Oui | Oui | Démarrage sous-agent | Suivre tâches parallèles |
| `SubagentStop` | Oui | Oui | Fin sous-agent | Agréger résultats |
| `PreCompact` | Oui | Oui | Compaction de conversation | Archiver transcription |
| `PermissionRequest` | Oui | Oui | Dialogue de permission | Gestion des permissions |
| `SessionStart` | Non | Oui | Initialisation session | Initialiser journalisation |
| `SessionEnd` | Non | Oui | Arrêt session | Nettoyer ressources |
| `Notification` | Oui | Oui | Messages d'état | Envoyer à Slack/PagerDuty |
| `Setup` | Non | Oui | Config/maintenance | Init de session |
| `TeammateIdle` | Non | Oui | Coéquipier inactif | Réassigner travail |
| `TaskCompleted` | Non | Oui | Tâche terminée | Agréger résultats |
| `ConfigChange` | Non | Oui | Fichier config change | Recharger paramètres |
| `WorktreeCreate` | Non | Oui | Git worktree créé | Suivre workspaces |
| `WorktreeRemove` | Non | Oui | Git worktree supprimé | Nettoyer resources |

## Configuration

```python
options = ClaudeAgentOptions(
    hooks={
        "PreToolUse": [HookMatcher(matcher="Write|Edit", hooks=[my_callback])]
    }
)
```

```typescript
options = {
  hooks: {
    PreToolUse: [{ matcher: "Write|Edit", hooks: [myCallback] }]
  }
}
```

## Matchers

- `Write|Edit` : correspondance exacte avec `|` pour alternatives
- `^mcp__` : expression régulière (contient autres caractères)
- `*` ou vide : correspond à tout
- `mcp__memory__.*` : tous les outils d'un serveur MCP

## Fonctions de rappel

### Entrées

- **Données d'entrée** : objet typé avec détails de l'événement
- **ID d'utilisation d'outil** : corrèle PreToolUse et PostToolUse
- **Contexte** : `signal` (AbortSignal) en TypeScript

### Sorties

Retourner `{}` pour autoriser sans modifications.

```python
return {
    "hookSpecificOutput": {
        "hookEventName": input_data["hook_event_name"],
        "permissionDecision": "deny",  # allow, deny, ask, defer
        "permissionDecisionReason": "Cannot modify .env files",
    }
}
```

Priorité des décisions: `deny` > `defer` > `ask` > `allow`

### Sortie asynchrone (non-bloquante)

```python
async def async_hook(input_data, tool_use_id, context):
    asyncio.create_task(send_to_logging_service(input_data))
    return {"async_": True, "asyncTimeout": 30000}
```

## Exemple: bloquer les fichiers .env

```python
async def protect_env_files(input_data, tool_use_id, context):
    file_path = input_data["tool_input"].get("file_path", "")
    file_name = file_path.split("/")[-1]
    if file_name == ".env":
        return {
            "hookSpecificOutput": {
                "hookEventName": input_data["hook_event_name"],
                "permissionDecision": "deny",
                "permissionDecisionReason": "Cannot modify .env files",
            }
        }
    return {}
```

## Exemple: modifier l'entrée de l'outil

```python
async def redirect_to_sandbox(input_data, tool_use_id, context):
    if input_data["tool_name"] == "Write":
        original_path = input_data["tool_input"].get("file_path", "")
        return {
            "hookSpecificOutput": {
                "hookEventName": input_data["hook_event_name"],
                "permissionDecision": "allow",
                "updatedInput": {
                    **input_data["tool_input"],
                    "file_path": f"/sandbox{original_path}",
                },
            }
        }
    return {}
```

## Exemple: envoyer notifications à Slack

```python
async def notification_handler(input_data, tool_use_id, context):
    try:
        await asyncio.to_thread(send_slack, input_data.get("message", ""))
    except Exception as e:
        print(f"Failed: {e}")
    return {}

options = ClaudeAgentOptions(
    hooks={"Notification": [HookMatcher(hooks=[notification_handler])]}
)
```
