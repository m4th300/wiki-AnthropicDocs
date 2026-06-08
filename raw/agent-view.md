# Vue Agent — Gérer plusieurs agents

> Lancez et gérez plusieurs sessions Claude Code à partir d'un seul écran.

La vue agent, ouverte avec `claude agents`, affiche toutes vos sessions en arrière-plan : ce qui s'exécute, ce qui a besoin d'intervention, et ce qui est terminé.

Note: La vue agent est un aperçu de recherche et nécessite Claude Code v2.1.139+.

## Démarrage rapide

### 1. Ouvrir la vue agent
```bash
claude agents
```

Appuyer sur `Esc` pour revenir au shell. Les sessions continuent de s'exécuter.

### 2. Dispatcher une session
Taper une invite et appuyer sur `Entrée`. Chaque invite dans la vue agent démarre sa propre nouvelle session (pas un suivi à une session existante).

### 3. Jeter un coup d'œil et répondre
- Sélectionner une ligne avec les touches fléchées
- Appuyer sur `Espace` pour ouvrir le panneau d'aperçu
- Taper une réponse et `Entrée` pour l'envoyer sans quitter la vue

### 4. S'attacher et se détacher
- `Entrée` ou `→` : s'attacher (session Claude Code interactive complète)
- `←` sur invite vide : se détacher et revenir au tableau

### 5. Mettre une session existante en arrière-plan
- `/bg` dans la session, ou
- `←` sur invite vide pour mettre en arrière-plan ET ouvrir la vue agent

## Surveiller les sessions

```text
Épinglées
  ✽ clawd walk cycle          Write assets/sprites/clawd-walk.png    3m

Prêtes pour examen
  ∙ jump physics              Opened PR with collision fix      PR #2048  2h

Nécessite une intervention
  ✻ power-up design           needs input: double jump or wall climb?   1m

En cours
  ✽ collision detection       Edit src/physics/CollisionSystem.ts       2m

Terminées
  ✻ title screen              result: menu, options, and credits done   9m
```

Sessions groupées par état: épinglées → nécessitent intervention → en cours → terminées.

## Filtrer par projet

```bash
claude agents --cwd ~/projects/my-app
```

Nécessite Claude Code v2.1.141+. Affiche uniquement les sessions de ce répertoire et ses worktrees.

## Lancer de nouveaux agents

### Depuis la vue agent
Taper directement dans le champ en bas de la vue.

### Depuis l'intérieur d'une session
Utiliser `/dispatch "Task description"` pour lancer une nouvelle session en arrière-plan.

### Depuis le shell
```bash
claude --bg "Fix the authentication bug"
```

## Gérer les sessions depuis le shell

```bash
# Lister les sessions en arrière-plan
claude sessions list

# S'attacher à une session spécifique
claude sessions attach <session-id>

# Arrêter une session
claude sessions stop <session-id>
```

## Architecture (sessions en arrière-plan)

Les sessions en arrière-plan s'exécutent via un processus superviseur (daemon). Quand vous fermez la vue agent ou votre terminal, les sessions continuent.

Le superviseur:
- Gère le cycle de vie des processus
- Persiste l'état des sessions
- Coordonne la communication entre la vue agent et les sessions

## Modes de permission et modèle

La nouvelle session utilise:
- Le modèle affiché dans l'en-tête de la vue agent
- Le même mode de permission que `claude` dans ce répertoire

## Limitations

- Chaque session utilise votre quota d'abonnement indépendamment
- Les sous-agents et coéquipiers ne sont pas listés comme lignes séparées
- Les sessions interactives d'autres terminaux n'apparaissent qu'après `/bg`
