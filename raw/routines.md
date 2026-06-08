# Automatiser le travail avec les routines

Source: https://code.claude.com/docs/fr/routines

Note : Les routines sont en aperçu de recherche.

## Ce que c'est

Une routine = configuration Claude Code enregistrée (invite + référentiels + connecteurs) qui s'exécute automatiquement sur l'infrastructure cloud Anthropic.

Continue même quand votre ordinateur est éteint. Disponible sur Pro, Max, Team, Enterprise avec Claude Code sur le web activé.

## Types de déclencheurs

- **Planifié** : cadence récurrente (chaque heure, nuit, semaine) ou exécution unique
- **API** : HTTP POST avec jeton porteur
- **GitHub** : événements repo (PR ouvertes, versions, etc.)

Une routine peut combiner plusieurs types de déclencheurs.

Gérer sur : [claude.ai/code/routines](https://claude.ai/code/routines) ou `/schedule` dans la CLI.

## Exemples de cas d'usage

- **Maintenance du carnet** : déclencheur nuit → lit les issues, applique étiquettes, assigne propriétaires, résumé Slack
- **Triage des alertes** : outil de monitoring → appel API → routine ouvre une PR brouillon avec correctif proposé
- **Examen de code** : déclencheur GitHub PR.opened → applique checklist, laisse commentaires en ligne
- **Vérification déploiement** : pipeline CD → appel API → smoke checks, analyse logs
- **Dérive documentation** : déclencheur hebdo → signale docs obsolètes, ouvre PRs de mise à jour
- **Portage bibliothèque** : PR fusionnée → porte les changements vers SDK parallèle

## Créer une routine

### Sur le web
1. Ouvrir [claude.ai/code/routines](https://claude.ai/code/routines) → New routine
2. Nom + invite (autonome, explicite sur ce qu'il faut faire et ce qui compte comme succès)
3. Sélectionner référentiels GitHub (cloné à chaque exécution, branche par défaut)
4. Sélectionner environnement cloud (accès réseau, variables d'env, script de setup)
5. Sélectionner déclencheur(s)
6. Configurer connecteurs et permissions (branches push)

### Depuis la CLI
```text
/schedule daily PR review at 9am
/schedule clean up feature flag in one week
/schedule list     # voir toutes les routines
/schedule update   # modifier une routine
/schedule run      # déclencher immédiatement
```

## Configuration des déclencheurs

### Planifié
- Fréquences prédéfinies : chaque heure, quotidienne, jours de semaine, hebdomadaire
- Intervalle minimum : 1 heure
- Expressions cron personnalisées via `/schedule update`
- Exécution unique : s'auto-désactive après déclenchement (ne compte pas vers le plafond quotidien)

### API
```bash
curl -X POST https://api.anthropic.com/v1/claude_code/routines/trig_01ABCD.../fire \
  -H "Authorization: Bearer sk-ant-oat01-xxxxx" \
  -H "anthropic-beta: experimental-cc-routine-2026-04-01" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"text": "Sentry alert SEN-4521 fired in prod."}'
```

Réponse :
```json
{"type": "routine_fire", "claude_code_session_id": "session_01...", "claude_code_session_url": "https://claude.ai/code/session_01..."}
```

### GitHub
Événements supportés : Pull request (opened, closed, assignée, étiquetée, synchronisée) et Release.

Filtres PR disponibles : Author, Title, Body, Base branch, Head branch, Labels, Is draft, Is merged.

Opérateurs : égal à, contient, commence par, est l'un de, n'est pas l'un de, correspond à regex.

## Environnement et accès réseau

Environnement Default = accès réseau Trusted (registres paquets courants, APIs cloud).
Pour accès supplémentaires : modifier "Network access" dans les paramètres de l'environnement.

## Connecteurs

Tous les connecteurs claude.ai du compte sont inclus par défaut. Les serveurs MCP ajoutés localement via CLI (pas sur claude.ai) ne sont pas disponibles.

Pour les utiliser dans une routine :
- Les ajouter sur [claude.ai/customize/connectors](https://claude.ai/customize/connectors)
- Ou les déclarer dans `.mcp.json` dans le repo

## Permissions de branche

Par défaut : Claude peut pousser uniquement sur des branches `claude/*`.
Activer "Allow unrestricted branch pushes" pour autoriser les autres branches.

## Utilisation et limites

- Réduit l'utilisation d'abonnement comme les sessions interactives
- Plafond quotidien sur le nombre d'exécutions par compte
- Exécutions uniques ne comptent pas vers le plafond quotidien
- Avec crédits d'utilisation : peut continuer au-delà du plafond (facturé)

## Dépannage

**`/schedule` retourne "Commande inconnue"** :
- Authentifié avec clé API ou fournisseur cloud (pas claude.ai) → nécessite connexion claude.ai
- Variables `DISABLE_TELEMETRY`, `DO_NOT_TRACK`, etc. définies → désactivent la récupération des flags
- CLI < v2.1.81 → `claude update`
- Dans une session Claude Code sur le web → gérer depuis l'interface web
