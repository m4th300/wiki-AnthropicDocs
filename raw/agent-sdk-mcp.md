# MCP dans le SDK Agent

> Configurez les serveurs MCP pour étendre votre agent avec des outils externes.

Le Model Context Protocol (MCP) est une norme ouverte pour connecter les agents IA aux outils externes et sources de données.

## Démarrage rapide

```typescript
for await (const message of query({
  prompt: "Use the docs MCP server to explain what hooks are",
  options: {
    mcpServers: {
      "claude-code-docs": {
        type: "http",
        url: "https://code.claude.com/docs/mcp"
      }
    },
    allowedTools: ["mcp__claude-code-docs__*"]
  }
}))
```

## Convention de nommage des outils MCP

Format: `mcp__{server-name}__{tool-name}`

Exemple: serveur `github`, outil `list_issues` → `mcp__github__list_issues`

## Types de transport

### Stdio (processus locaux)
```python
mcp_servers={
    "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {"GITHUB_TOKEN": os.environ["GITHUB_TOKEN"]},
    }
}
```

### HTTP/SSE (serveurs distants)
```python
mcp_servers={
    "remote-api": {
        "type": "sse",  # ou "http" pour HTTP streaming
        "url": "https://api.example.com/mcp/sse",
        "headers": {"Authorization": f"Bearer {os.environ['API_TOKEN']}"},
    }
}
```

### Serveurs MCP SDK (en processus)
Définir des outils personnalisés directement dans votre application. Voir guide des outils personnalisés.

## Autoriser les outils MCP

Sans autorisation, Claude voit les outils mais ne peut pas les appeler.

```python
allowed_tools=["mcp__github__*"]  # Tous les outils du serveur github
allowed_tools=["mcp__db__query"]  # Un outil spécifique
```

Note: `permissionMode: "acceptEdits"` n'approuve PAS automatiquement les outils MCP. Utiliser `allowedTools` avec wildcard.

## Configuration via fichier .mcp.json

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

Chargé quand la source `project` est activée dans `settingSources`.

## Découvrir les outils disponibles

```typescript
for await (const message of query({ prompt: "...", options })) {
  if (message.type === "system" && message.subtype === "init") {
    console.log("Available MCP tools:", message.mcp_servers);
  }
}
```

## Authentification OAuth2

Le SDK ne gère pas le flux OAuth automatiquement. Compléter le flux OAuth dans votre application, puis passer le token via les headers.

## Recherche d'outils MCP

Activée par défaut. Retient les définitions d'outils du contexte et charge uniquement ceux nécessaires.

Variable d'environnement `ENABLE_TOOL_SEARCH`:
- Non défini : activé (sauf sur Vertex AI ou proxy tiers)
- `true` : toujours activé
- `false` : désactivé, tous les outils chargés au démarrage
- `auto` : s'active si outils > 10% de la fenêtre de contexte
- `auto:N` : seuil personnalisé

## Gestion des erreurs

Vérifier le message `init` pour l'état de connexion:
```python
if isinstance(message, SystemMessage) and message.subtype == "init":
    failed_servers = [
        s for s in message.data.get("mcp_servers", [])
        if s.get("status") != "connected"
    ]
```

Causes courantes d'échec:
- Variables d'environnement manquantes
- Serveur non installé
- Chaîne de connexion invalide
- Problèmes réseau
