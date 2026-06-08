# Gérer les approbations et l'entrée utilisateur dans le SDK Agent

> Présentez les demandes d'approbation et les questions de clarification de Claude aux utilisateurs.

## Deux situations déclenchant `canUseTool`

1. **Permission d'utiliser un outil** : Claude veut utiliser un outil non auto-approuvé
2. **Questions de clarification** : Claude appelle l'outil `AskUserQuestion`

Le callback peut rester en attente indéfiniment. Pour les longs délais, utiliser le hook `defer` pour permettre au processus de quitter et reprendre plus tard.

## Configuration du callback

```python
async def handle_tool_request(tool_name, input_data, context):
    ...

options = ClaudeAgentOptions(can_use_tool=handle_tool_request)
```

```typescript
async function handleToolRequest(toolName, input, options) {
  // options.signal: AbortSignal
  // options.suggestions: PermissionUpdate[] (pour mémoriser les décisions)
}
const options = { canUseTool: handleToolRequest };
```

Note Python: `can_use_tool` nécessite le mode streaming et un hook `PreToolUse` factice qui retourne `{"continue_": True}`.

## Types de réponse

| Réponse | Python | TypeScript |
|---------|--------|------------|
| Autoriser | `PermissionResultAllow(updated_input=...)` | `{ behavior: "allow", updatedInput }` |
| Refuser | `PermissionResultDeny(message=...)` | `{ behavior: "deny", message }` |

## Patterns de réponse

### Approuver
```python
return PermissionResultAllow(updated_input=input_data)
```

### Approuver avec modifications
```python
sandboxed_input = {**input_data}
sandboxed_input["command"] = input_data["command"].replace("/tmp", "/tmp/sandbox")
return PermissionResultAllow(updated_input=sandboxed_input)
```

### Approuver et mémoriser
```python
persist = [s for s in context.suggestions if s.destination == "localSettings"]
return PermissionResultAllow(updated_input=input_data, updated_permissions=persist)
```
Écrit une règle dans `.claude/settings.local.json` pour les futures sessions.

### Rejeter
```python
return PermissionResultDeny(message="User rejected this action")
```

### Suggérer une alternative
```python
return PermissionResultDeny(
    message="User doesn't want to delete files. Archive to .zip instead."
)
```

## Gérer les questions de clarification (AskUserQuestion)

Claude appelle cet outil quand il a besoin de plus de direction.

```python
async def can_use_tool(tool_name, input_data, context):
    if tool_name == "AskUserQuestion":
        return await handle_clarifying_questions(input_data)
    return await prompt_for_approval(tool_name, input_data)
```

### Format de la question

```json
{
  "questions": [
    {
      "question": "How should I format the output?",
      "header": "Format",
      "options": [
        { "label": "Summary", "description": "Brief overview" },
        { "label": "Detailed", "description": "Full explanation" }
      ],
      "multiSelect": false
    }
  ]
}
```

### Format de la réponse

```python
return PermissionResultAllow(
    updated_input={
        "questions": input_data.get("questions", []),
        "answers": {
            "How should I format the output?": "Summary",
            "Which sections should I include?": ["Introduction", "Conclusion"],
        },
    }
)
```

Clé = texte de `question`, valeur = `label` de l'option sélectionnée. Pour multi-select: tableau de labels ou joints par `", "`.

### Aperçus d'options (TypeScript)

```typescript
toolConfig: {
  askUserQuestion: { previewFormat: "html" }  // "html" ou "markdown"
}
```

Ajoute un champ `preview` à chaque option pour l'affichage visuel.

## Méthodes alternatives d'entrée

- **Streaming input** : envoyer des messages pendant l'exécution (chat interactif, interruptions)
- **Outils personnalisés** : formulaires complexes, systèmes d'approbation externes

## Important

Pour `AskUserQuestion`: inclure dans le tableau `tools` si vous le restreignez explicitement:
```python
tools=["Read", "Glob", "Grep", "AskUserQuestion"]
```

Limitations:
- `AskUserQuestion` non disponible dans les sous-agents générés via l'outil Agent
- 1-4 questions par appel, 2-4 options chacune
