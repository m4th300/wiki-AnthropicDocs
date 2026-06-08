# Sessions dans le SDK Agent

> Comment les sessions conservent l'historique des conversations, et quand utiliser continue, resume et fork.

Une session = l'historique des conversations accumulé pendant que l'agent travaille. Écrit automatiquement sur le disque.

Note: Les sessions conservent la **conversation**, pas le système de fichiers. Pour les instantanés de fichiers, utiliser le file checkpointing.

## Choisir une approche

| Ce que vous construisez | Ce qu'il faut utiliser |
|------------------------|----------------------|
| Tâche unique | Rien d'extra. Un seul `query()`. |
| Chat multi-tours dans un seul processus | `ClaudeSDKClient` (Python) ou `continue: true` (TypeScript) |
| Reprendre après redémarrage (dernière session) | `continue_conversation=True` (Python) / `continue: true` (TS) |
| Reprendre une session spécifique passée | Capturer l'ID et passer à `resume` |
| Explorer approche alternative | Bifurquer la session |
| Sans état, rien sur disque (TS) | `persistSession: false` |

## Continue vs Resume vs Fork

- **Continue** : trouve la session la plus récente dans le répertoire courant (pas besoin de suivre l'ID)
- **Resume** : prend un ID de session spécifique (plusieurs sessions en parallèle, ou pas la plus récente)
- **Fork** : crée une nouvelle session avec copie de l'historique. L'original reste inchangé.

## Gestion automatique des sessions

### Python: `ClaudeSDKClient`

```python
async with ClaudeSDKClient(options=options) as client:
    # Première requête: client capture l'ID de session
    await client.query("Analyze the auth module")
    async for message in client.receive_response():
        print_response(message)

    # Deuxième requête: continue automatiquement la même session
    await client.query("Now refactor it to use JWT")
    async for message in client.receive_response():
        print_response(message)
```

### TypeScript: `continue: true`

```typescript
// Première requête: crée une nouvelle session
for await (const message of query({ prompt: "Analyze the auth module" })) { ... }

// Deuxième requête: continue: true reprend la session la plus récente
for await (const message of query({
  prompt: "Now refactor it to use JWT",
  options: { continue: true }
})) { ... }
```

## Capturer l'ID de session

```python
session_id = None
async for message in query(prompt="Analyze...", options=options):
    if isinstance(message, ResultMessage):
        session_id = message.session_id
```

```typescript
let sessionId: string | undefined;
for await (const message of query({ prompt: "Analyze..." })) {
  if (message.type === "result") {
    sessionId = message.session_id;
  }
}
```

## Reprendre par ID

```python
async for message in query(
    prompt="Now implement the refactoring you suggested",
    options=ClaudeAgentOptions(
        resume=session_id,
        allowed_tools=["Read", "Edit", "Write", "Glob", "Grep"],
    ),
):
    if isinstance(message, ResultMessage) and message.subtype == "success":
        print(message.result)
```

Raisons courantes de reprendre:
- Suivi sur une tâche terminée (contexte déjà chargé)
- Récupérer après `error_max_turns` ou `error_max_budget_usd`
- Redémarrer le processus

Tip: Si la reprise retourne une session nouvelle, vérifier que le `cwd` correspond. Les sessions sont stockées dans `~/.claude/projects/<encoded-cwd>/`.

## Bifurquer

```python
# Fork: branche à partir de session_id vers une nouvelle session
async for message in query(
    prompt="Instead of JWT, implement OAuth2",
    options=ClaudeAgentOptions(
        resume=session_id,
        fork_session=True,
    ),
):
    if isinstance(message, ResultMessage):
        forked_id = message.session_id  # Nouvel ID distinct de session_id
```

Note: Fork branche l'historique de conversation, PAS le système de fichiers. Les modifications de fichiers par l'agent forké sont réelles.

## Sessions multi-hôtes

Les fichiers de session sont locaux à la machine. Options:
1. Déplacer le fichier de session vers le nouvel hôte (même chemin, même `cwd`)
2. Utiliser un `SessionStore` externe (voir session-storage)
3. Capturer les résultats comme état d'application et passer dans un nouveau prompt

## Fonctions de gestion de sessions

```python
# Python
await list_sessions(options=None)           # liste les sessions
await get_session_messages(session_id)      # messages d'une session
await get_session_info(session_id)          # métadonnées
await rename_session(session_id, title)     # renommer
await tag_session(session_id, tag)          # tagger
```

```typescript
// TypeScript
await listSessions({ dir, limit, includeWorktrees });
await getSessionMessages(sessionId, options);
await getSessionInfo(sessionId, options);
await renameSession(sessionId, title, options);
await tagSession(sessionId, tag, options);
```
