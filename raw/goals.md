# Garder Claude orienté vers un objectif

> Définissez une condition d'achèvement avec /goal et Claude continue de travailler sur plusieurs tours jusqu'à ce que la condition soit satisfaite.

> `/goal` nécessite Claude Code v2.1.139 ou une version ultérieure.

La commande `/goal` définit une condition d'achèvement et Claude continue de travailler vers celle-ci sans que vous ayez besoin de le relancer à chaque étape. Après chaque tour, un petit modèle rapide vérifie si la condition est satisfaite. Si ce n'est pas le cas, Claude commence un autre tour au lieu de vous rendre le contrôle. L'objectif s'efface automatiquement une fois la condition satisfaite.

Utilisez un objectif pour un travail substantiel avec un état final vérifiable :

* Migrer un module vers une nouvelle API jusqu'à ce que chaque site d'appel se compile et que les tests réussissent
* Implémenter un document de conception jusqu'à ce que tous les critères d'acceptation soient satisfaits
* Diviser un grand fichier en modules ciblés jusqu'à ce que chacun soit en dessous d'un budget de taille
* Traiter une file d'attente de problèmes étiquetés jusqu'à ce que la queue soit vide

Cette page couvre comment :

* Comparer les approches de flux de travail autonome : `/loop`, Stop hooks, et mode auto
* Définir un objectif et écrire une condition efficace
* Vérifier le statut, effacer rapidement, et exécuter de manière non-interactive
* Voir comment fonctionne l'évaluation et les exigences

## Comparer les façons de maintenir une session en cours

Trois approches maintiennent la session actuelle en cours entre les invites. Choisissez en fonction de ce qui devrait démarrer le tour suivant :

| Approche | Le tour suivant commence quand | S'arrête quand |
| :------- | :----------------------------- | :------------- |
| `/goal` | Le tour précédent se termine | Un modèle confirme que la condition est satisfaite |
| `/loop` | Un intervalle de temps s'écoule | Vous l'arrêtez, ou Claude décide que le travail est terminé |
| Stop hook | Le tour précédent se termine | Votre propre script ou invite décide |

`/goal` et un Stop hook se déclenchent tous les deux après chaque tour. `/goal` est un raccourci limité à la session : vous tapez une condition et elle est active pour la session actuelle uniquement. Un Stop hook réside dans votre fichier de paramètres, s'applique à chaque session dans sa portée, et peut exécuter un script pour des vérifications déterministes ou une invite pour des vérifications évaluées par le modèle.

Le mode auto en lui-même approuve les appels d'outils au sein d'un seul tour mais ne démarre pas un nouveau. Claude s'arrête quand il juge le travail terminé. `/goal` ajoute un évaluateur séparé qui vérifie votre condition après chaque tour, donc l'achèvement est décidé par un modèle frais plutôt que par celui qui effectue le travail. Les deux sont complémentaires : le mode auto supprime les invites par outil, et `/goal` supprime les invites par tour.

## Utiliser `/goal`

Un seul objectif peut être actif par session. La même commande le définit, le vérifie et l'efface selon l'argument.

### Définir un objectif

Exécutez `/goal` suivi de la condition que vous souhaitez satisfaire. Si un objectif est déjà actif, le nouveau le remplace.

```text
/goal all tests in test/auth pass and the lint step is clean
```

Définir un objectif démarre immédiatement un tour, avec la condition elle-même comme directive. Vous n'avez pas besoin d'envoyer une invite séparée. Pendant que l'objectif est actif, un indicateur `◎ /goal active` montre depuis combien de temps l'objectif s'exécute.

Après chaque tour, l'évaluateur retourne une courte raison expliquant pourquoi la condition est ou n'est pas satisfaite. La raison la plus récente apparaît dans la vue de statut et dans la transcription afin que vous puissiez voir vers quoi Claude travaille ensuite.

Un objectif continue de s'exécuter jusqu'à ce que la condition soit satisfaite ou que vous exécutiez `/goal clear`. Exécutez `/goal` sans argument pour voir les tours et les jetons dépensés jusqu'à présent.

### Écrire une condition efficace

L'évaluateur juge votre condition par rapport à ce que Claude a présenté dans la conversation. Il n'exécute pas les commandes ou ne lit pas les fichiers indépendamment, donc écrivez la condition comme quelque chose que la propre sortie de Claude peut démontrer. « Tous les tests dans `test/auth` réussissent » fonctionne parce que Claude exécute les tests et le résultat se retrouve dans la transcription pour que l'évaluateur le lise.

