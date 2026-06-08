# Se connecter aux serveurs MCP (démarrage rapide)

Source: https://code.claude.com/docs/fr/mcp-quickstart

## Ajouter un serveur MCP (processus général)

```bash
# Serveur HTTP
claude mcp add --transport http claude-code-docs https://code.claude.com/docs/mcp

# Serveur local (stdio)
claude mcp add playwright -- npx -y @playwright/mcp@latest

# Vérifier l'état
claude mcp list

# Supprimer
claude mcp remove claude-code-docs
```

## Portées de configuration

| Portée | Fichier | Disponible pour |
|--------|---------|----------------|
| `local` (défaut) | `~/.claude.json`, entrée du projet | Vous, ce projet uniquement |
| `project` | `.mcp.json` racine du projet | Tous les collaborateurs |
| `user` | `~/.claude.json`, clé `mcpServers` | Vous, tous les projets |

```bash
claude mcp add --scope user --transport http <name> <url>
claude mcp add --scope project --transport http <name> <url>
```

## Format JSON direct (.mcp.json)

```json
{
  "mcpServers": {
    "claude-code-docs": {
      "type": "http",
      "url": "https://code.claude.com/docs/mcp"
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

## Serveurs qui nécessitent une connexion OAuth

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
# Puis dans Claude : /mcp → sélectionner → Authenticate
```

Serveurs avec jeton statique :
```bash
claude mcp add --transport http github <url> --header "Authorization: Bearer <token>"
```

## Connexion depuis d'autres surfaces

- **Desktop** : interface connecteurs
- **Claude Desktop (app chat)** : `claude mcp add-from-claude-desktop` sur macOS/WSL
- **VS Code** : voir VS Code → Connect external tools
- **claude.ai** : connecteurs sur claude.ai/customize/connectors → chargés automatiquement dans la CLI

## États de connexion

| Statut | Signification |
|--------|------------|
| `✓ Connected` | Prêt à utiliser |
| `! Needs authentication` | Connexion OAuth requise |
| `✗ Failed to connect` | Ne répond pas |
| `⏸ Pending approval` | À portée projet, pas encore approuvé |

## Dépannage

**Aucun serveur configuré** : vérifier que vous êtes dans le bon projet.
**Failed to connect (HTTP)** : `curl -I <url>` pour tester. 404/405 = normal pour POST-only.
**Timeout** : `MCP_TIMEOUT=60000 claude` pour les serveurs qui mettent longtemps à démarrer.
**Serveur stdio ne démarre pas** : exécuter la commande directement dans le terminal pour voir l'erreur.
**Changes .mcp.json non appliqués** : relancer Claude Code + `claude mcp reset-project-choices` si nécessaire.

## Note sur les coûts de contexte

Chaque serveur MCP connecté prend de l'espace dans la context window (noms d'outils, instructions).
Supprimer les serveurs non utilisés libère cet espace.
