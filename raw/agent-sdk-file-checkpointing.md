# File Checkpointing dans le SDK Agent

> Suivre les modifications de fichiers et restaurer les fichiers à n'importe quel état antérieur

## Ce que le point de contrôle suit

Les modifications via les outils: `Write`, `Edit`, `NotebookEdit`

Non suivi: modifications via commandes `Bash` (`echo > file.txt`, `sed -i`, etc.)

Note: Le rembobinage restaure les fichiers sur disque, pas la conversation. L'historique reste intact.

## Implémenter le point de contrôle

### 1. Activer

```python
options = ClaudeAgentOptions(
    enable_file_checkpointing=True,
    permission_mode="acceptEdits",
    extra_args={"replay-user-messages": None},  # Requis pour recevoir les UUID
)
```

```typescript
const opts = {
  enableFileCheckpointing: true,
  permissionMode: "acceptEdits" as const,
  extraArgs: { "replay-user-messages": null }  // Requis
};
```

### 2. Capturer l'UUID du point de contrôle

```python
checkpoint_id = None
session_id = None

async with ClaudeSDKClient(options) as client:
    await client.query("Refactor the authentication module")
    async for message in client.receive_response():
        if isinstance(message, UserMessage) and message.uuid and not checkpoint_id:
            checkpoint_id = message.uuid  # Premier UUID = point de restauration initial
        if isinstance(message, ResultMessage) and not session_id:
            session_id = message.session_id
```

```typescript
for await (const message of response) {
  if (message.type === "user" && message.uuid && !checkpointId) {
    checkpointId = message.uuid;
  }
  if ("session_id" in message) sessionId = message.session_id;
}
```

### 3. Rembobiner

```python
async with ClaudeSDKClient(
    ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
) as client:
    await client.query("")  # Invite vide pour ouvrir la connexion
    async for message in client.receive_response():
        await client.rewind_files(checkpoint_id)
        break
```

```typescript
const rewindQuery = query({ prompt: "", options: { ...opts, resume: sessionId } });
for await (const msg of rewindQuery) {
  await rewindQuery.rewindFiles(checkpointId);
  break;
}
```

Via CLI:
```bash
claude -p --resume <session-id> --rewind-files <checkpoint-uuid>
```

## Patterns courants

### Point de contrôle avant opérations risquées

```python
safe_checkpoint = None

async with ClaudeSDKClient(options) as client:
    await client.query("Refactor...")
    async for message in client.receive_response():
        if isinstance(message, UserMessage) and message.uuid:
            safe_checkpoint = message.uuid  # Met à jour avant chaque tour

        if your_revert_condition and safe_checkpoint:
            await client.rewind_files(safe_checkpoint)
            break
```

### Points de restauration multiples

```python
checkpoints = []

async for message in client.receive_response():
    if isinstance(message, UserMessage) and message.uuid:
        checkpoints.append({
            "id": message.uuid,
            "description": f"After turn {len(checkpoints) + 1}",
        })

# Plus tard: rembobiner vers n'importe quel point
target = checkpoints[0]
```

## Limitations

| Limitation | Description |
|------------|-------------|
| Write/Edit/NotebookEdit uniquement | Bash non suivi |
| Même session | Points de contrôle liés à la session d'origine |
| Contenu de fichiers uniquement | Création/déplacement de répertoires non annulés |
| Fichiers locaux | Fichiers distants non suivis |
| Incompatible avec SessionStore | Ne pas combiner avec `session_store` |
| Incompatible avec `persistSession: false` | Nécessite persistance locale |

## Dépannage

- **Options non reconnues** : mettre à jour le SDK vers la dernière version
- **`message.uuid` undefined** : ajouter `extra_args={"replay-user-messages": None}`
- **"No file checkpoint found"** : vérifier que `enable_file_checkpointing=True` sur la session d'origine
- **"ProcessTransport is not ready"** : reprendre la session avec invite vide avant d'appeler `rewindFiles()`
