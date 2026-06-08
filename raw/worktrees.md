# Exécuter des sessions parallèles avec worktrees

Source: https://code.claude.com/docs/fr/worktrees

## Ce que c'est

Un git worktree = répertoire de travail séparé avec ses propres fichiers et branche, partageant le même historique et télécommande. Sessions Claude parallèles sans conflit entre elles.

L'app Desktop crée automatiquement un worktree pour chaque nouvelle session.

## Démarrer Claude dans un worktree

```bash
claude --worktree feature-auth     # créé sous .claude/worktrees/feature-auth/ sur branche worktree-feature-auth
claude --worktree                  # nom aléatoire (ex: bright-running-fox)
claude -w bugfix-123               # alias court
claude --worktree "#1234"          # à partir d'une PR GitHub
```

La branche de base par défaut : `origin/HEAD` (branche par défaut du repo télécommande).
Si pas de télécommande → `HEAD` local.

Configurer pour partir du HEAD local :
```json
{ "worktree": { "baseRef": "head" } }
```

Avant d'utiliser `--worktree` dans un répertoire, accepter la boîte de dialogue de confiance une première fois avec `claude` simple.

## Copier les fichiers gitignorés

`.worktreeinclude` à la racine du projet :
```text
.env
.env.local
config/secrets.json
```
Seuls les fichiers correspondants ET gitignorés sont copiés. Les fichiers suivis ne sont jamais dupliqués.

Ajouter `.claude/worktrees/` au `.gitignore`.

## Isoler les subagents avec worktrees

Demander à Claude : "utilise les worktrees pour tes agents"
Ou dans le frontmatter d'un subagent : `isolation: worktree`

Chaque subagent obtient un worktree temporaire, supprimé automatiquement s'il n'y a pas de modifications.

## Branche de base des worktrees de subagents

Identique à `--worktree` : `origin/HEAD` sauf si `worktree.baseRef: "head"`.

## Nettoyage

- **Pas de modifications** : worktree et branche supprimés automatiquement
  - Si la session a un nom → Claude demande si conserver pour plus tard
- **Des modifications existent** : Claude demande de conserver ou supprimer
  - Supprimer = supprime répertoire, branche, et TOUTES les modifications (non validées ET commits)
- **Exécutions non-interactives** (`-p`) : pas de nettoyage automatique → utiliser `git worktree remove`

Worktrees de subagents et sessions en arrière-plan : supprimés automatiquement après `cleanupPeriodDays` si pas de modifications non validées ni commits non poussés.

## Gestion manuelle

```bash
git worktree add ../project-feature-a -b feature-a    # nouvelle branche
git worktree add ../project-bugfix bugfix-123          # branche existante
cd ../project-feature-a && claude                     # démarrer Claude
git worktree list                                      # lister
git worktree remove ../project-feature-a              # supprimer
```

## Contrôle de version non-git

Configurer les hooks `WorktreeCreate` et `WorktreeRemove` pour SVN, Perforce, Mercurial, etc.

```json
{
  "hooks": {
    "WorktreeCreate": [{
      "hooks": [{
        "type": "command",
        "command": "bash -c 'NAME=$(jq -r .name); DIR=\"$HOME/.claude/worktrees/$NAME\"; svn checkout https://svn.example.com/repo/trunk \"$DIR\" >&2 && echo \"$DIR\"'"
      }]
    }]
  }
}
```

Le hook remplace entièrement la logique `git worktree` par défaut (`.worktreeinclude` n'est pas traité).
