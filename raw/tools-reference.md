# Référence des outils

> Référence complète des outils que Claude Code peut utiliser, y compris les exigences de permission et le comportement par outil.

Claude Code a accès à un ensemble d'outils intégrés qui l'aident à comprendre et modifier votre base de code. Les noms d'outils sont les chaînes exactes que vous utilisez dans les règles de permission spécifiques aux outils, les listes d'outils subagent, et les correspondances de hook. Pour désactiver complètement un outil, ajoutez son nom au tableau `deny` dans vos paramètres de permission.

Pour ajouter des outils personnalisés, connectez un serveur MCP. Pour étendre Claude avec des flux de travail réutilisables basés sur des invites, écrivez une skill, qui s'exécute via l'outil `Skill` existant plutôt que d'ajouter une nouvelle entrée d'outil.

## Tableau des outils

| Outil                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Permission requise |
| :--------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------- |
| `Agent`                | Crée un subagent avec sa propre fenêtre de contexte pour gérer une tâche. Voir comportement de l'outil Agent                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | Non                |
| `AskUserQuestion`      | Pose des questions à choix multiples pour recueillir les exigences ou clarifier l'ambiguïté                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Non                |
| `Bash`                 | Exécute des commandes shell dans votre environnement. Voir comportement de l'outil Bash                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Oui                |
| `CronCreate`           | Planifie une invite récurrente ou ponctuelle dans la session actuelle. Les tâches sont limitées à la session et restaurées sur `--resume` ou `--continue` si non expirées.                                                                                                                                                                                                                                                                                                                                                                                                                                              | Non                |
| `CronDelete`           | Annule une tâche planifiée par ID                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Non                |
| `CronList`             | Liste toutes les tâches planifiées dans la session                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | Non                |
| `Edit`                 | Effectue des modifications ciblées sur des fichiers spécifiques. Voir comportement de l'outil Edit                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | Oui                |
| `EnterPlanMode`        | Bascule en mode plan pour concevoir une approche avant de coder                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Non                |
| `EnterWorktree`        | Crée un git worktree isolé et y bascule. Passez un `path` pour basculer dans un worktree existant du référentiel actuel au lieu d'en créer un nouveau. De dans une session worktree, ou à partir d'un subagent avec un répertoire de travail épinglé tel que `isolation: worktree`, seule la forme `path` est disponible et la cible doit être sous `.claude/worktrees/`                                                                                                                                                                                                                                                | Non                |
| `ExitPlanMode`         | Présente un plan pour approbation et quitte le mode plan                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Oui                |
| `ExitWorktree`         | Quitte une session worktree et revient au répertoire d'origine. Non disponible pour les subagents qui s'exécutent déjà dans leur propre répertoire de travail, comme avec `isolation: worktree`                                                                                                                                                                                                                                                                                                                                                                                                                         | Non                |
| `Glob`                 | Trouve des fichiers en fonction de la correspondance de motifs. Voir comportement de l'outil Glob                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Non                |
| `Grep`                 | Recherche des motifs dans le contenu des fichiers. Voir comportement de l'outil Grep                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Non                |
| `ListMcpResourcesTool` | Liste les ressources exposées par les serveurs MCP connectés                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | Non                |
| `LSP`                  | Intelligence du code via les serveurs de langage : accéder aux définitions, trouver les références, signaler les erreurs de type et les avertissements. Voir comportement de l'outil LSP                                                                                                                                                                                                                                                                                                                                                                                                                                | Non                |
| `Monitor`              | Exécute une commande en arrière-plan et renvoie chaque ligne de sortie à Claude, afin qu'il puisse réagir aux entrées de journal, aux modifications de fichiers, ou au statut interrogé en milieu de conversation. Nécessite Claude Code v2.1.98 ou version ultérieure.                                                                                                                                                                                                                                                                                                                                                 | Oui                |
| `NotebookEdit`         | Modifie les cellules de notebook Jupyter. Voir comportement de l'outil NotebookEdit                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Oui                |
| `PowerShell`           | Exécute des commandes PowerShell nativement. Voir outil PowerShell pour la disponibilité                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Oui                |
| `PushNotification`     | Envoie une notification de bureau, et une notification push sur téléphone quand Remote Control est connecté. La livraison push s'effectue via l'infrastructure hébergée par Anthropic, qui n'est pas accessible depuis Amazon Bedrock, Google Vertex AI, ou Microsoft Foundry                                                                                                                                                                                                                                                                                                                                           | Non                |
| `Read`                 | Lit le contenu des fichiers. Voir comportement de l'outil Read                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Non                |
| `ReadMcpResourceTool`  | Lit une ressource MCP spécifique par URI                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Non                |
| `RemoteTrigger`        | Crée, met à jour, exécute et liste les Routines sur claude.ai. Soutient la commande `/schedule`. Les Routines vivent sur claude.ai et nécessitent un plan Pro, Max, Team ou Enterprise, donc cet outil n'est pas accessible depuis Amazon Bedrock, Google Vertex AI, ou Microsoft Foundry                                                                                                                                                                                                                                                                                                                               | Non                |
| `ScheduleWakeup`       | Reprogramme la prochaine itération d'une `/loop` auto-rythmée. Claude l'appelle à la fin de chaque itération pour choisir quand la prochaine s'exécute, entre une minute et une heure. Non disponible sur Amazon Bedrock, Google Vertex AI, ou Microsoft Foundry.                                                                                                                                                                                                                                                                                                                                                       | Non                |
| `SendMessage`          | Envoie un message à un coéquipier de l'équipe d'agents, ou reprend un subagent par son ID d'agent. Les subagents arrêtés se reprennent automatiquement en arrière-plan. Disponible uniquement quand `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` est défini                                                                                                                                                                                                                                                                                                                                                                 | Non                |
| `ShareOnboardingGuide` | Télécharge `ONBOARDING.md` et retourne un lien de partage que les coéquipiers peuvent ouvrir dans Claude Code. Appelé depuis `/team-onboarding` après que le guide soit écrit. Disponible pour les abonnés claude.ai sur les plans Pro, Max, Team et Enterprise                                                                                                                                                                                                                                                                                                                                                          | Oui                |
| `Skill`                | Exécute une skill dans la conversation principale                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Oui                |
| `TaskCreate`           | Crée une nouvelle tâche dans la liste des tâches                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Non                |
| `TaskGet`              | Récupère les détails complets d'une tâche spécifique                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Non                |
| `TaskList`             | Liste toutes les tâches avec leur statut actuel                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Non                |
| `TaskOutput`           | (Obsolète) Récupère la sortie d'une tâche en arrière-plan. Préférez `Read` sur le chemin du fichier de sortie de la tâche                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Non                |
| `TaskStop`             | Arrête une tâche en arrière-plan en cours d'exécution par ID                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | Non                |
| `TaskUpdate`           | Met à jour le statut de la tâche, les dépendances, les détails, ou supprime les tâches                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | Non                |
| `TeamCreate`           | Crée une équipe d'agents avec plusieurs coéquipiers. Disponible uniquement quand `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` est défini                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Non                |
| `TeamDelete`           | Dissout une équipe d'agents et nettoie les processus des coéquipiers. Disponible uniquement quand `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` est défini                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Non                |
| `TodoWrite`            | (min v2.1.142) Gère la liste de contrôle des tâches de la session. Désactivé par défaut à partir de v2.1.142 en faveur de `TaskCreate`, `TaskGet`, `TaskList`, et `TaskUpdate`. Définissez `CLAUDE_CODE_ENABLE_TASKS=0` pour réactiver                                                                                                                                                                                                                                                                                                                                                                                  | Non                |
| `ToolSearch`           | Recherche et charge les outils différés quand la recherche d'outils est activée                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Non                |
| `WaitForMcpServers`    | (min v2.1.142) Attend un ou plusieurs serveurs MCP qui se connectent toujours en arrière-plan. Claude l'appelle quand un serveur nécessaire n'est pas encore connecté. N'apparaît que quand la recherche d'outils est désactivée.                                                                                                                                                                                                                                                                                                                                                                                        | Non                |
| `WebFetch`             | Récupère le contenu d'une URL spécifiée. Voir comportement de l'outil WebFetch                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Oui                |
| `WebSearch`            | Effectue des recherches web. Voir comportement de l'outil WebSearch                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Oui                |
| `Workflow`             | Exécute un flux de travail dynamique : un script qui orchestre de nombreux subagents en arrière-plan et retourne un résultat consolidé                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | Oui                |
| `Write`                | Crée ou remplace des fichiers. Voir comportement de l'outil Write                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Oui                |