Une condition qui tient sur plusieurs tours a généralement :

* **Un état final mesurable** : un résultat de test, un code de sortie de build, un nombre de fichiers, une queue vide
* **Une vérification énoncée** : comment Claude devrait le prouver, comme « `npm test` sort 0 » ou « `git status` est propre »
* **Des contraintes qui importent** : tout ce qui ne doit pas changer en chemin, comme « aucun autre fichier de test n'est modifié »

La condition peut contenir jusqu'à 4 000 caractères.

Pour limiter la durée d'exécution d'un objectif, incluez une clause de tour ou de temps dans la condition, comme `or stop after 20 turns`. Claude rapporte la progression par rapport à cette clause à chaque tour et l'évaluateur la juge à partir de la conversation.

### Vérifier le statut

Exécutez `/goal` sans arguments pour voir l'état actuel.

```text
/goal
```

Si un objectif est actif, le statut affiche :

* La condition
* Depuis combien de temps il s'exécute
* Combien de tours ont été évalués
* La dépense de jetons actuelle
* La raison la plus récente de l'évaluateur

Si aucun objectif n'est actif mais qu'un a été atteint plus tôt dans la session, le statut affiche la condition atteinte ainsi que sa durée, son nombre de tours et sa dépense de jetons.

### Effacer un objectif

Exécutez `/goal clear` pour supprimer un objectif actif avant que sa condition soit satisfaite.

```text
/goal clear
```

`stop`, `off`, `reset`, `none`, et `cancel` sont acceptés comme alias pour `clear`. L'exécution de `/clear` pour démarrer une nouvelle conversation supprime également tout objectif actif.

### Reprendre avec un objectif actif

Un objectif qui était encore actif quand une session s'est terminée est restauré quand vous reprenez cette session avec `--resume` ou `--continue`. La condition est conservée, mais le nombre de tours, le minuteur et la ligne de base de dépense de jetons se réinitialisent à la reprise. Un objectif qui était déjà atteint ou effacé n'est pas restauré.

### Exécuter de manière non-interactive

`/goal` fonctionne en mode non-interactive, dans l'application de bureau, et via Remote Control. Définir un objectif avec `-p` exécute la boucle jusqu'à l'achèvement en une seule invocation :

```bash
claude -p "/goal CHANGELOG.md has an entry for every PR merged this week"
```

Interrompez le processus avec Ctrl+C pour arrêter un objectif non-interactive avant que la condition soit satisfaite.

## Comment fonctionne l'évaluation

`/goal` est un wrapper autour d'un Stop hook basé sur une invite limité à la session. Chaque fois que Claude termine un tour, la condition et la conversation jusqu'à présent sont envoyées à votre petit modèle rapide configuré, qui par défaut est Haiku. Le modèle retourne une décision oui ou non et une courte raison. Un « non » indique à Claude de continuer à travailler et inclut la raison comme guidance pour le tour suivant. Un « oui » efface l'objectif et enregistre une entrée atteinte dans la transcription.

L'évaluateur s'exécute sur le fournisseur pour lequel votre session est configurée. Il n'appelle pas les outils, donc il ne peut juger que ce que Claude a déjà présenté dans la conversation.

Les jetons d'évaluation sont facturés sur le petit modèle rapide configuré pour votre fournisseur et sont généralement négligeables par rapport à la dépense du tour principal.

## Exigences

`/goal` s'exécute uniquement dans les espaces de travail où vous avez accepté la boîte de dialogue de confiance, car l'évaluateur fait partie du système de hooks. `/goal` est également indisponible quand `disableAllHooks` est défini à n'importe quel niveau de paramètres ou quand `allowManagedHooksOnly` est défini dans les paramètres gérés. Dans chaque cas, la commande vous indique pourquoi au lieu de ne rien faire silencieusement.

## Voir aussi

* Exécuter une invite à plusieurs reprises avec `/loop` : réexécuter sur un intervalle de temps au lieu de jusqu'à ce qu'une condition soit satisfaite
* Hooks basés sur une invite : écrivez votre propre Stop hook quand vous avez besoin d'une logique d'évaluation personnalisée
* Mode auto : approuvez les appels d'outils automatiquement afin que chaque tour d'objectif s'exécute sans surveillance
* Comparaison de planification : exécutez un travail selon un calendrier indépendant de toute session ouverte
