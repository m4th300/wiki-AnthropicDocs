# Comment Claude se souvient de votre projet (CLAUDE.md + Auto Memory)

Source: https://code.claude.com/docs/fr/memory

## Deux mécanismes de mémoire

| | Fichiers CLAUDE.md | Mémoire automatique |
|--|-------------------|---------------------|
| Qui l'écrit | Vous | Claude |
| Ce qu'il contient | Instructions et règles | Apprentissages et modèles |
| Portée | Projet, utilisateur ou organisation | Par référentiel, partagé entre worktrees |
| Chargé | Chaque session | Premières 200 lignes ou 25 KB de MEMORY.md |
| À utiliser pour | Normes de codage, flux de travail, architecture | Commandes de build, insights de débogage |

Les deux sont du contexte (pas une configuration appliquée). Pour bloquer une action indépendamment de Claude, utiliser un hook PreToolUse.

## Fichiers CLAUDE.md

### Emplacements et portée (du plus large au plus spécifique)

| Portée | Emplacement | Partagé avec |
|--------|------------|-------------|
| Politique gérée | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS), `/etc/claude-code/CLAUDE.md` (Linux) | Tous les utilisateurs |
| Utilisateur | `~/.claude/CLAUDE.md` | Juste vous (tous projets) |
| Projet | `./CLAUDE.md` ou `./.claude/CLAUDE.md` | Équipe via contrôle de source |
| Local | `./CLAUDE.local.md` (ajouter à .gitignore) | Juste vous (projet actuel) |

Les fichiers sont concaténés (pas remplacés) dans l'ordre de chargement.

### Création

```bash
/init    # Génère automatiquement un CLAUDE.md de démarrage
# CLAUDE_CODE_NEW_INIT=1 pour flux interactif multi-phases
```

### Bonnes pratiques d'écriture

- < 200 lignes par fichier (plus long = moins d'adhérence)
- Headers markdown + puces (pas de paragraphes denses)
- Spécifique et vérifiable : "Utiliser l'indentation à 2 espaces" vs "Formater le code"
- Pas d'instructions conflictuelles entre les fichiers CLAUDE.md

### Imports

```markdown
Consultez @README pour un aperçu du projet.
# Instructions supplémentaires
- flux de travail git @docs/git-instructions.md
```

Les imports sont chargés au démarrage (comptent dans le contexte). Profondeur max : 4 niveaux.

### AGENTS.md

Claude lit `CLAUDE.md`, pas `AGENTS.md`. Pour compatibilité :
```markdown
@AGENTS.md
## Instructions spécifiques Claude Code
...
```

### Comment les fichiers se chargent

En remontant depuis le répertoire de travail : `foo/bar/CLAUDE.md` + `foo/CLAUDE.md` + fichiers CLAUDE.local.md à côté.
Les fichiers dans les sous-répertoires se chargent à la demande (quand Claude lit des fichiers de ce sous-répertoire).

### Exclure des CLAUDE.md

```json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

## `.claude/rules/` (organisation avancée)

Fichiers markdown dans `.claude/rules/`. Avec frontmatter `paths:` → chargés seulement quand Claude lit des fichiers correspondants.

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# Règles API
- Tous les endpoints doivent inclure validation des entrées
```

Pattern glob | Correspond à
`**/*.ts` | Tous TypeScript
`src/**/*` | Tout sous `src/`
`*.md` | Markdown à la racine

```markdown
---
paths:
  - "src/**/*.{ts,tsx}"
  - "lib/**/*.ts"
---
```

Les règles sans `paths` se chargent au démarrage (comme CLAUDE.md).
Les règles avec `paths` se chargent à la demande.

## Mémoire automatique

### Activation

Par défaut activée. Désactiver :
- `/memory` dans une session → bascule
- `autoMemoryEnabled: false` dans settings.json
- `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`

### Emplacement

`~/.claude/projects/<project>/memory/` (dérivé du référentiel git)
Tous les worktrees du même repo partagent le même répertoire.

Personnaliser : `autoMemoryDirectory: "~/my-custom-memory-dir"` dans settings.json

### Structure

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # Index concis (chargé à chaque session, 200 premières lignes/25KB)
├── debugging.md       # Notes détaillées sur les modèles de débogage
├── api-conventions.md # Décisions de conception d'API
└── ...
```

MEMORY.md = index. Autres fichiers = notes détaillées chargées à la demande.

### Utilisation

- Quand vous voyez "Writing memory" ou "Recalled memory" → Claude met à jour les fichiers
- Pour demander à Claude de se souvenir : "toujours utiliser pnpm, pas npm" → sauvegardé en auto memory
- Pour ajouter à CLAUDE.md : "ajouter ceci à CLAUDE.md" ou éditer manuellement
- `/memory` → voir et éditer tous les fichiers de mémoire

## Commande `/memory`

Liste tous les fichiers CLAUDE.md, CLAUDE.local.md et rules chargés dans la session.
Bascule auto memory activée/désactivée.
Lien pour ouvrir le dossier de mémoire automatique.

## Dépannage

**Claude ne suit pas mon CLAUDE.md**
- `/memory` pour vérifier que le fichier est chargé
- Rendre les instructions plus spécifiques
- Vérifier les conflits entre les fichiers CLAUDE.md

**Instructions perdues après `/compact`**
- CLAUDE.md racine survit à la compaction et se recharge du disque
- CLAUDE.md imbriqués se rechargent à la demande
- Instructions dans la conversation seulement → à ajouter dans CLAUDE.md

**CLAUDE.md trop volumineux**
- Utiliser les règles spécifiques au chemin (`.claude/rules/`)
- Réduire le contenu non nécessaire à chaque session
