# MCP (Model Context Protocol)

Source: https://code.claude.com/docs/fr/mcp

## Ce que c'est

MCP = standard ouvert pour connecter les outils d'IA aux sources de données et services externes.
Les serveurs MCP donnent à Claude de nouveaux outils pour Slack, Jira, bases de données, navigateurs, etc.

## Ce que vous pouvez faire avec MCP

- Implémenter des fonctionnalités à partir de JIRA : "Add feature from issue ENG-4521 and create a PR"
- Analyser les données de surveillance : "Check Sentry and Statsig for this feature's usage"
- Interroger des bases de données : "Find emails of 10 users who used feature X from PostgreSQL"
- Intégrer des designs : "Update email template based on new Figma designs shared on Slack"

## Connexion d'un serveur MCP

### Via la CLI
```bash
claude mcp add <server-name> <command>
# Exemples :
claude mcp add sqlite uvx mcp-server-sqlite@latest database.db
claude mcp add memory npx @modelcontextprotocol/server-memory
```

### Via `.mcp.json` (partageable dans le projet)
```json
{
  "mcpServers": {
    "sqlite": {
      "command": "uvx",
      "args": ["mcp-server-sqlite@latest", "database.db"]
    }
  }
}
```

### Via `/mcp` dans une session
Gérer les serveurs MCP interactivement.

## Types de transport MCP

- **stdio** : le plus commun, Claude lance le serveur comme processus enfant
- **HTTP/SSE** : pour les serveurs distants
- **SSE uniquement** : serveurs SSE (Server-Sent Events)

## MCP Tool Search (économie de contexte)

Les schémas d'outils MCP sont différés jusqu'à utilisation. Seuls les noms d'outils se chargent au démarrage. Claude récupère le schéma complet à la demande.

Pour les serveurs inactifs : utiliser `CLAUDE_CODE_DISABLE_MCP_TOOL_SEARCH=1` pour revenir au chargement complet.

## Ressources MCP

Référencer les ressources avec `@server:resource` dans vos prompts :
```text
Show me the data from @github:repos/owner/repo/issues
@sqlite:query?sql=SELECT * FROM users LIMIT 10
```

## Serveurs MCP populaires

| Catégorie | Serveur | Usage |
|-----------|---------|-------|
| Fichiers | `@modelcontextprotocol/server-filesystem` | Accès système de fichiers |
| Mémoire | `@modelcontextprotocol/server-memory` | Mémoire persistante |
| GitHub | `@modelcontextprotocol/server-github` | Issues, PRs, repos |
| Slack | `@modelcontextprotocol/server-slack` | Messagerie |
| Base de données | `mcp-server-sqlite`, `mcp-server-postgres` | Requêtes SQL |
| Web | `@modelcontextprotocol/server-puppeteer` | Navigation web |

## Managed MCP servers (Enterprise)

Déployer des serveurs MCP à l'échelle de l'organisation via `managed-mcp.json`.

```json
{
  "allowedMcpServers": ["github", "jira"],
  "allowManagedMcpServersOnly": true
}
```

## Sécurité MCP

- Faire confiance aux serveurs MCP comme au code qu'ils exécutent
- Ne connecter que des serveurs connus et vérifiés
- Les serveurs MCP peuvent injecter des instructions dans le contexte
- Le mode auto protège contre certaines injections

## Démarr
 rapide MCP

Guide disponible à `/fr/mcp-quickstart` pour connecter le premier serveur de bout en bout.