## Configurer les outils avec les règles de permission et les hooks

Pour la plupart, Claude décide quand utiliser ces outils et vous n'avez pas besoin de les nommer vous-même lors de l'interaction avec Claude. Vous référencez les noms d'outils directement lors de la définition des permissions et d'autres configurations :

* dans `permissions.allow` et `permissions.deny` dans les paramètres, et l'interface `/permissions`
* dans les drapeaux CLI `--allowedTools` et `--disallowedTools`
* dans les options `allowedTools` et `disallowedTools` du SDK Agent
* dans le frontmatter `tools` ou `disallowedTools` d'un subagent
* dans le frontmatter `allowed-tools` d'une skill
* dans la condition `if` d'un hook

Tous ces éléments acceptent le même format de règle, `ToolName(specifier)`. Le spécificateur dépend de l'outil, et plusieurs outils partagent un format :

| Format de règle                | S'applique à              | Détails                                                                     |
| :----------------------------- | :------------------------ | :-------------------------------------------------------------------------- |
| `Bash(npm run *)`              | Bash, Monitor             | Correspondance de motif de commande                                         |
| `PowerShell(Get-ChildItem *)`  | PowerShell                | Correspondance de motif de commande                                         |
| `Read(~/secrets/**)`           | Read, Grep, Glob, LSP     | Correspondance de motif de chemin                                           |
| `Edit(/src/**)`                | Edit, Write, NotebookEdit | Correspondance de motif de chemin                                           |
| `Skill(deploy *)`              | Skill                     | Correspondance de nom de skill                                              |
| `Agent(Explore)`               | Agent                     | Correspondance de type de subagent                                          |
| `WebFetch(domain:example.com)` | WebFetch                  | Correspondance de domaine                                                   |
| `WebSearch`                    | WebSearch                 | Pas de spécificateur ; autoriser ou refuser l'outil dans son ensemble       |

