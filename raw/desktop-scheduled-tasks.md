# Tâches planifiées Desktop

Source: https://code.claude.com/docs/fr/desktop-scheduled-tasks

## Ce que c'est

Tâches planifiées locales dans l'application Claude Code Desktop.
S'exécute sur votre machine. Nécessite l'application ouverte et l'ordinateur actif.

Créer via : onglet **Routines** → **New routine** → **Local** (vs Remote = Routines cloud).

## Comparaison

| | Cloud (Routines) | Desktop | /loop |
|--|------|---------|-------|
| Nécessite machine allumée | Non | Oui | Oui |
| Nécessite session ouverte | Non | Non | Oui |
| Accès fichiers locaux | Non (clone frais) | Oui | Oui |
| Intervalle minimum | 1 heure | 1 minute | 1 minute |

## Créer une tâche

Champs :
- **Name** : identifiant (kebab-case, unique)
- **Description** : résumé court
- **Instructions** : prompt Claude + sélecteur mode permission + sélecteur modèle + dossier de travail
- **Schedule** : fréquence

Schedules disponibles : Manual, Hourly, Daily (heure), Weekdays, Weekly (heure + jour)
Pour des intervalles personnalisés → demander à Claude en session.

Option **Worktree** : chaque exécution dans son propre git worktree isolé.

## Comportement

- Desktop vérifie la planification chaque minute
- Démarre une nouvelle session quand une tâche est due
- Notifications de bureau à chaque activation
- Sessions apparaissent sous section **Scheduled** dans la barre latérale
- Petit délai déterministe après l'heure planifiée (échelonnement)

## Exécutions manquées

Au réveil : Desktop vérifie les exécutions manquées des 7 derniers jours → une seule exécution de rattrapage pour la plus récente. Les exécutions plus anciennes sont rejetées.

Rédiger des prompts robustes : « Examinez uniquement les commits d'aujourd'hui. S'il est après 17h, ignorer la révision. »

## Permissions

Mode de permission défini par tâche. Les règles de `~/.claude/settings.json` s'appliquent aussi.
Pour éviter les arrêts : **Run now** → approuver chaque outil → "always allow".
Examiner les approbations sur la page de détail de la tâche.

## Gérer les tâches

Depuis la page de détail : **Run now**, **Status** (Active/Paused), **Edit**, **Review history**, **Review allowed permissions**, **Delete**.

Aussi possible en session Desktop : "pause my dependency-audit task", "show me my scheduled tasks".

Fichier de prompt : `~/.claude/scheduled-tasks/<task-name>/SKILL.md`

## Pour les tâches sans machine allumée

Créer une [Routines](/fr/routines) distante à la place.
