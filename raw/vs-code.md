# Utiliser Claude Code dans VS Code

Source: https://code.claude.com/docs/fr/vs-code

## Installation

VS Code 1.98.0+ requis.
- [Installer pour VS Code](vscode:extension/anthropic.claude-code)
- [Installer pour Cursor](cursor:extension/anthropic.claude-code)
- Ou rechercher "Claude Code" dans Extensions (Cmd+Shift+X)

Fonctionne aussi dans : Devin Desktop, Kiro, autres forks VS Code.

## Démarrer

1. Ouvrir le panneau Claude Code :
   - Icône Spark en haut à droite de l'éditeur (nécessite un fichier ouvert)
   - Barre d'activité (côté gauche)
   - Palette de commandes : Cmd+Shift+P → "Claude Code"
   - Barre d'état : cliquer "✱ Claude Code" en bas à droite
2. Se connecter (première fois)
3. Envoyer une invite

## Zone de saisie

- **Modes de permission** : cliquer l'indicateur de mode en bas (Normal/Plan/Auto-accepter)
- **Menu de commandes** : `/` ou taper `/` → voir commandes, modèles, réflexion étendue, Remote Control
- **Indicateur de contexte** : affiche l'utilisation de la context window
- **Réflexion étendue** : via menu `/`. `Ctrl+O` pour développer/réduire les blocs de réflexion
- **Entrée multiligne** : `Shift+Entrée`

## Références de fichiers

- `@nom-fichier` : Claude lit le fichier complet
- `@répertoire/` (avec slash) : liste du répertoire
- Sélection dans l'éditeur : Claude voit automatiquement le texte sélectionné
- `Option+K` (Mac) / `Alt+K` (Win/Linux) : insérer mention @ avec chemin + numéros de ligne
- Glisser-déposer fichiers dans la zone de saisie (tenir Shift)
- PDF : demander des pages spécifiques

## Commandes et raccourcis

| Commande | Raccourci |
|----------|----------|
| Focus Input (basculer éditeur↔Claude) | `Cmd+Esc` / `Ctrl+Esc` |
| Open in New Tab | `Cmd+Shift+Esc` / `Ctrl+Shift+Esc` |
| New Conversation | `Cmd+N` / `Ctrl+N` (si `enableNewConversationShortcut: true`) |
| Reopen Closed Session | `Cmd+Shift+T` / `Ctrl+Shift+T` |
| Insert @-Mention Reference | `Option+K` / `Alt+K` |

**macOS Tahoe** : `Cmd+Esc` peut être pris par Game Overlay → Paramètres système → Clavier → Raccourcis → Contrôleurs de jeu → Décocher Game Overlay.

## Sessions et historique

- Bouton **Historique des sessions** en haut du panneau
- Rechercher par mot-clé ou parcourir par heure
- Onglet **Distant** pour les sessions cloud claude.ai (nécessite claude.ai Subscription)

## Personnalisation

- **Position** : glisser le panneau (barre latérale secondaire, principale, ou onglet d'éditeur)
- **Plusieurs conversations** : "Ouvrir dans un nouvel onglet" ou "dans une nouvelle fenêtre"
- **Mode terminal** : paramètre `claudeCode.useTerminal = true`

## Paramètres de l'extension

| Paramètre | Défaut | Description |
|-----------|--------|-------------|
| `useTerminal` | false | Lancer en mode terminal |
| `initialPermissionMode` | default | Mode de permission initial |
| `preferredLocation` | panel | `sidebar` ou `panel` |
| `autosave` | true | Enregistrement auto avant lecture/écriture |
| `useCtrlEnterToSend` | false | Ctrl+Entrée au lieu d'Entrée |
| `respectGitIgnore` | true | Exclure les patterns .gitignore |
| `allowDangerouslySkipPermissions` | false | Ajouter Bypass permissions au sélecteur |

## Plugins

`/plugins` pour ouvrir l'interface de gestion :
- Onglet Plugins : installer/désactiver, chercher
- Onglet Marchés : ajouter/supprimer des sources
- Portée d'installation : vous uniquement, projet, ou locale

## Chrome

`@browser` dans la zone de saisie :
```text
@browser go to localhost:3000 and check the console for errors
```
Nécessite extension "Claude in Chrome" v1.0.36+.

## Serveur MCP IDE intégré

L'extension exécute un serveur MCP local nommé `ide` (non configurable, masqué de `/mcp`).
- Port aléatoire sur `127.0.0.1` uniquement
- Jeton d'authentification aléatoire par session (dans `~/.claude/ide/`, permissions 0600)
- Outils visibles par Claude : `mcp__ide__getDiagnostics`, `mcp__ide__executeCode` (Jupyter)
- `executeCode` demande TOUJOURS confirmation dans VS Code avant exécution

## Extension VS Code vs CLI

| Fonctionnalité | CLI | Extension |
|----------------|-----|-----------|
| Toutes les commandes/skills | Oui | Sous-ensemble |
| Configuration MCP | Oui | Partiel |
| Checkpoints | Oui | Oui |
| Raccourci bash `!` | Oui | Non |
| Complétion Tab | Oui | Non |

## Désinstallation

Extensions → Claude Code → Désinstaller
Supprimer les données : `rm -rf ~/.vscode/globalStorage/anthropic.claude-code`

## Lien URI handler

```
vscode://anthropic.claude-code/open?prompt=review%20my%20changes
vscode://anthropic.claude-code/open?session=<session-id>
```