Les outils non listés ici, tels que `ExitPlanMode` ou `ShareOnboardingGuide`, acceptent uniquement le nom d'outil nu sans spécificateur.

Une règle d'autorisation `Edit(...)` accorde également l'accès en lecture au même chemin, donc vous n'avez pas besoin d'une règle `Read(...)` correspondante.

Les champs `matcher` des hooks utilisent les noms d'outils nus, pas le format entre parenthèses.

## Comportement de l'outil Agent

L'outil Agent crée un subagent dans une fenêtre de contexte séparée. Le subagent travaille sur sa tâche de manière autonome, puis retourne un seul résultat textuel à la conversation parent. Le parent ne voit pas les appels d'outils intermédiaires ou les sorties du subagent, seulement ce résultat final.

Le même outil Agent lance également les subagents forked quand le mode fork est activé. Un fork hérite de la conversation parent complète au lieu de commencer à zéro, s'exécute toujours en arrière-plan, et affiche toujours les invites de permission dans votre terminal.

Les outils qu'un subagent nommé peut utiliser dépendent des champs `tools` et `disallowedTools` dans la définition du subagent :

* **Aucun champ défini** : le subagent hérite de chaque outil disponible pour le parent.
* **`tools` uniquement** : le subagent obtient uniquement les outils listés.
* **`disallowedTools` uniquement** : le subagent obtient chaque outil parent sauf ceux listés.
* **Les deux définis** : `disallowedTools` a la priorité. Un outil listé dans les deux est supprimé.

Lancer le subagent ne demande pas lui-même la permission. Les appels d'outils du subagent sont vérifiés par rapport à vos règles de permission au fur et à mesure qu'il s'exécute :

* **Les subagents au premier plan** affichent les mêmes invites de permission que vous verriez dans la conversation principale.
* **Les subagents en arrière-plan** n'affichent pas d'invites. Ils s'exécutent avec les permissions déjà accordées dans la session et refusent automatiquement tout appel d'outil qui demanderait autrement une permission. Après un refus, le subagent continue sans cet outil.

## Comportement de l'outil Bash

L'outil Bash exécute chaque commande dans un processus séparé avec le comportement de persistance suivant :

