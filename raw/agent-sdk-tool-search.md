# Recherche d'outils dans le SDK Agent

> Adaptez votre agent à des milliers d'outils en découvrant et chargeant uniquement ce qui est nécessaire.

## Problème résolu

- 50 outils = ~10-20K tokens de fenêtre de contexte
- Précision de sélection se dégrade avec >30-50 outils chargés simultanément

## Fonctionnement

1. Les définitions d'outils sont retenues du contexte
2. L'agent reçoit un résumé des outils disponibles
3. Recherche les outils pertinents quand nécessaire (3-5 résultats)
4. Les outils chargés restent disponibles pour les tours suivants
5. Si compaction → outils supprimés, l'agent recherche à nouveau

Trade-off: aller-retour supplémentaire à la première découverte, mais contexte plus petit à chaque tour.

Note: Avec <10 outils, charger tout au départ est généralement plus rapide.

## Modèles supportés

- Claude Sonnet 4 ou version ultérieure
- Claude Opus 4 ou version ultérieure
- Pas Haiku

## Configuration via `ENABLE_TOOL_SEARCH`

| Valeur | Comportement |
|--------|--------------|
| Non défini | Activé (désactivé par défaut sur Vertex AI ou proxy tiers) |
| `true` | Toujours activé (peut échouer sur Vertex AI avant Sonnet 4.5) |
| `auto` | S'active si outils > 10% de la fenêtre de contexte |
| `auto:N` | S'active si outils > N% (exemple: `auto:5`) |
| `false` | Désactivé, tous les outils chargés au démarrage |

```python
options = ClaudeAgentOptions(
    mcp_servers={"enterprise-tools": {"type": "http", "url": "https://..."}},
    allowed_tools=["mcp__enterprise-tools__*"],
    env={"ENABLE_TOOL_SEARCH": "auto:5"},
)
```

S'applique à tous les outils: serveurs MCP distants ET serveurs MCP SDK personnalisés.

## Optimiser la découverte

- Noms descriptifs: `search_slack_messages` > `query_slack`
- Descriptions avec mots-clés spécifiques: "Rechercher les messages Slack par mot-clé, canal ou plage de dates"
- Message système listant les catégories: "You can search for tools to interact with Slack, GitHub, and Jira."

## Limites

- Maximum 10 000 outils dans le catalogue
- 3-5 outils par recherche
- Modèles: Sonnet 4+, Opus 4+ (pas Haiku)
