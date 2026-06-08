# Stockage de session externe dans le SDK Agent

> Miroir les transcriptions de session vers S3, Redis ou votre propre backend pour que n'importe quel hôte puisse les reprendre.

## Pourquoi utiliser un SessionStore?

- **Déploiements multi-hôtes** : serverless, workers autoscalés, exécuteurs CI
- **Durabilité** : les conteneurs locaux sont éphémères
- **Conformité** : stockage que vous gouvernez avec vos propres règles

## L'interface `SessionStore`

```typescript
type SessionKey = {
  projectKey: string;
  sessionId: string;
  subpath?: string;
};

type SessionStore = {
  // Requis
  append(key: SessionKey, entries: SessionStoreEntry[]): Promise<void>;
  load(key: SessionKey): Promise<SessionStoreEntry[] | null>;
  
  // Optionnel
  listSessions?(projectKey: string): Promise<Array<{ sessionId: string; mtime: number }>>;
  delete?(key: SessionKey): Promise<void>;
  listSubkeys?(key: { projectKey: string; sessionId: string }): Promise<string[]>;
};
```

```python
class SessionStore(Protocol):
    async def append(self, key: SessionKey, entries: list[SessionStoreEntry]) -> None: ...
    async def load(self, key: SessionKey) -> list[SessionStoreEntry] | None: ...
    
    # Optionnel
    async def list_sessions(self, project_key: str) -> list[SessionStoreListEntry]: ...
    async def delete(self, key: SessionKey) -> None: ...
    async def list_subkeys(self, key: SessionListSubkeysKey) -> list[str]: ...
```

### Méthodes

| Méthode | Requise | Appelée quand |
|---------|--------|---------------|
| `append` | Oui | Après chaque lot d'entrées de transcription |
| `load` | Oui | Avant lancement du sous-processus si `resume` défini |
| `listSessions` | Non | Par `listSessions({ sessionStore })` et `continue: true` |
| `delete` | Non | Par `deleteSession({ sessionStore })` |
| `listSubkeys` | Non | Pour découvrir les transcriptions de sous-agents à la reprise |

## Démarrage rapide avec InMemorySessionStore

```python
from claude_agent_sdk import InMemorySessionStore, query, ClaudeAgentOptions, ResultMessage

store = InMemorySessionStore()

session_id = None
async for message in query(
    prompt="List the Python files under src/",
    options=ClaudeAgentOptions(session_store=store),
):
    if isinstance(message, ResultMessage):
        session_id = message.session_id

# Reprendre depuis le magasin
async for message in query(
    prompt="Summarize what those files do",
    options=ClaudeAgentOptions(session_store=store, resume=session_id),
):
    if isinstance(message, ResultMessage) and message.subtype == "success":
        print(message.result)
```

## Adaptateurs de référence

Disponibles dans le repo TypeScript SDK:

| Adaptateur | Client | Modèle de stockage |
|------------|--------|-------------------|
| `S3SessionStore` | `@aws-sdk/client-s3` | Fichiers JSONL par batch |
| `RedisSessionStore` | `ioredis` | Liste RPUSH/LRANGE + sorted-set |
| `PostgresSessionStore` | `pg` | Une ligne par entrée, JSONB |

```typescript
import { S3SessionStore } from "./S3SessionStore";  // Copier depuis examples/session-stores/s3
import { S3Client } from "@aws-sdk/client-s3";

const store = new S3SessionStore({
  bucket: "my-claude-sessions",
  prefix: "transcripts",
  client: new S3Client({ region: "us-east-1" }),
});
```

## Valider votre adaptateur

```python
import pytest
from claude_agent_sdk.testing import run_session_store_conformance

@pytest.mark.asyncio
async def test_my_store_conformance():
    await run_session_store_conformance(MyRedisStore)
```

## Notes de comportement

### Architecture à double écriture
Le magasin est un miroir, pas un remplacement. Le sous-processus écrit d'abord sur disque local; le SDK transfère en copie.

Incompatible avec:
- `persistSession: false` (nécessite écritures locales)
- `enableFileCheckpointing` (blobs de sauvegarde non mirrorés)

### Écritures en miroir au mieux
Si `append()` échoue → message `{ type: "system", subtype: "mirror_error" }` dans l'itérateur, la requête continue. Surveiller `mirror_error` si la durabilité est importante.

### Rétention
Le SDK ne supprime jamais de votre magasin. Implémenter vos propres TTL/politiques de rétention.

## Fonctions SDK supportant SessionStore

`query()`, `startup()`, `listSessions()`, `getSessionInfo()`, `getSessionMessages()`, `renameSession()`, `tagSession()`, `deleteSession()`, `forkSession()`, `listSubagents()`, `getSubagentMessages()`
