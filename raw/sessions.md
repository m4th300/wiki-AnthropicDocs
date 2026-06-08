# Gérer les sessions

Source: https://code.claude.com/docs/fr/sessions

## Ce que c'est

Une session = conversation enregistrée liée à un répertoire de projet, stockée localement en continu.

Permet de reprendre, brancher, et basculer entre les conversations.

## Reprendre une session

| Commande | Action |
|----------|--------|
| `claude --continue` | Dernière session dans le répertoire courant |
| `claude --resume` | Ouvrir le sélecteur de sessions |
| `claude --resume <name>` | Reprendre directement par nom |
| `claude --from-pr <number>` | Reprendre la session liée à cette PR |
| `/resume` | Basculer depuis une session active |

Les sessions créées avec `claude -p` ou le SDK Agent n'apparaissent pas dans le sélecteur (mais reprennables avec `claude --resume <session-id>`).

## Sélecteur de sessions

Le sélecteur affiche par défaut les sessions du worktree courant.

| Raccourci | Action |
|----------|--------|
| `↑` / `↓` | Naviguer |
| `→` / `←` | Développer/réduire groupes |
| `Enter` | Reprendre |
| `Space` | Prévisualiser |
| `Ctrl+R` | Renommer |
| `/` ou caractère | Mode recherche |
| `Ctrl+A` | Afficher tous les projets |
| `Ctrl+W` | Afficher tous les worktrees du repo |
| `Ctrl+B` | Filtrer par branche git courante |
| `Esc` | Quitter |

Rechercher : coller une URL de PR/MR GitHub/GitLab/Bitbucket → trouve la session correspondante.

## Nommer les sessions

```bash
claude -n auth-refactor              # au démarrage
/rename auth-refactor                # pendant une session
```

Dans le sélecteur : mettre en surbrillance → `Ctrl+R`.

Reprendre par nom : `claude --resume auth-refactor` ou `/resume auth-refactor`.

L'acceptation d'un plan en plan mode nomme automatiquement la session depuis le contenu du plan.

## Résolution des noms entre worktrees

| Commande | Correspondance exacte | Nom ambigu |
|----------|----------------------|------------|
| `claude --resume <name>` | Reprend directement | Ouvre sélecteur avec nom pré-rempli |
| `/resume <name>` | Reprend directement | Signale une erreur |

## Brancher une session

Créer une copie de la conversation en cours, laisser l'original intact.

```text
/branch try-streaming-approach
```

```bash
claude --continue --fork-session
```

Les permissions "autoriser pour cette session" ne sont PAS reportées à la nouvelle branche.

Si la même session est ouverte dans deux terminaux sans brancher → les messages s'entrelacent dans une seule transcription.

## Gérer le contexte dans une session

- `/clear` : recommencer avec contexte vide (session précédente reste accessible)
- `/compact [instructions]` : remplacer l'historique par un résumé
- `/context` : voir ce qui consomme le contexte

## Exporter et localiser les données

```text
/export                        # copier dans le presse-papiers
/export session-backup.txt     # écrire dans un fichier
```

### Stockage

Chemin : `~/.claude/projects/<project>/<session-id>.jsonl`
`<project>` = dérivé du chemin du répertoire de travail.

Changer l'emplacement : `CLAUDE_CONFIG_DIR`
Durée de conservation : 30 jours par défaut (modifier avec `cleanupPeriodDays`)

### Désactiver l'écriture de transcription

- `CLAUDE_CODE_SKIP_PROMPT_HISTORY` (toutes sessions)
- `--no-session-persistence` (mode non-interactif uniquement)