* Quand Claude exécute `cd` dans la session principale, le nouveau répertoire de travail persiste dans les commandes Bash ultérieures tant qu'il reste à l'intérieur du répertoire du projet ou d'un répertoire de travail supplémentaire ajouté avec `--add-dir`, `/add-dir`, ou `additionalDirectories`. Les sessions subagent ne reportent jamais les modifications de répertoire de travail.
  * Si `cd` aboutit en dehors de ces répertoires, Claude Code réinitialise au répertoire du projet et ajoute `Shell cwd was reset to <dir>` au résultat de l'outil.
  * Pour désactiver ce report afin que chaque commande Bash démarre dans le répertoire du projet, définissez `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1`.
* Les variables d'environnement ne persistent pas. Un `export` dans une commande ne sera pas disponible dans la suivante.
* Les alias et les fonctions shell définis dans votre fichier de démarrage shell sont disponibles. Au démarrage de la session, Claude Code source `~/.zshrc`, `~/.bashrc`, ou `~/.profile` selon votre shell, capture les alias, fonctions et options shell résultants, et les applique à chaque commande Bash.

Activez votre environnement virtualenv ou conda avant de lancer Claude Code. Pour que les variables d'environnement persistent entre les commandes Bash, définissez `CLAUDE_ENV_FILE` sur un script shell avant de lancer Claude Code, ou utilisez un hook SessionStart pour le remplir dynamiquement.

Deux limites limitent chaque commande :

* **Délai d'expiration** : deux minutes par défaut. Claude peut demander jusqu'à 10 minutes par commande avec le paramètre `timeout`. Remplacez la valeur par défaut et le plafond avec `BASH_DEFAULT_TIMEOUT_MS` et `BASH_MAX_TIMEOUT_MS`.
* **Longueur de sortie** : 30 000 caractères par défaut. Quand une commande produit plus que cela, Claude Code enregistre la sortie complète dans un fichier du répertoire de session et donne à Claude le chemin du fichier plus un court aperçu du début. Augmentez la limite avec `BASH_MAX_OUTPUT_LENGTH`, jusqu'à un plafond dur de 150 000 caractères.

Pour les processus de longue durée tels que les serveurs de développement ou les builds de surveillance, Claude peut définir `run_in_background: true` pour démarrer la commande en tant que tâche en arrière-plan et continuer à travailler pendant qu'elle s'exécute. Listez et arrêtez les tâches en arrière-plan avec `/tasks`.

## Comportement de l'outil Edit

L'outil Edit effectue un remplacement de chaîne exact. Il prend un `old_string` et un `new_string` et remplace le premier par le second. Il n'utilise pas regex ou la correspondance floue.

Trois vérifications doivent réussir pour qu'une modification s'applique :

* **Lecture avant modification** : Claude doit avoir lu le fichier dans la conversation actuelle, et le fichier ne doit pas avoir changé sur le disque depuis cette lecture. Cette vérification s'exécute en premier, avant toute correspondance de chaîne.
* **Correspondance** : `old_string` doit apparaître dans le fichier exactement comme écrit. Une seule différence de caractère d'espace ou d'indentation est suffisante pour manquer.
* **Unicité** : `old_string` doit apparaître exactement une fois. Quand il apparaît plus d'une fois, Claude fournit soit une chaîne plus longue avec suffisamment de contexte environnant pour épingler une occurrence, soit définit `replace_all: true` pour les remplacer tous.

Afficher un fichier avec Bash satisfait également l'exigence de lecture avant modification quand la commande est `cat`, `head`, `tail`, `sed -n 'X,Yp'`, `grep`, `egrep`, ou `fgrep` sur un seul fichier sans pipes ou redirections.

## Comportement de l'outil Glob

L'outil Glob trouve des fichiers par motif de nom. Il supporte la syntaxe glob standard incluant `**` pour la correspondance de répertoire récursive :

* `**/*.js` correspond à tous les fichiers `.js` à n'importe quelle profondeur
* `src/**/*.ts` correspond à tous les fichiers `.ts` sous `src/`
* `*.{json,yaml}` correspond aux fichiers `.json` et `.yaml` dans le répertoire actuel

Les résultats sont triés par heure de modification et limités à 100 fichiers. Si la limite est atteinte, Claude voit un drapeau de troncature dans le résultat et peut affiner le motif.

