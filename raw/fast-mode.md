# Mode rapide (Fast Mode)

Source: https://code.claude.com/docs/fr/fast-mode

Note : Aperçu de recherche. Nécessite Claude Code v2.1.36+.

## Ce que c'est

Mode haute vitesse pour Claude Opus : jusqu'à 2,5 fois plus rapide à un coût par jeton plus élevé.
Même qualité et capacités, réponses plus rapides.

Supporté sur : Opus 4.8, Opus 4.7, Opus 4.6. PAS disponible sur Sonnet, Haiku, autres modèles.

## Activer/Désactiver

```text
/fast                          # Toggle dans la CLI
```

Ou définir dans les paramètres :
```json
{ "fastMode": true }
```

Icône `↯` apparaît à côté de l'invite quand actif.

Basculer vers Fast Mode → bascule automatiquement vers Opus si vous êtes sur un autre modèle.
Désactiver → reste sur Opus (pas de retour au modèle précédent).

## Tarification

| Modèle | Entrée (MTok) | Sortie (MTok) |
|--------|-------------|-------------|
| Opus 4.8 | $10 | $50 |
| Opus 4.7 et 4.6 | $30 | $150 |

Facturé via crédits d'utilisation (pas dans l'utilisation incluse du plan).

**Note** : La première activation dans une conversation → coût du cache invalide pour tout le contexte existant.
Activer dès le DÉBUT de la session = moins cher.

## Exigences

- API Anthropic ou abonnement Claude (PAS disponible sur Bedrock, Vertex, Azure Foundry, Claude Platform on AWS)
- Crédits d'utilisation activés
- Sur Team/Enterprise : admin doit l'activer dans la console d'administration

## Configuration admin

- Claude.ai Team/Enterprise → désactivé par défaut
- Activer via : Console → Claude Code Preferences, ou claude.ai/admin-settings/claude-code
- Désactiver complètement : `CLAUDE_CODE_DISABLE_FAST_MODE=1`

## Opt-in par session (admin)

```json
{ "fastModePerSessionOptIn": true }
```

Empêche le Fast Mode de persister entre les sessions. Utile pour contrôler les coûts.

## Gérer les limites de débit

Opus 4.8, 4.7 et 4.6 partagent le même pool de limites de débit en Fast Mode.
Si limite atteinte ou crédits épuisés → bascule automatiquement vers la vitesse standard (icône `↯` devient grise).
Réactivation automatique quand le cooldown expire.

## Fast Mode vs Niveau d'effort

| Paramètre | Effet |
|---------|-------|
| Fast Mode | Même qualité, latence inférieure, coût plus élevé |
| Niveau d'effort inférieur | Moins de réflexion, plus rapide, qualité potentiellement inférieure |

Combinaison possible : Fast Mode + effort low = vitesse maximale sur tâches simples.

## Note sur Opus 4.6

Le Fast Mode sur Opus 4.6 est déprécié et sera supprimé ~30 jours après le lancement d'Opus 4.8.
Migrer vers Opus 4.8 ou 4.7 pour conserver l'accélération.

## Fast Mode dans le prompt caching

Activer Fast Mode en cours de session invalide le cache d'invite (en-tête de requête change).
Pour minimiser les coûts : activer au DÉBUT de la session plutôt qu'en milieu.
