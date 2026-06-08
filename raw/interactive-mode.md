# Mode interactif

Source: https://code.claude.com/docs/fr/interactive-mode

## Raccourcis clavier essentiels

### Contrôles généraux

| Raccourci | Description |
|---------|-------------|
| `Ctrl+C` | Interrompre / effacer l'entrée |
| `Ctrl+D` | Quitter |
| `Esc` | Interrompre Claude (conserve le travail effectué) |
| `Esc + Esc` | Effacer le brouillon OU ouvrir le menu de rembobinage |
| `Shift+Tab` | Basculer les modes de permission (default/acceptEdits/plan/auto) |
| `Ctrl+O` | Basculer la visionneuse de transcription (détails d'outils, appels MCP) |
| `Ctrl+G` ou `Ctrl+X Ctrl+E` | Ouvrir dans l'éditeur de texte par défaut |
| `Ctrl+R` | Recherche inversée dans l'historique des commandes |
| `Ctrl+L` | Redessiner l'écran |
| `Ctrl+T` | Basculer la liste des tâches |
| `Ctrl+B` | Tâches en arrière-plan (Tmux : appuyer deux fois) |
| `Option+P` (Mac) / `Alt+P` | Changer de modèle |
| `Option+T` (Mac) / `Alt+T` | Basculer la réflexion étendue |
| `Option+O` (Mac) / `Alt+O` | Basculer le mode rapide |

### Coller une image

- macOS : `Ctrl+V` ou `Cmd+V` (iTerm2)
- Windows/WSL : `Alt+V`

### Raccourcis d'édition

| Raccourci | Description |
|---------|-------------|
| `Ctrl+A` / `Ctrl+E` | Début / fin de ligne |
| `Ctrl+K` | Supprimer jusqu'à la fin |
| `Ctrl+U` | Supprimer jusqu'au début |
| `Ctrl+W` | Supprimer le mot précédent |
| `Ctrl+Y` | Coller le texte supprimé |
| `Alt+B` / `Alt+F` | Naviguer par mot (nécessite Option=Meta sur macOS) |

### Entrée multiligne

| Méthode | Raccourci |
|---------|----------|
| Universel | `\` + Entrée |
| iTerm2, WezTerm, Ghostty, Kitty, Warp | `Shift+Entrée` (natif) |
| VS Code, Cursor, etc. | `/terminal-setup` pour installer |
| N'importe quel terminal | `Ctrl+J` |

### Commandes rapides

| Préfixe | Action |
|---------|--------|
| `/` | Commande ou skill |
| `!` | Mode shell (exécution directe + ajout au contexte) |
| `@` | Autocomplétion de chemin de fichier |

## /btw — Questions latérales

```
/btw what was the name of that config file again?
```

- Question rapide sans ajouter au contexte principal
- Disponible PENDANT que Claude travaille (n'interrompt pas)
- Pas d'accès aux outils (répond depuis le contexte existant)
- Réponse unique (pas de suivi dans la superposition)
- Touches dans la superposition : Espace/Entrée/Esc → rejeter, `f` → diviser en session, `x` → effacer la liste

## Mode éditeur Vim

Activer via `/config` → Mode éditeur.

### Modes de Vim

- `Esc` : entrer en NORMAL depuis INSERT ou VISUAL
- `i`/`a`/`o`/`O` : entrer en INSERT
- `v`/`V` : entrer en VISUAL

### Navigation (NORMAL)

`h`/`j`/`k`/`l`, `w`/`e`/`b`, `0`/`$`/`^`, `gg`/`G`, `f{char}`/`F{char}`, `t{char}`, `;`/`,`

### Édition (NORMAL)

`x` (supprimer char), `dd` (ligne), `D` (jusqu'à fin), `dw`/`de`/`db`, `cc`/`C`/`cw`, `yy`/`yw`, `p`/`P`, `>>`/`<<`, `J`, `u`, `.`

### Mode VISUAL

`d`/`x`, `y`, `c`/`s`, `r{char}`, `~`/`u`/`U`, `>`/`<`, `J`, `o`

**Note** : En mode normal, `j`/`k` sur le bord de l'entrée → navigation dans l'historique.

## Commandes en arrière-plan

```bash
! npm test      # Mode shell
```

- `Ctrl+B` : déplacer une invocation bash vers l'arrière-plan
- Sortie écrite dans un fichier, Claude peut la récupérer avec Read
- ID unique pour chaque tâche
- Nettoyées quand Claude Code se ferme
- Désactiver : `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`

## Liste des tâches

Pour le travail multi-étapes, Claude crée une liste de tâches dans la zone d'état.
`Ctrl+T` pour basculer. Affiche jusqu'à 5 tâches à la fois.
`CLAUDE_CODE_TASK_LIST_ID=my-project claude` pour partager entre sessions.

## Récapitulatif de session

Apparaît quand vous revenez après 3+ minutes d'absence (après 3+ tours).
`/recap` pour générer à la demande.
Désactiver : `/config` → désactiver "Récapitulatif de session".

## Statut de révision PR

Badge PR cliquable dans le pied de page si une branche a une PR ouverte.
Couleurs : vert (approuvé), jaune (en attente), rouge (modifications demandées), gris (brouillon).
Nécessite `gh` CLI installé et authentifié.

## Suggestions d'invite

Apparaissent grisées dans l'entrée d'invite.
Tab ou Flèche droite pour accepter, commencer à taper pour rejeter.
Désactiver : `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false`

## Configuration Option comme Meta (macOS)

Pour raccourcis Alt+B, Alt+F, Alt+Y, Alt+M, Alt+P :
- iTerm2 : Paramètres → Profils → Touches → Option gauche/droite = "Esc+"
- Terminal Apple : Paramètres → Profils → Clavier → "Utiliser Option comme touche Meta"
- VS Code : `"terminal.integrated.macOptionIsMeta": true`