Glob ne respecte pas `.gitignore` par défaut, donc il trouve les fichiers gitignorés aux côtés des fichiers suivis. Cela diffère de Grep, qui ignore les fichiers gitignorés. Pour faire respecter `.gitignore` à Glob, définissez `CLAUDE_CODE_GLOB_NO_IGNORE=false` avant de lancer Claude Code.

## Comportement de l'outil Grep

L'outil Grep recherche des motifs dans le contenu des fichiers. Où Glob trouve des fichiers par nom, Grep trouve des lignes à l'intérieur d'eux.

Grep est construit sur ripgrep et utilise la syntaxe regex de ripgrep, pas grep POSIX. Les motifs qui incluent des métacaractères regex ont besoin d'échappement. Par exemple, trouver `interface{}` dans le code Go prend le motif `interface\{\}`.

Trois modes de sortie contrôlent ce qui revient :

* `files_with_matches` : chemins de fichiers uniquement, pas de contenu de ligne. C'est la valeur par défaut.
* `content` : lignes correspondantes avec numéro de fichier et de ligne.
* `count` : nombre de correspondances par fichier.

Claude peut limiter les résultats par fichier avec le paramètre `glob`, tel que `**/*.tsx`, ou par langage avec le paramètre `type`, tel que `py` ou `rust`. Par défaut, les motifs correspondent dans une seule ligne. Claude peut définir `multiline: true` pour correspondre à travers les limites de ligne.

Grep respecte `.gitignore`, donc les fichiers gitignorés sont ignorés. Pour rechercher un fichier gitignore, Claude transmet son chemin directement.

## Comportement de l'outil LSP

L'outil LSP donne à Claude l'intelligence du code à partir d'un serveur de langage en cours d'exécution. Après chaque modification de fichier, il signale automatiquement les erreurs de type et les avertissements afin que Claude puisse corriger les problèmes sans une étape de compilation séparée. Claude peut également l'appeler directement pour naviguer dans le code :

* Accéder à la définition d'un symbole
* Trouver toutes les références à un symbole
* Obtenir les informations de type à une position
* Lister les symboles dans un fichier ou un espace de travail
* Trouver les implémentations d'une interface
* Tracer les hiérarchies d'appels

L'outil est inactif jusqu'à ce que vous installiez un plugin d'intelligence du code pour votre langage. Le plugin regroupe la configuration du serveur de langage, et vous installez le binaire du serveur séparément.

## Outil Monitor

L'outil Monitor nécessite Claude Code v2.1.98 ou version ultérieure.

L'outil Monitor permet à Claude de surveiller quelque chose en arrière-plan et de réagir quand cela change, sans mettre en pause la conversation. Demandez à Claude de :

* Suivre un fichier journal et signaler les erreurs au fur et à mesure qu'elles apparaissent
* Interroger une PR ou un travail CI et signaler quand son statut change
* Surveiller un répertoire pour les modifications de fichiers
* Suivre la sortie de tout script de longue durée que vous lui pointez

Claude écrit un petit script pour la surveillance, l'exécute en arrière-plan, et reçoit chaque ligne de sortie à son arrivée. Vous continuez à travailler dans la même session et Claude intervient quand un événement arrive. Arrêtez une surveillance en demandant à Claude de l'annuler ou en terminant la session.

Monitor utilise les mêmes règles de permission que Bash, donc les motifs `allow` et `deny` que vous avez définis pour Bash s'appliquent ici aussi. Il n'est pas disponible sur Amazon Bedrock, Google Vertex AI, ou Microsoft Foundry. Il n'est également pas disponible quand `DISABLE_TELEMETRY` ou `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` est défini.

Les plugins peuvent déclarer des moniteurs qui démarrent automatiquement quand le plugin est actif.

## Comportement de l'outil NotebookEdit

NotebookEdit modifie un notebook Jupyter une cellule à la fois, ciblant les cellules par leur `cell_id`. Il n'effectue pas de remplacement de chaîne à travers le notebook comme le fait Edit sur les fichiers simples.

Trois modes de modification contrôlent ce qui arrive à la cellule cible :

* `replace` : remplace la source de la cellule. C'est la valeur par défaut.
* `insert` : ajoute une nouvelle cellule après la cible. Sans `cell_id`, la nouvelle cellule va au début du notebook. Nécessite que `cell_type` soit défini à `code` ou `markdown`.
* `delete` : supprime la cellule cible.

