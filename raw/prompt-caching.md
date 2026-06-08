# Comment Claude Code utilise le prompt caching

Source: https://code.claude.com/docs/fr/prompt-caching

## Principe de base

Sans caching : l'API retraite tout l'historique à chaque tour.
Avec caching : réutilise le contenu déjà traité, ne retraite que ce qui est nouveau.

Le cache fonctionne par correspondance de **préfixe exact** au début de chaque requête.

## Structure des couches du cache

| Couche | Contenu | Change quand |
|--------|---------|-------------|
| Prompt système | Instructions principales, définitions d'outils, style de sortie | Ensemble des outils chargés change, ou mise à jour Claude Code |
| Contexte du projet | CLAUDE.md, mémoire auto, règles non délimitées | Démarrage de session, ou après `/clear` ou `/compact` |
| Conversation | Vos messages, réponses de Claude, résultats d'outils | À chaque tour |

**Règle** : un changement dans une couche invalide tout ce qui suit. Prompt système changé → tout recalcule.

Le modèle et le niveau d'effort font partie de la clé de cache (pas du texte du prompt).

## Actions qui INVALIDENT le cache

| Action | Pourquoi |
|--------|---------|
| Changer de modèle (`/model`) | Chaque modèle a son propre cache |
| Modifier le niveau d'effort (`/effort`) | Indexé par effort + modèle |
| Activer le mode rapide | Ajoute un en-tête de requête dans la clé |
| Connecter/déconnecter un serveur MCP | Modifie les définitions d'outils (si pas différés) |
| Activer/désactiver un plugin MCP | Même raison que MCP |
| Refuser un outil entier (`Bash`, `WebFetch`) | Supprime l'outil du prompt système |
| Compacter la conversation (`/compact`) | Remplace l'historique de messages |
| Mettre à niveau Claude Code | Met à jour le prompt système |

**Choisir modèle et effort AU DÉBUT de la session** pour maximiser les cache hits.

### MCP et cache

- Outils **différés** (MCP Tool Search, par défaut) : connexion/déconnexion n'invalide pas le cache
- Outils **chargés dans le préfixe** : tout changement invalide le cache
  - Cas où les outils ne sont pas différés : Haiku, Vertex AI, passerelle `ANTHROPIC_BASE_URL`, ou serveur `alwaysLoad`

### Mode rapide et cache

- Activer en cours de session → invalide le cache (en-tête de requête change)
- Désactiver/réactiver par la suite → conserve le cache
- Nécessite Claude Code v2.1.86+ pour le comportement de conservation

## Actions qui CONSERVENT le cache

| Action | Raison |
|--------|--------|
| Éditer des fichiers dans le repo | Contenu lit que lors de la lecture par Claude (après la conversation) |
| Éditer CLAUDE.md en cours de session | Chargé une fois au démarrage, pas rechargé en cours |
| Changer le style de sortie | Fait partie du prompt système, chargé une fois au démarrage |
| Changer le mode de permission | Ne change pas le prompt système ni les outils |
| Invoquer des skills et commandes | S'ajoutent à la fin de la conversation |
| Exécuter `/recap` | Ajoute comme sortie de commande (pas de remplacement) |
| Rembobiner la conversation | Revient à un préfixe déjà en cache |
| Générer un sous-agent | Démarre son propre cache séparé |

## Durée de vie du cache (TTL)

| Situation | TTL |
|-----------|-----|
| Abonnement Claude | 1 heure (automatique, inclus dans le plan) |
| Si crédits d'utilisation dépassés | 5 minutes (baisse automatique) |
| Clé API, Bedrock, Vertex, Foundry | 5 minutes (par défaut) |

Opter pour 1 heure sur API/cloud : `ENABLE_PROMPT_CACHING_1H=1`
Forcer 5 minutes : `FORCE_PROMPT_CACHING_5M=1`

## Portée du cache

Cache effectivement limité à une machine et un répertoire (le prompt système intègre le répertoire, la plateforme, le shell, etc.).

Deux sessions dans des répertoires différents → préfixes différents → pas de partage de cache.
Sessions parallèles dans le même répertoire → peuvent partager le cache.

## Sous-agents et cache

Les sous-agents démarrent leur propre cache séparé.
Un **fork** (subagent qui hérite du contexte du parent) lit le cache du parent.

Les sous-agents utilisent le TTL de 5 minutes même sur un abonnement.

## Vérifier les performances du cache

Deux comptages dans la réponse API :
- `cache_creation_input_tokens` : tokens écrits dans le cache (facturés au taux d'écriture)
- `cache_read_input_tokens` : tokens servis depuis le cache (~10% du taux d'entrée standard)

Ratio lecture/création élevé → caching fonctionne bien.
Création élevée tour après tour → quelque chose change dans le préfixe.

## Désactiver le prompt caching

```
DISABLE_PROMPT_CACHING=1           # tous les modèles
DISABLE_PROMPT_CACHING_HAIKU=1     # Haiku uniquement
DISABLE_PROMPT_CACHING_SONNET=1    # Sonnet uniquement
DISABLE_PROMPT_CACHING_OPUS=1      # Opus uniquement
```
