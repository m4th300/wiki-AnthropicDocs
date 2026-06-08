# Configuration du modèle

Source: https://code.claude.com/docs/fr/model-config

## Alias de modèle

| Alias | Comportement |
|-------|-------------|
| `default` | Efface tout remplacement, revient au modèle recommandé pour votre compte |
| `best` | Modèle le plus capable (= `opus` actuellement) |
| `sonnet` | Dernier Sonnet (tâches de codage quotidiennes) |
| `opus` | Dernier Opus (raisonnement complexe) |
| `haiku` | Rapide et efficace pour les tâches simples |
| `sonnet[1m]` | Sonnet avec context window de 1M tokens |
| `opus[1m]` | Opus avec context window de 1M tokens |
| `opusplan` | Opus en plan mode → bascule vers Sonnet pour l'exécution |

### Résolution des alias par fournisseur

| Fournisseur | `opus` → | `sonnet` → |
|-------------|---------|-----------|
| API Anthropic | Opus 4.8 | Sonnet 4.6 |
| Claude Platform on AWS | Opus 4.7 | Sonnet 4.6 |
| Bedrock/Vertex/Foundry | Opus 4.6 | Sonnet 4.5 |

Pour épingler une version : utiliser le nom complet (ex: `claude-opus-4-8`) ou `ANTHROPIC_DEFAULT_OPUS_MODEL`.

## Définir le modèle

Priorité :
1. `/model <alias|name>` pendant la session (enregistre par défaut dans paramètres utilisateur)
2. `claude --model <alias|name>` au démarrage (session uniquement)
3. `ANTHROPIC_MODEL=<alias|name>` (session uniquement)
4. `settings.json` : champ `model` (permanent)

Depuis v2.1.153 : `/model` enregistre comme défaut de la session (`Enter`), ou `s` pour cette session uniquement.

## Comportement par défaut du modèle

| Plan | Modèle par défaut |
|------|------------------|
| Max, Team Premium, Enterprise pay-as-you-go, API Anthropic | Opus 4.8 |
| Claude Platform on AWS | Opus 4.7 |
| Pro, Team Standard, sièges d'abonnement Enterprise | Sonnet 4.6 |
| Bedrock, Vertex, Foundry | Sonnet 4.5 |

## `opusplan` : mode hybride

- En plan mode → utilise Opus (raisonnement complexe, décisions architecturales)
- En mode exécution → bascule vers Sonnet (génération de code, implémentation)
- Phase Opus : fenêtre 200K standard (la mise à niveau 1M automatique ne s'applique pas)

## Niveaux d'effort (effort level)

Contrôle le raisonnement adaptatif. Modèles supportés :

| Modèle | Niveaux |
|--------|---------|
| Opus 4.8 et 4.7 | low, medium, high, xhigh, max |
| Opus 4.6 et Sonnet 4.6 | low, medium, high, max |

Défauts : `high` sur Opus 4.8, Opus 4.6 et Sonnet 4.6 ; `xhigh` sur Opus 4.7.

Si niveau non supporté → revient au niveau le plus élevé supporté au-dessous.

| Niveau | Quand l'utiliser |
|--------|-----------------|
| `low` | Tâches courtes, sensibles à la latence, non-critiques |
| `medium` | Réduire les coûts pour le travail sensible |
| `high` | Équilibre (défaut Opus 4.8/4.6, Sonnet 4.6) |
| `xhigh` | Raisonnement plus profond (défaut Opus 4.7) |
| `max` | Session uniquement, rendements décroissants possibles |
| `ultracode` | Session uniquement, planifie des dynamic workflows |

### Définir le niveau d'effort

- `/effort` → curseur interactif
- `/effort high` → niveau direct
- `/effort auto` → réinitialise au défaut
- `--effort` flag au démarrage
- `CLAUDE_CODE_EFFORT_LEVEL` (priorité maximale)
- `effortLevel` dans settings.json (low/medium/high/xhigh uniquement)

### `ultrathink`

Inclure "ultrathink" dans une invite → raisonnement plus profond à ce tour uniquement, sans changer le niveau de session.

## Réflexion étendue

- `Option+T` (Mac) / `Alt+T` (Win/Linux) : basculer pour la session
- `/config` : définir le défaut global (`alwaysThinkingEnabled`)
- `MAX_THINKING_TOKENS=0` : désactiver indépendamment de l'effort
- `Ctrl+O` : voir le raisonnement en texte gris italique

Sur Opus 4.7+ : raisonnement adaptatif (réfléchit quand pertinent). `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1` pour revenir au budget fixe (Opus 4.6/Sonnet 4.6 uniquement).

## Contexte étendu (1M tokens)

Disponibilité :
| Plan | Opus 1M | Sonnet 1M |
|------|---------|-----------|
| Max, Team, Enterprise | Inclus dans l'abo | Nécessite crédits d'utilisation |
| Pro | Nécessite crédits | Nécessite crédits |
| API et pay-as-you-go | Accès complet | Accès complet |

Utilisation :
```bash
/model opus[1m]
/model claude-opus-4-8[1m]
```

Désactiver complètement : `CLAUDE_CODE_DISABLE_1M_CONTEXT=1`

## Restreindre la sélection (entreprise)

```json
{ "availableModels": ["sonnet", "haiku"] }
```

L'option "Par défaut" reste toujours disponible même avec `availableModels: []`.

## Variables d'environnement de modèle

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Modèle pour alias `opus` / plan mode `opusplan` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Modèle pour alias `sonnet` / exécution `opusplan` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Modèle pour alias `haiku` / fonctionnalités arrière-plan |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Modèle pour tous les subagents et agent teams |
| `ANTHROPIC_MODEL` | Modèle pour la session lancée avec cette variable |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | Ajouter une option personnalisée au sélecteur `/model` |

## Épingler les modèles pour déploiements tiers

Pour Bedrock/Vertex/Foundry : épingler les versions avant déploiement.

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='us.anthropic.claude-opus-4-8'   # Bedrock
export ANTHROPIC_DEFAULT_SONNET_MODEL='claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='claude-haiku-4-5'
```

Pour activer 1M avec modèle épinglé :
```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='claude-opus-4-8[1m]'
```

## Mise en cache des invites (prompt caching)

Activée automatiquement. Désactiver :
```
DISABLE_PROMPT_CACHING=1                # tous les modèles
DISABLE_PROMPT_CACHING_HAIKU=1          # Haiku uniquement
DISABLE_PROMPT_CACHING_SONNET=1         # Sonnet uniquement
DISABLE_PROMPT_CACHING_OPUS=1           # Opus uniquement
```
