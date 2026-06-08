# Référence SDK Agent Python

## Installation

```bash
pip install claude-agent-sdk
```

Nécessite Python 3.10+.

## Deux approches principales

1. **`query()`** : pour les tâches ponctuelles avec gestion automatique des sessions
2. **`ClaudeSDKClient`** : pour les conversations continues avec contrôle explicite des sessions

## `query()`

```python
async def query(
    *,
    prompt: str | AsyncIterable[dict[str, Any]],
    options: ClaudeAgentOptions | None = None,
    transport: Transport | None = None
) -> AsyncIterator[Message]
```

## `ClaudeSDKClient`

```python
class ClaudeSDKClient:
    async def connect(self, prompt=None) -> None
    async def query(self, prompt, session_id="default") -> None
    async def receive_messages(self) -> AsyncIterator[Message]
    async def receive_response(self) -> AsyncIterator[Message]
    async def interrupt(self) -> None
    async def disconnect(self) -> None
    async def set_permission_mode(self, mode) -> None
    async def rewind_files(self, checkpoint_id) -> None
```

## `ClaudeAgentOptions`

| Champ | Type | Défaut | Description |
|-------|------|---------|-------------|
| `tools` | `list[str] | ToolsPreset` | None | Outils disponibles |
| `allowed_tools` | `list[str]` | `[]` | Outils auto-approuvés |
| `disallowed_tools` | `list[str]` | `[]` | Outils interdits |
| `system_prompt` | `str | SystemPromptPreset` | None | Prompt système |
| `mcp_servers` | `dict[str, McpServerConfig]` | `{}` | Serveurs MCP |
| `permission_mode` | `PermissionMode` | None | Mode de permission |
| `model` | `str` | None | Modèle Claude |
| `thinking` | `ThinkingConfig` | None | Configuration du raisonnement |
| `cwd` | `str | Path` | None | Répertoire de travail |
| `env` | `dict[str, str]` | `{}` | Variables d'environnement |
| `max_turns` | `int` | None | Tours maximum |
| `max_budget_usd` | `float` | None | Budget maximum USD |
| `effort` | `str` | None | Niveau d'effort |
| `resume` | `str` | None | ID de session à reprendre |
| `continue_conversation` | `bool` | False | Reprendre dernière session |
| `fork_session` | `bool` | False | Bifurquer la session |
| `setting_sources` | `list[str]` | Toutes | Sources de paramètres |
| `session_store` | `SessionStore` | None | Stockage de session externe |
| `enable_file_checkpointing` | `bool` | False | Points de contrôle de fichiers |
| `can_use_tool` | `Callable` | None | Callback d'approbation d'outils |
| `hooks` | `dict` | None | Hooks d'événements |
| `agents` | `dict` | None | Définitions de sous-agents |
| `skills` | `str | list[str]` | None | Skills à activer |
| `plugins` | `list[dict]` | None | Plugins à charger |

## `PermissionMode`

```python
PermissionMode = Literal[
    "default",
    "acceptEdits",
    "plan",
    "dontAsk",
    "bypassPermissions",
]
```

## Types de messages

```python
@dataclass
class UserMessage:
    content: str | list[ContentBlock]
    uuid: str | None = None
    parent_tool_use_id: str | None = None

@dataclass
class AssistantMessage:
    content: list[ContentBlock]
    model: str
    usage: dict[str, Any] | None = None

@dataclass
class ResultMessage:
    subtype: str  # success, error_max_turns, error_max_budget_usd, error_during_execution
    duration_ms: int
    is_error: bool
    num_turns: int
    session_id: str
    total_cost_usd: float | None = None
    usage: dict[str, Any] | None = None
    result: str | None = None
```

## Blocs de contenu

- `TextBlock` : texte brut
- `ThinkingBlock` : contenu de raisonnement étendu
- `ToolUseBlock` : invocation d'outil
- `ToolResultBlock` : résultats d'exécution d'outil

## Gestion des sessions

```python
async def list_sessions(options=None) -> list[SDKSessionInfo]
async def get_session_messages(session_id, options=None) -> list[SessionMessage]
async def get_session_info(session_id, options=None) -> SDKSessionInfo | None
async def rename_session(session_id, title, options=None) -> None
async def tag_session(session_id, tag, options=None) -> None
```

## Décorateur `@tool()`

```python
@tool("greet", "Greet a user", {"name": str})
async def greet(args: dict[str, Any]) -> dict[str, Any]:
    return {"content": [{"type": "text", "text": f"Hello, {args['name']}!"}]}
```

## `create_sdk_mcp_server()`

```python
def create_sdk_mcp_server(
    name: str,
    version: str = "1.0.0",
    tools: list[SdkMcpTool[Any]] | None = None
) -> McpSdkServerConfig
```

## Choisir entre `query()` et `ClaudeSDKClient`

- `query()` : chaque appel crée une session, adapté aux tâches ponctuelles
- `ClaudeSDKClient` : gère les IDs de session en interne entre les appels, adapté aux conversations continues multi-tours
