# Contrôler l'accès aux serveurs MCP pour votre organisation

Source: https://code.claude.com/docs/fr/managed-mcp

## Modèles de contrôle MCP

| Modèle | Ce qu'il fait | Configurer |
|--------|--------------|-----------|
| **Désactiver MCP** | Aucun serveur | `managed-mcp.json` avec carte vide |
| **Déploiement fixe** | Ensemble approuvé, pas d'ajout possible | `managed-mcp.json` avec les serveurs |
| **Catalogue approuvé** | Liste blanche, utilisateurs choisissent | `allowedMcpServers` + `allowManagedMcpServersOnly: true` |
| **Plugins uniquement** | MCP depuis plugins uniquement | `strictPluginOnlyCustomization` avec `mcp` |
| **Liste blanche souple** | Liste blanche extensible par utilisateurs | `allowedMcpServers` sans `allowManagedMcpServersOnly` |
| **Liste noire uniquement** | Bloquer certains serveurs | `deniedMcpServers` |
| **Aucune restriction** | Tout autorisé | Pas de config managed MCP |

## managed-mcp.json (contrôle exclusif)

Si `managed-mcp.json` est déployé → Claude Code charge UNIQUEMENT ces serveurs.
Les utilisateurs ne peuvent pas en ajouter d'autres (y compris via plugins).

Chemins :
- macOS : `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux/WSL : `/etc/claude-code/managed-mcp.json`
- Windows : `C:\Program Files\ClaudeCode\managed-mcp.json`

Format (même que `.mcp.json`) :
```json
{
  "mcpServers": {
    "github": {"type": "http", "url": "https://api.githubcopilot.com/mcp/"},
    "company-internal": {
      "type": "stdio",
      "command": "/usr/local/bin/company-mcp-server",
      "args": ["--config", "/etc/company/mcp-config.json"]
    }
  }
}
```

### Authentification par utilisateur

Ne pas stocker les clés API dans `managed-mcp.json` (lisible par tous).
Utiliser :
- Expansion `${VAR}` pour lire depuis l'environnement de l'utilisateur
- OAuth ou en-têtes par utilisateur
- `headersHelper` pour générer des identifiants au moment de la connexion

### Désactiver MCP entièrement

```json
{"mcpServers": {}}
```

### Autoriser les connecteurs claude.ai aux côtés de l'ensemble géré

```json
{"allowAllClaudeAiMcps": true}
```
Nécessite Claude Code v2.1.149+. Lisible uniquement depuis les sources de politique administrateur.

### Valider

```bash
claude mcp list                # Doit montrer uniquement les serveurs managed
claude mcp add --transport http test https://example.com/mcp  # Doit échouer
```

## Listes blanches et listes noires

Ne sont pas un registre → filtrent les serveurs configurés.

Correspondance :
| Clé | Correspond à |
|-----|-------------|
| `serverUrl` | URL avec wildcards `*` |
| `serverCommand` | Commande exacte + arguments |
| `serverName` | Étiquette utilisateur (attention : pas sûr seul) |

Ordre d'évaluation :
1. Fusionner les listes
2. Vérifier la liste noire (priorité absolue)
3. Vérifier la liste blanche

**⚠️ Ne pas utiliser `serverName` seul pour la sécurité** — les utilisateurs peuvent nommer n'importe quel serveur.

Exemple de configuration stricte :
```json
{
  "allowedMcpServers": [
    {"serverUrl": "https://api.githubcopilot.com/*"},
    {"serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]}
  ],
  "deniedMcpServers": [
    {"serverUrl": "https://*.untrusted.example.com/*"}
  ]
}
```

## Rendre la liste blanche faisant autorité

```json
{"allowManagedMcpServersOnly": true}
```

+ `allowedMcpServers` dans les paramètres gérés = liste blanche impossible à élargir par les utilisateurs.
La liste noire fusionne toujours de toutes les sources.

## Comment les restrictions apparaissent aux utilisateurs

| Restriction | Message |
|------------|---------|
| managed-mcp.json présent | `Cannot add MCP server: enterprise MCP configuration is active` |
| Serveur sur liste noire | `Cannot add MCP server "<name>": server is explicitly blocked by enterprise policy` |
| Serveur hors liste blanche | `Cannot add MCP server "<name>": not allowed by enterprise policy` |
| Serveur précédemment configuré maintenant bloqué | Disparaît silencieusement → informer les utilisateurs |

## Surveiller l'utilisation MCP

`OTEL_LOG_TOOL_DETAILS=1` + export OpenTelemetry = voir les serveurs que les utilisateurs invoquent.