Les règles de permission utilisent le format de chemin `Edit(...)`. Une règle comme `Edit(notebooks/**)` couvre les appels NotebookEdit sur les fichiers dans ce répertoire.

## Outil PowerShell

L'outil PowerShell permet à Claude d'exécuter des commandes PowerShell nativement. Sur Windows, cela signifie que les commandes s'exécutent dans PowerShell au lieu d'être acheminées via Git Bash. Sur Windows sans Git Bash, l'outil est activé automatiquement. Sur Windows avec Git Bash installé, l'outil est en déploiement progressif. Sur Linux, macOS et WSL, l'outil est opt-in.

### Activer l'outil PowerShell

Définissez `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` dans votre environnement ou dans `settings.json` :

```json
{
  "env": {
    "CLAUDE_CODE_USE_POWERSHELL_TOOL": "1"
  }
}
```

Sur Windows, définissez la variable à `0` pour refuser le déploiement. Sur Linux, macOS et WSL, l'outil nécessite PowerShell 7 ou version ultérieure : installez `pwsh` et assurez-vous qu'il est sur votre `PATH`.

Sur Windows, Claude Code détecte automatiquement `pwsh.exe` pour PowerShell 7+ avec un repli sur `powershell.exe` pour PowerShell 5.1. Lorsque l'outil est activé, Claude traite PowerShell comme le shell principal. L'outil Bash reste disponible pour les scripts POSIX lorsque Git Bash est installé.

Claude Code lance PowerShell avec `-ExecutionPolicy Bypass` au niveau du processus uniquement, de sorte que les scripts `.ps1` et les importations de modules fonctionnent sur les installations Windows par défaut sans modifier la politique de la machine. Le contournement au niveau du processus ne remplace pas la politique de groupe `MachinePolicy` ou `UserPolicy`. Pour respecter la politique d'exécution effective de la machine à la place, définissez `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY=1`.

### Sélection du shell dans les paramètres, les hooks et les skills

Trois paramètres supplémentaires contrôlent où PowerShell est utilisé :

* `"defaultShell": "powershell"` dans `settings.json` : achemine les commandes interactives `!` via PowerShell. Nécessite que l'outil PowerShell soit activé.
* `"shell": "powershell"` sur les hooks de commande individuels : exécute ce hook dans PowerShell. Les hooks lancent PowerShell directement, donc cela fonctionne indépendamment de `CLAUDE_CODE_USE_POWERSHELL_TOOL`.
* `shell: powershell` dans le frontmatter de skill : exécute les blocs `` !`command` `` dans PowerShell. Nécessite que l'outil PowerShell soit activé.

### Limitations de l'aperçu

L'outil PowerShell a les limitations connues suivantes pendant l'aperçu :

* Les profils PowerShell ne sont pas chargés
* Sur Windows, le sandboxing n'est pas pris en charge

## Comportement de l'outil Read

L'outil Read prend un chemin de fichier et retourne le contenu avec les numéros de ligne. Claude est instruit de toujours transmettre les chemins absolus.

Par défaut, Read retourne le fichier depuis le début. Lorsqu'une lecture de fichier complet dépasse la limite de jetons, Read retourne la première page avec un avis `PARTIAL view` qui indique à Claude quelle quantité du fichier il a reçue et comment lire plus avec `offset` et `limit`. Une lecture qui transmet un `offset` ou `limit` explicite et dépasse toujours la limite de jetons retourne une erreur.

Read gère plusieurs types de fichiers au-delà du texte simple :

* **Images** : PNG, JPG et autres formats d'image sont retournés en tant que contenu visuel que Claude peut voir, pas en tant que bytes bruts. Claude Code redimensionne et récompresse les grandes images pour s'adapter aux limites de taille d'image du modèle avant de les envoyer.
* **PDFs** : Claude lit les fichiers `.pdf` courts en entier. Pour les PDFs plus longs que 10 pages, il lit par plages avec un paramètre `pages`, tel que `"1-5"`, jusqu'à 20 pages à la fois.
* **Notebooks Jupyter** : les fichiers `.ipynb` retournent toutes les cellules avec leurs sorties, y compris le code, le markdown et les visualisations.

Read ne lit que les fichiers, pas les répertoires. Claude utilise `ls` via l'outil Bash pour lister le contenu des répertoires.

