# Référence SDK Agent TypeScript

## Installation

```bash
npm install @anthropic-ai/claude-agent-sdk
```

Le SDK inclut un binaire Claude Code natif. Si votre gestionnaire de paquets ignore les dépendances optionnelles, définissez `pathToClaudeCodeExecutable`.

## Fonctions principales

### `query()`

Point d'entrée principal. Retourne un async generator qui streame les messages.

```typescript
function query({
  prompt,
  options
}: {
  prompt: string | AsyncIterable<SDKUserMessage>;
  options?: Options;
}): Query;
```

### `startup()`

Préconfigure le sous-processus CLI (réduit la latence de démarrage).

```typescript
function startup(params?: {
  options?: Options;
  initializeTimeoutMs?: number;
}): Promise<WarmQuery>;
```

### `tool()`

Crée un outil MCP type-safe.

```typescript
function tool<Schema extends AnyZodRawShape>(
  name: string,
  description: string,
  inputSchema: Schema,
  handler: (args: InferShape<Schema>, extra: unknown) => Promise<CallToolResult>,
  extras?: { annotations?: ToolAnnotations }
): SdkMcpToolDefinition<Schema>;
```

### `createSdkMcpServer()`

Crée un serveur MCP en processus.

```typescript
function createSdkMcpServer(options: {
  name: string;
  version?: string;
  tools?: Array<SdkMcpToolDefinition<any>>;
}): McpSdkServerConfigWithInstance;
```

### Gestion des sessions

- `listSessions(options?)` : liste les sessions passées
- `getSessionMessages(sessionId, options?)` : lit les messages d'une session
- `getSessionInfo(sessionId, options?)` : métadonnées d'une session
- `renameSession(sessionId, title, options?)` : renomme une session
- `tagSession(sessionId, tag, options?)` : tague une session

### `resolveSettings()`

Résout les paramètres Claude Code effectifs pour un répertoire.

## Options (type `Options`)

| Propriété | Type | Par défaut | Description |
|-----------|------|---------|-------------|
| `abortController` | `AbortController` | Nouvelle instance | Annuler les opérations |
| `additionalDirectories` | `string[]` | `[]` | Répertoires supplémentaires |
| `agents` | `Record<string, AgentDefinition>` | - | Sous-agents programmatiques |
| `allowedTools` | `string[]` | `[]` | Outils auto-approuvés |
| `cwd` | `string` | `process.cwd()` | Répertoire de travail |
| `effort` | `'low'|'medium'|'high'|'xhigh'|'max'` | `'high'` | Niveau de raisonnement |
| `enableFileCheckpointing` | `boolean` | `false` | Suivi des changements de fichiers |
| `maxBudgetUsd` | `number` | - | Coût maximum en USD |
| `maxTurns` | `number` | - | Nombre maximum de tours |
| `mcpServers` | `Record<string, McpServerConfig>` | `{}` | Serveurs MCP |
| `model` | `string` | Défaut CLI | Modèle Claude |
| `permissionMode` | `PermissionMode` | `'default'` | Mode de permission |
| `persistSession` | `boolean` | `true` | Persister la session sur disque |
| `resume` | `string` | - | ID de session à reprendre |
| `sessionId` | `string` | Auto | UUID spécifique pour la session |
| `settingSources` | `SettingSource[]` | Toutes | Sources de paramètres à charger |
| `systemPrompt` | `string | preset` | Minimal | Configuration du prompt système |
| `thinking` | `ThinkingConfig` | `{type:'adaptive'}` | Configuration du raisonnement |

## Type `AgentDefinition`

```typescript
type AgentDefinition = {
  description: string;           // Quand utiliser ce sous-agent
  tools?: string[];              // Outils autorisés
  disallowedTools?: string[];    // Outils interdits
  prompt: string;                // Prompt système
  model?: string;                // Surcharge du modèle
  mcpServers?: AgentMcpServerSpec[];
  skills?: string[];
  initialPrompt?: string;
  maxTurns?: number;
  background?: boolean;          // Tâche de fond non bloquante
  memory?: "user" | "project" | "local";
  effort?: "low" | "medium" | "high" | "xhigh" | "max" | number;
  permissionMode?: PermissionMode;
};
```

## Types de messages

- `system` (subtype `init` ou `compact_boundary`) : événements du cycle de vie
- `assistant` : réponses de Claude avec blocs de contenu
- `user` : résultats d'outils et entrées utilisateur
- `result` : fin de boucle avec sous-types: `success`, `error_max_turns`, `error_max_budget_usd`, `error_during_execution`

## `PermissionMode`

```typescript
type PermissionMode =
  | "default"
  | "acceptEdits"
  | "bypassPermissions"
  | "plan"
  | "multiStep"
  | "free";
```

## `SettingSource`

| Valeur | Description | Emplacement |
|--------|-------------|-------------|
| `'user'` | Paramètres globaux utilisateur | `~/.claude/settings.json` |
| `'project'` | Paramètres projet partagés | `.claude/settings.json` |
| `'local'` | Paramètres projet locaux | `.claude/settings.local.json` |

## Variables d'environnement importantes

- `API_TIMEOUT_MS` : timeout par requête (défaut: 600000)
- `CLAUDE_CODE_MAX_RETRIES` : tentatives max API (défaut: 10)
- `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` : timeout stagnation (défaut: 600000)
- `CLAUDE_ENABLE_STREAM_WATCHDOG=1` : détection inactivité stream
- `ENABLE_TOOL_SEARCH` : contrôle la recherche d'outils (`true`/`false`/`auto`/`auto:N`)
