# Personnaliser les raccourcis clavier

> Personnalisez les raccourcis clavier dans Claude Code avec un fichier de configuration de keybindings.

> Les raccourcis clavier personnalisables nécessitent Claude Code v2.1.18 ou une version ultérieure. Vérifiez votre version avec `claude --version`.

Claude Code prend en charge les raccourcis clavier personnalisables. Exécutez `/keybindings` pour créer ou ouvrir votre fichier de configuration à `~/.claude/keybindings.json`.

## Fichier de configuration

Le fichier de configuration des keybindings est un objet avec un tableau `bindings`. Chaque bloc spécifie un contexte et une carte de touches aux actions.

Les modifications du fichier keybindings sont automatiquement détectées et appliquées sans redémarrer Claude Code.

| Champ | Description |
| :---- | :---------- |
| `$schema` | URL JSON Schema optionnelle pour l'autocomplétion de l'éditeur |
| `$docs` | URL de documentation optionnelle |
| `bindings` | Tableau de blocs de bindings par contexte |

Cet exemple lie `Ctrl+E` pour ouvrir un éditeur externe dans le contexte chat, et délie `Ctrl+U` :

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null
      }
    }
  ]
}
```

## Contextes

Chaque bloc de binding spécifie un **contexte** où les bindings s'appliquent :

| Contexte | Description |
| :------- | :---------- |
| `Global` | S'applique partout dans l'application |
| `Chat` | Zone de saisie principale du chat |
| `Autocomplete` | Le menu d'autocomplétion est ouvert |
| `Settings` | Menu des paramètres |
| `Confirmation` | Dialogs de permission et de confirmation |
| `Tabs` | Composants de navigation par onglets |
| `Help` | Le menu d'aide est visible |
| `Transcript` | Visionneuse de transcription |
| `HistorySearch` | Mode de recherche dans l'historique (Ctrl+R) |
| `Task` | Une tâche en arrière-plan est en cours d'exécution |
| `ThemePicker` | Dialog de sélection de thème |
| `Attachments` | Navigation des pièces jointes d'images dans les dialogs de sélection |
| `Footer` | Navigation des indicateurs de pied de page (tâches, équipes, diff) |
| `MessageSelector` | Sélection de messages pour le dialog de retour en arrière et résumé |
| `DiffDialog` | Navigation dans le visualiseur de diff |
| `ModelPicker` | Niveau d'effort du sélecteur de modèle |
| `Select` | Composants de sélection/liste génériques |
| `Plugin` | Dialog de plugin (parcourir, découvrir, gérer) |
| `Scroll` | Défilement de la conversation et sélection de texte en mode plein écran |
| `Doctor` | Écran de diagnostics `/doctor` |

## Actions disponibles

Les actions suivent un format `namespace:action`, comme `chat:submit` pour envoyer un message ou `app:toggleTodos` pour afficher la liste des tâches. Chaque contexte a des actions spécifiques disponibles.

### Actions App

Actions disponibles dans le contexte `Global` :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `app:interrupt` | Ctrl+C | Annuler l'opération courante |
| `app:exit` | Ctrl+D | Quitter Claude Code |
| `app:redraw` | (non lié) | Forcer le redessin du terminal |
| `app:toggleTodos` | Ctrl+T | Basculer la visibilité de la liste des tâches |
| `app:toggleTranscript` | Ctrl+O | Basculer la transcription détaillée |

### Actions History

Actions pour naviguer dans l'historique des commandes :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `history:search` | Ctrl+R | Ouvrir la recherche d'historique |
| `history:previous` | Up | Élément d'historique précédent |
| `history:next` | Down | Élément d'historique suivant |

### Actions Chat

Actions disponibles dans le contexte `Chat` :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `chat:cancel` | Escape | Annuler la saisie courante |
| `chat:clearInput` | Ctrl+L | Forcer un redessin complet de l'écran, en préservant la saisie |
| `chat:clearScreen` | Cmd+K | En rendu plein écran, appuyer deux fois en deux secondes pour exécuter `/clear` |
| `chat:killAgents` | Ctrl+X Ctrl+K | Tuer tous les sous-agents d'arrière-plan en cours d'exécution dans cette session |
| `chat:cycleMode` | Shift+Tab* | Cycle des modes de permission |
| `chat:modelPicker` | Meta+P | Ouvrir le sélecteur de modèle |
| `chat:fastMode` | Meta+O | Basculer le mode rapide |
| `chat:thinkingToggle` | Meta+T | Basculer la réflexion étendue |
| `chat:submit` | Enter | Soumettre le message |
| `chat:newline` | Ctrl+J | Insérer une nouvelle ligne sans soumettre |
| `chat:undo` | Ctrl+\_, Ctrl+Shift+- | Annuler la dernière action |
| `chat:externalEditor` | Ctrl+G, Ctrl+X Ctrl+E | Ouvrir dans un éditeur externe |
| `chat:stash` | Ctrl+S | Mettre en attente l'invite courante |
| `chat:imagePaste` | Ctrl+V (Alt+V sur Windows et WSL) | Coller une image depuis le presse-papier |

*Sur Windows sans mode VT (Node <24.2.0/<22.17.0, Bun <1.2.23), par défaut Meta+M.

### Actions Autocomplete

Actions disponibles dans le contexte `Autocomplete` :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `autocomplete:accept` | Tab | Accepter la suggestion |
| `autocomplete:dismiss` | Escape | Fermer le menu |
| `autocomplete:previous` | Up | Suggestion précédente |
| `autocomplete:next` | Down | Suggestion suivante |

### Actions Confirmation

Actions disponibles dans le contexte `Confirmation` :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `confirm:yes` | Y, Enter | Confirmer l'action |
| `confirm:no` | N, Escape | Refuser l'action |
| `confirm:previous` | Up | Option précédente |
| `confirm:next` | Down | Option suivante |
| `confirm:nextField` | Tab | Champ suivant |
| `confirm:previousField` | (non lié) | Champ précédent |
| `confirm:toggle` | Space | Basculer la sélection |
| `confirm:cycleMode` | Shift+Tab | Cycle des modes de permission |
| `confirm:toggleExplanation` | Ctrl+E | Basculer l'explication de permission |

### Actions Transcript

Actions disponibles dans le contexte `Transcript` :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `transcript:toggleShowAll` | Ctrl+E | Basculer afficher tout le contenu |
| `transcript:exit` | q, Ctrl+C, Escape | Quitter la vue transcription |

### Actions History Search

Actions disponibles dans le contexte `HistorySearch` :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `historySearch:next` | Ctrl+R | Correspondance suivante |
| `historySearch:accept` | Escape, Tab | Accepter la sélection |
| `historySearch:cancel` | Ctrl+C | Annuler la recherche |
| `historySearch:execute` | Enter | Exécuter la commande sélectionnée |
| `historySearch:cycleScope` | Ctrl+S | Cycle de portée : session, projet, partout |

### Actions Task

Actions disponibles dans le contexte `Task` :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `task:background` | Ctrl+B | Mettre en arrière-plan la tâche courante |

### Actions Settings

Actions disponibles dans le contexte `Settings` :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `settings:search` | / | Entrer en mode recherche |
| `settings:retry` | R | Réessayer le chargement des données d'utilisation (en cas d'erreur) |
| `settings:close` | Enter | Sauvegarder les modifications et fermer le panel de config |

### Actions Scroll

Actions disponibles dans le contexte `Scroll` quand le rendu plein écran est activé :

| Action | Défaut | Description |
| :----- | :----- | :---------- |
| `scroll:lineUp` | (non lié) | Défiler d'une ligne vers le haut |
| `scroll:lineDown` | (non lié) | Défiler d'une ligne vers le bas |
| `scroll:pageUp` | PageUp | Défiler de la moitié de la hauteur de la fenêtre vers le haut |
| `scroll:pageDown` | PageDown | Défiler de la moitié de la hauteur de la fenêtre vers le bas |
| `scroll:top` | Ctrl+Home | Aller au début de la conversation |
| `scroll:bottom` | Ctrl+End | Aller au dernier message et réactiver le suivi automatique |
| `selection:copy` | Ctrl+Shift+C / Cmd+C | Copier le texte sélectionné dans le presse-papier |

## Syntaxe des touches

### Modificateurs

Utilisez les touches de modification avec le séparateur `+` :

* `ctrl` ou `control` - Touche Control
* `shift` - Touche Shift
* `alt`, `opt`, `option`, ou `meta` - Touche Alt sur Windows et Linux, Option sur macOS
* `cmd`, `command`, `super`, ou `win` - Touche Command sur macOS, Windows sur Windows, Super sur Linux

Exemples :

```text
ctrl+k          Ctrl + K
shift+tab       Shift + Tab
meta+p          Option + P sur macOS, Alt + P ailleurs
ctrl+shift+c    Modificateurs multiples
```

### Lettres majuscules

Une lettre majuscule seule implique Shift. Par exemple, `K` équivaut à `shift+k`.

### Accords (Chords)

Les accords sont des séquences de touches séparées par des espaces :

```text
ctrl+k ctrl+s   Appuyer sur Ctrl+K, relâcher, puis Ctrl+S
```

### Touches spéciales

* `escape` ou `esc` - Touche Escape
* `enter` ou `return` - Touche Enter
* `tab` - Touche Tab
* `space` - Barre d'espace
* `up`, `down`, `left`, `right` - Touches fléchées
* `backspace`, `delete` - Touches de suppression

## Délier les raccourcis par défaut

Définissez une action à `null` pour délier un raccourci par défaut :

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+s": null
      }
    }
  ]
}
```