## Comportement de l'outil WebFetch

WebFetch prend une URL et une invite décrivant ce qu'il faut extraire. Il récupère la page, convertit la réponse en Markdown quand le serveur retourne du HTML, et exécute l'invite contre le contenu en utilisant un petit modèle rapide. Pour la plupart des récupérations, Claude reçoit la réponse de ce modèle, pas la page brute. L'étape de conversion n'est pas configurable.

Cela rend WebFetch lossy par conception. L'invite d'extraction détermine ce qui atteint Claude, donc un résultat qui dit qu'une page ne mentionne pas quelque chose peut seulement signifier que l'invite ne l'a pas demandé.

Quelques comportements façonnent la réponse que Claude reçoit :

* Les URLs HTTP sont automatiquement mises à niveau vers HTTPS.
* Les grandes pages sont tronquées à une limite de caractères fixe avant le traitement.
* Les réponses sont mises en cache pendant 15 minutes, donc les récupérations répétées de la même URL retournent rapidement.
* Quand une URL redirige vers un hôte différent, WebFetch retourne un résultat textuel qui nomme l'URL d'origine et la cible de redirection au lieu de la suivre. Claude récupère ensuite la nouvelle URL avec un deuxième appel WebFetch.

Dans les modes de permission par défaut et `acceptEdits`, WebFetch demande la première fois qu'il atteint un nouveau domaine. Pour autoriser un domaine à l'avance sans demande, ajoutez une règle de permission comme `WebFetch(domain:example.com)`. Les modes `auto` et `bypassPermissions` ignorent complètement la demande.

WebFetch définit un en-tête `User-Agent` commençant par `Claude-User`, et un en-tête `Accept` qui préfère Markdown à HTML.

## Comportement de l'outil WebSearch

WebSearch exécute une requête contre le backend de recherche web d'Anthropic et retourne les titres et URLs des résultats. Il ne récupère pas les pages de résultats. Pour lire une page que Claude trouve dans les résultats de recherche, il suit avec WebFetch.

L'outil peut émettre jusqu'à huit recherches backend par appel, affinant la recherche en interne avant de retourner les résultats. Claude peut limiter les résultats avec `allowed_domains` pour inclure uniquement certains hôtes, ou `blocked_domains` pour les exclure. Les deux listes ne peuvent pas être combinées dans un seul appel.

Le backend de recherche n'est pas configurable. Pour rechercher avec un fournisseur différent, ajoutez un serveur MCP qui expose un outil de recherche.

Les règles de permission WebSearch ne prennent pas de spécificateur. Une entrée `WebSearch` nu dans `allow` ou `deny` est la seule forme.

WebSearch est disponible sur l'API Claude et Microsoft Foundry. Sur Google Cloud Vertex AI, il fonctionne avec les modèles Claude 4, y compris Opus, Sonnet et Haiku. Amazon Bedrock n'expose pas l'outil de recherche web côté serveur.

## Comportement de l'outil Write

L'outil Write crée un nouveau fichier ou remplace un fichier existant avec le contenu complet fourni. Il n'ajoute pas ou ne fusionne pas.

Si le chemin cible existe déjà, Claude doit avoir lu ce fichier au moins une fois dans la conversation actuelle avant de le remplacer. Une Write vers un fichier existant non lu échoue avec une erreur. Cette contrainte ne s'applique pas aux nouveaux fichiers.

Afficher le fichier avec Bash satisfait également cette exigence selon les mêmes règles décrites dans le comportement de l'outil Edit.

Pour les modifications partielles d'un fichier existant, Claude utilise Edit au lieu de Write.

## Vérifier quels outils sont disponibles

Votre ensemble d'outils exact dépend de votre fournisseur, de votre plateforme et de vos paramètres. Pour vérifier ce qui est chargé dans une session en cours d'exécution, demandez directement à Claude :

```
What tools do you have access to?
```

Claude donne un résumé conversationnel. Pour les noms d'outils MCP exacts, exécutez `/mcp`.

## Voir aussi

* Serveurs MCP : ajouter des outils personnalisés en connectant des serveurs externes
* Permissions : système de permission, syntaxe des règles, et motifs spécifiques aux outils
* Subagents : configurer l'accès aux outils pour les subagents
* Hooks : exécuter des commandes personnalisées avant ou après l'exécution des outils
