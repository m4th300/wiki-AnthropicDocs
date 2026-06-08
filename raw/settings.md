# Paramètres Claude Code

Source: https://code.claude.com/docs/fr/settings

## Hiérarchie des scopes

| Scope | Emplacement | Priorité |
|-------|------------|---------|
| Managed (highest) | `/Library/Application Support/ClaudeCode/`, `/etc/claude-code/`, `C:\Program Files\ClaudeCode\` | 1 |
| CLI args | Session uniquement | 2 |
| Local | `.claude/settings.local.json` | 3 |
| Project | `.claude/settings.json` | 4 |
| User (lowest) | `~/.claude/settings.json` | 5 |

Arrays fusionnent entre les niveaux. Scalaires : couche supérieure gagne.

## Fichiers de configuration

- **Paramètres utilisateur** : `~/.claude/settings.json`
- **Paramètres projet** (partagés) : `.claude/settings.json`
- **Paramètres locaux** (gitignored) : `.claude/settings.local.json`
- **État OAuth + MCP** : `~/.claude.json`
- **Serveurs MCP projet** : `.mcp.json`

## Paramètres clés

| Paramètre | Description | Exemple |
|-----------|-------------|---------|
| `agent` | Subagent principal par défaut | `"code-reviewer"` |
| `alwaysThinkingEnabled` | Extended thinking par défaut | `true` |
| `autoMemoryEnabled` | Activer/désactiver auto memory | `true` |
| `autoMode` | Règles personnalisées du classificateur auto mode | `{"soft_deny": ["Never run terraform apply"]}` |
| `availableModels` | Restreindre les modèles disponibles via `/model` | `["sonnet", "haiku"]` |
| `defaultShell` | Shell pour commandes `!` | `"powershell"` |
| `disableAgentView` | Désactiver background agents | `true` |
| `disableAllHooks` | Désactiver tous les hooks | `true` |
| `disableAutoMode` | Empêcher activation auto mode | `"disable"` |
| `editorMode` | Keybindings : `"normal"` ou `"vim"` | `"vim"` |
| `effortLevel` | Niveau de réflexion persistant | `"xhigh"` |
| `env` | Variables d'environnement de session | `{"FOO": "bar"}` |
| `forceLoginMethod` | Restreindre login : `"claudeai"` ou `"console"` | `"claudeai"` |
| `language` | Langue de réponse et dictée | `"japanese"` |
| `model` | Modèle par défaut | `"claude-sonnet-4-6"` |
| `outputStyle` | Style de sortie | `"Explanatory"` |
| `permissions` | Règles allow/deny | Voir structure |
| `showThinkingSummaries` | Afficher résumés extended thinking | `true` |

## Exemple settings.json

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp"
  }
}
```

## Paramètres managed seulement

| Paramètre | Description |
|-----------|-------------|
| `allowManagedPermissionRulesOnly` | Seulement les règles managed s'appliquent |
| `allowManagedMcpServersOnly` | Seulement les MCP servers managed |
| `allowManagedHooksOnly` | Seulement les hooks managed |
| `strictPluginOnlyCustomization` | Bloquer user/project skills/agents/hooks/MCP |
| `claudeMd` | CLAUDE.md géré via settings.json |
| `policyHelper` | Exécutable calculant les settings managed dynamiquement |
| `forceRemoteSettingsRefresh` | Bloquer démarrage jusqu'à remote settings fetchés |

## Drop-in directory pour managed settings

```
/etc/claude-code/managed-settings.json
/etc/claude-code/managed-settings.d/10-telemetry.json
/etc/claude-code/managed-settings.d/20-security.json
```

Les fichiers dans `managed-settings.d/` sont fusionnés alphabétiquement. Arrays concaténés et dédupliqués.

## Table des emplacements par scope

| Feature | User | Project | Local |
|---------|------|---------|-------|
| Settings | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |
| Subagents | `~/.claude/agents/` | `.claude/agents/` | - |
| CLAUDE.md | `~/.claude/CLAUDE.md` | `CLAUDE.md` | `CLAUDE.local.md` |
| MCP servers | `~/.claude.json` | `.mcp.json` | `~/.claude.json` |

Sur Windows : `~/.claude` = `%USERPROFILE%\.claude`
