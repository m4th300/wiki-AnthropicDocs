# Tâches planifiées (/loop)

Source: https://code.claude.com/docs/fr/scheduled-tasks

Note : Nécessite Claude Code v2.1.72+. Tâches limitées à la session.

## Options de planification

| Option | S'exécute sur | Nécessite machine allumée | Nécessite session ouverte |
|--------|-------------|--------------------------|--------------------------|
| Cloud (Routines) | Cloud Anthropic | Non | Non |
| Desktop | Votre machine | Oui | Non |
| /loop | Votre machine | Oui | Oui |

## /loop — Exécuter un prompt de manière répétée

### Avec intervalle et prompt

```text
/loop 5m check if the deployment finished and tell me what happened
```

Exécute le prompt toutes les 5 minutes. Unités : `s`, `m`, `h`, `d`.

### Avec prompt uniquement (intervalle choisi par Claude)

```text
/loop check whether CI passed and address any review comments
```

Claude choisit l'intervalle dynamiquement selon ce qu'il observe.

### Sans prompt (maintenance intégrée)

```text
/loop
```

Exécute le prompt de maintenance intégré :
1. Continuer le travail inachevé de la conversation
2. S'occuper de la MR de la branche courante (commentaires, CI, conflits)
3. Nettoyage (chasse aux bogues, simplification) quand rien d'autre

### Personnaliser avec loop.md

Deux emplacements (premier trouvé utilisé) :
- `.claude/loop.md` (niveau projet, priorité)
- `~/.claude/loop.md` (niveau utilisateur)

Format : texte libre Markdown. Maximum 25 000 octets.

```markdown
Check the `release/next` PR. If CI is red, pull the failing job log,
diagnose, and push a minimal fix. If new review comments have arrived,
address each one and resolve the thread.
```

### Arrêter une boucle

`Esc` pendant l'attente de la prochaine itération.

## Rappels ponctuels

```text
remind me at 3pm to push the release branch
in 45 minutes, check whether the integration tests passed
```

La tâche se supprime après son exécution.

## Gérer les tâches planifiées

```text
what scheduled tasks do I have?
cancel the deploy check job
```

Outils sous-jacents : `CronCreate`, `CronList`, `CronDelete`
Max 50 tâches par session.

## Comportement

- Vérification chaque seconde, mise en file d'attente à faible priorité
- Prompt planifié s'exécute entre vos tours (pas pendant une réponse)
- Fuseau horaire local
- Gigue : offset déterministe par tâche (jusqu'à 30 min pour tâches récurrentes)
- **Expiration 7 jours** : tâches récurrentes s'auto-suppriment après 7 jours

## Référence cron

Format : `minute heure jour-du-mois mois jour-de-la-semaine`

```
*/5 * * * *   Toutes les 5 minutes
0 9 * * *     Chaque jour à 9h
0 9 * * 1-5   Jours de semaine à 9h
30 14 15 3 *  15 mars à 14h30
```

## Limites

- Tâches ne s'exécutent que pendant que Claude Code est inactif
- Pas de rattrapage pour les exécutions manquées
- `/clear` efface toutes les tâches de session
- `--resume` restaure les tâches non expirées

## Désactiver

`CLAUDE_CODE_DISABLE_CRON=1`

## Bedrock/Vertex/Foundry

- Sans intervalle : exécution fixe toutes les 10 minutes
- `loop.md` non lu sur ces plateformes