## Raccourcis réservés

Ces raccourcis ne peuvent pas être reliés :

| Raccourci | Raison |
| :-------- | :----- |
| Ctrl+C | Interruption/annulation codée en dur |
| Ctrl+D | Sortie codée en dur |
| Ctrl+M | Identique à Enter dans les terminaux (les deux envoient CR) |
| Caps Lock | Non livré aux applications terminales |

## Conflits de terminal

Certains raccourcis peuvent entrer en conflit avec les multiplexeurs de terminal :

| Raccourci | Conflit |
| :-------- | :------ |
| Ctrl+B | Préfixe tmux (appuyer deux fois pour envoyer) |
| Ctrl+A | Préfixe GNU screen |
| Ctrl+Z | Suspension de processus Unix (SIGTSTP) |

## Interaction avec le mode Vim

Quand le mode vim est activé via `/config` → Mode éditeur, les keybindings et le mode vim fonctionnent indépendamment :

* **Mode Vim** gère la saisie au niveau de l'entrée de texte (mouvement du curseur, modes, motions)
* **Keybindings** gèrent les actions au niveau du composant (basculer les todos, soumettre, etc.)
* La touche Escape en mode vim passe d'INSERT à NORMAL ; elle ne déclenche pas `chat:cancel`
* La plupart des raccourcis Ctrl+touche passent à travers le mode vim vers le système de keybindings

## Validation

Claude Code valide vos keybindings et affiche des avertissements pour :

* Erreurs d'analyse (JSON invalide ou structure invalide)
* Noms de contexte invalides
* Conflits avec des raccourcis réservés
* Conflits avec des multiplexeurs de terminal
* Bindings en double dans le même contexte

Exécutez `/doctor` pour voir les avertissements de keybindings.
