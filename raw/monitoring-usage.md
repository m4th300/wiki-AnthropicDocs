# Surveillance (OpenTelemetry)

Source: https://code.claude.com/docs/fr/monitoring-usage

## Configuration rapide

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer your-token"
export OTEL_METRIC_EXPORT_INTERVAL=60000   # 60 secondes (défaut)
export OTEL_LOGS_EXPORT_INTERVAL=5000      # 5 secondes (défaut)
```

## Pour les administrateurs (dans managed-settings.json)

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.example.com:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer example-token"
  }
}
```

Note : Claude Code ne propage PAS les variables OTEL aux sous-processus.

## Attributs standard (présents sur toutes les métriques et événements)

| Attribut | Description |
|---------|-------------|
| `session.id` | Identifiant de session unique |
| `organization.id` | UUID de l'organisation |
| `user.account_uuid` | UUID du compte |
| `user.email` | Email de l'utilisateur (si OAuth) |
| `terminal.type` | Type de terminal (iTerm, vscode, cursor, tmux) |
| `OTEL_RESOURCE_ATTRIBUTES` | Attributs personnalisés |

## Métriques exportées

| Métrique | Description | Unité |
|---------|-------------|-------|
| `claude_code.session.count` | Sessions CLI démarrées | count |
| `claude_code.lines_of_code.count` | Lignes de code modifiées | count |
| `claude_code.pull_request.count` | PRs créées | count |
| `claude_code.commit.count` | Commits git créés | count |
| `claude_code.cost.usage` | Coût de la session | USD |
| `claude_code.token.usage` | Tokens utilisés | tokens |
| `claude_code.active_time.total` | Temps actif total | s |

## Événements exportés (via OTEL logs)

| Événement | Description |
|---------|-------------|
| `claude_code.user_prompt` | Prompt soumis |
| `claude_code.tool_result` | Exécution d'outil terminée |
| `claude_code.api_request` | Requête API à Claude |
| `claude_code.api_error` | Erreur API |
| `claude_code.tool_decision` | Décision de permission |
| `claude_code.permission_mode_changed` | Changement de mode |
| `claude_code.auth` | Login/logout |
| `claude_code.mcp_server_connection` | État de connexion MCP |
| `claude_code.feedback_survey` | Sondage de qualité |

## Variables de contrôle supplémentaires

| Variable | Description |
|---------|-------------|
| `OTEL_LOG_USER_PROMPTS=1` | Journaliser le contenu des prompts |
| `OTEL_LOG_TOOL_DETAILS=1` | Journaliser les paramètres des outils |
| `OTEL_LOG_TOOL_CONTENT=1` | Journaliser les entrées/sorties des outils |
| `OTEL_LOG_RAW_API_BODIES=1` | Journaliser les corps JSON complets |

## Traces (bêta)

```bash
export CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1
export OTEL_TRACES_EXPORTER=otlp
```

Hiérarchie des spans :
```
claude_code.interaction
├── claude_code.llm_request
├── claude_code.hook
└── claude_code.tool
    ├── claude_code.tool.blocked_on_user
    └── claude_code.tool.execution
```

## Support multi-équipes

```bash
export OTEL_RESOURCE_ATTRIBUTES="department=engineering,team.id=platform,cost_center=eng-123"
```

Format : pas d'espaces, clés=valeurs séparées par virgules.

## Configurations exemples

```bash
# Console (debug)
export OTEL_METRICS_EXPORTER=console
export OTEL_METRIC_EXPORT_INTERVAL=1000

# Prometheus
export OTEL_METRICS_EXPORTER=prometheus

# SIEM
export OTEL_LOG_TOOL_DETAILS=1
export OTEL_EXPORTER_OTLP_LOGS_ENDPOINT=https://siem.example.com:4318/v1/logs
```

## Corrélation des événements

Utiliser `prompt.id` pour lier tous les événements produits pour un seul prompt utilisateur.

## Audit de sécurité

Les événements OTel sont la source d'audit pour l'activité Claude Code :
- Attribution utilisateur via `user.email`, `user.account_uuid`, `organization.id`
- Activité MCP auditée avec `OTEL_LOG_TOOL_DETAILS=1`
- Intégration SIEM via `OTEL_EXPORTER_OTLP_LOGS_ENDPOINT`
