# Fonctionnement de la boucle d'agent SDK

> Comprenez le cycle de vie des messages, l'exécution des outils, la fenêtre de contexte et l'architecture.

## La boucle en un coup d'œil

1. **Recevoir le prompt** → SystemMessage (subtype: `init`) avec métadonnées
2. **Évaluer et répondre** → AssistantMessage avec texte et/ou appels d'outils
3. **Exécuter les outils** → UserMessage avec résultats d'outils
4. **Répéter** jusqu'à réponse sans appels d'outils
5. **Résultat** → AssistantMessage final + ResultMessage (coût, session_id, etc.)

## Tours

Un tour = aller-retour complet (Claude produit output incluant appels d'outils → SDK exécute → résultats renvoyés à Claude).

Limite avec `max_turns` (compte uniquement les tours d'utilisation d'outils) ou `max_budget_usd`.

## Types de messages

| Type | Quand | Contenu |
|------|-------|---------|
| `SystemMessage` | Démarrage + compaction | Subtype `init` ou `compact_boundary` |
| `AssistantMessage` | Après chaque réponse de Claude | Blocs texte + appels d'outils |
| `UserMessage` | Après chaque exécution d'outil | Résultats d'outils + entrées utilisateur |
| `StreamEvent` | Si messages partiels activés | Événements streaming bruts |
| `ResultMessage` | Fin de boucle | Résultat final, coût, session_id |

### Python: vérification avec isinstance()
```python
from claude_agent_sdk import AssistantMessage, ResultMessage

if isinstance(message, AssistantMessage):
    print(f"Turn: {len(message.content)} content blocks")
if isinstance(message, ResultMessage) and message.subtype == "success":
    print(message.result)
```

### TypeScript: vérification avec .type
```typescript
if (message.type === "assistant") {
  console.log(message.message.content.length);  // Notez: message.message.content
}
if (message.type === "result" && message.subtype === "success") {
  console.log(message.result);
}
```

## Sous-types de ResultMessage

| Sous-type | Succès | Champ `result` |
|-----------|--------|---------------|
| `success` | Oui | Oui |
| `error_max_turns` | Non | Non |
| `error_max_budget_usd` | Non | Non |
| `error_during_execution` | Non | Non |
| `error_max_structured_output_retries` | Non | Non |

Tous portent `total_cost_usd`, `usage`, `num_turns`, `session_id` (certains peuvent être None en Python).

## Outils intégrés

| Catégorie | Outils |
|-----------|--------|
| Opérations sur fichiers | `Read`, `Edit`, `Write` |
| Recherche | `Glob`, `Grep` |
| Exécution | `Bash` |
| Web | `WebSearch`, `WebFetch` |
| Découverte | `ToolSearch` (charge dynamiquement les outils à la demande) |
| Orchestration | `Agent`, `Skill`, `AskUserQuestion`, `TaskCreate`, `TaskUpdate` |

## Exécution parallèle des outils

- Outils en lecture seule (`Read`, `Glob`, `Grep` + outils MCP avec `readOnlyHint`) → peuvent s'exécuter en parallèle
- Outils modifiant l'état (`Edit`, `Write`, `Bash`) → séquentiels

## Contrôler la boucle

| Option | Contrôle | Défaut |
|--------|----------|--------|
| `max_turns` / `maxTurns` | Tours max d'utilisation d'outils | Pas de limite |
| `max_budget_usd` / `maxBudgetUsd` | Coût max USD | Pas de limite |
| `effort` | Profondeur du raisonnement | Python: non défini; TypeScript: `"high"` |

### Niveaux d'effort

| Niveau | Utilisation |
|--------|-------------|
| `"low"` | Fichiers simples, listage |
| `"medium"` | Éditions de routine |
| `"high"` | Refactorisations, débogage |
| `"xhigh"` | Codage complexe, agents (recommandé Opus 4.7) |
| `"max"` | Problèmes multi-étapes profonds |

`effort` ≠ Extended thinking. Indépendants l'un de l'autre.

## La fenêtre de contexte

Tout s'accumule: prompt système + définitions d'outils + historique + résultats d'outils.

### Ce qui consomme du contexte

| Source | Quand | Impact |
|--------|-------|--------|
| Prompt système | Chaque requête | Petit fixe (mis en cache) |
| CLAUDE.md | Démarrage, via settingSources | Contenu complet (mis en cache) |
| Définitions d'outils | Chaque requête (différé par défaut) | Schémas MCP différés par recherche d'outils |
| Historique conversation | S'accumule | Croît avec chaque tour |

### Compaction automatique

Quand la fenêtre approche sa limite, le SDK compacte en résumant l'historique plus ancien.

Émet `compact_boundary` dans le flux. Les instructions persistantes → CLAUDE.md (réinjecté chaque requête).

Personnaliser dans CLAUDE.md:
```markdown
# Summary instructions

When summarizing this conversation, always preserve:
- The current task objective
- File paths that have been modified
- Decisions made and the reasoning
```

## Exemple complet

```python
async for message in query(
    prompt="Find and fix the bug causing test failures in the auth module",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Bash", "Glob", "Grep"],
        setting_sources=["project"],
        max_turns=30,
        effort="high",
    ),
):
    if isinstance(message, ResultMessage):
        session_id = message.session_id
        if message.subtype == "success":
            print(f"Done: {message.result}")
        elif message.subtype == "error_max_turns":
            print(f"Hit turn limit. Resume {session_id} to continue.")
        if message.total_cost_usd is not None:
            print(f"Cost: ${message.total_cost_usd:.4f}")
```
