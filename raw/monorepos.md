# Configurer Claude Code dans un monorepo ou une grande base de code

> Configurez Claude Code pour les monorepos et les grandes bases de code à arbre unique avec des fichiers CLAUDE.md imbriqués, des worktrees épars, l'intelligence de code et des compétences par package afin que Claude reste concentré sur le code sur lequel vous travaillez.

Une grande base de code peut être un dépôt avec des millions de lignes ou un monorepo avec de nombreux packages. Claude Code fonctionne à n'importe quelle taille, mais à mesure que la base de code grandit, les valeurs par défaut réglées pour les petits projets peuvent remplir la fenêtre de contexte avec des instructions et des lectures de fichiers sans rapport avec la tâche.

## Ce que ce guide couvre

Chaque paramètre ci-dessous est indépendant. Ils se superposent plutôt que de se remplacer.

| Je veux | Utiliser |
| :------ | :------- |
| Charger uniquement les conventions pour le code que vous touchez | Fichiers CLAUDE.md par répertoire |
| Exclure les fichiers CLAUDE.md pour les packages sur lesquels je ne travaille jamais | `claudeMdExcludes` |
| Bloquer Claude d'ouvrir la sortie de build, le code généré et les dépendances vendored | Règles de refus `Read` dans `permissions.deny` |
| Trouver la définition ou les appelants d'un symbole via le serveur de langage | Un plugin d'intelligence de code |
| N'extraire que les répertoires nécessaires quand Claude crée un worktree | `worktree.sparsePaths` |
| Lire et modifier un package voisin ou un autre dépôt depuis la même session | `--add-dir` ou `additionalDirectories` |
| Donner à Claude des procédures spécifiques à une zone qui se chargent uniquement quand c'est pertinent | Compétences par répertoire |
| Remplacer de nombreux fichiers CLAUDE.md par répertoire par un ensemble de conventions que tout le monde installe | Un plugin dans un marketplace interne |

## L'exemple de monorepo

```text
monorepo/
  CLAUDE.md                     # instructions racine
  packages/
    api/
      CLAUDE.md                 # instructions spécifiques à l'API
      .claude/skills/
      src/
    web/
      CLAUDE.md                 # instructions spécifiques au frontend
      .claude/skills/
      src/
    shared/
      CLAUDE.md                 # instructions de la bibliothèque partagée
      src/
```

## Choisir où démarrer Claude

| Démarrer depuis | Accès aux fichiers | CLAUDE.md chargé au démarrage | Utiliser quand |
| :-------------- | :----------------- | :---------------------------- | :------------- |
| Racine du dépôt | Chaque fichier | Racine uniquement ; les fichiers de sous-répertoires se chargent à la demande | Les tâches couvrent plusieurs packages ou sous-systèmes |
| Un sous-répertoire | Ce sous-arbre uniquement | Celui du répertoire plus tous les ancêtres | Le travail est limité à un package ou sous-système |

Les paramètres de projet dans `.claude/settings.json` se chargent uniquement depuis votre répertoire de démarrage et ne sont pas hérités des répertoires parents comme le sont les fichiers CLAUDE.md.

## Superposer les fichiers CLAUDE.md par répertoire

Claude Code charge chaque fichier CLAUDE.md depuis votre répertoire de travail et chaque répertoire parent au démarrage, puis charge le fichier de chaque sous-répertoire à la demande quand il y lit des fichiers.

Un découpage courant est deux niveaux :

* **CLAUDE.md racine** : instructions qui s'appliquent partout (normes de codage, conventions de commit, disposition du dépôt)
* **CLAUDE.md par sous-répertoire** : conventions spécifiques à la pile de cette zone

Exemple de CLAUDE.md racine :

```markdown
Ce dépôt est un monorepo avec trois packages sous packages/ :

- packages/api: API REST Node.js avec Express, TypeScript et PostgreSQL
- packages/web: Frontend React avec Vite, TypeScript et TailwindCSS
- packages/shared: utilitaires TypeScript partagés

Exécuter les commandes depuis le répertoire du package, pas la racine du monorepo.
```

Exemple de `packages/api/CLAUDE.md` :

```markdown
Ce package est le serveur API REST.

- Exécuter les tests : `npm test` (utilise Vitest)
- Exécuter le serveur de développement : `npm run dev` (port 3001)
- Migrations de base de données : `npm run migrate`
- Variables d'environnement : copier `.env.example` vers `.env`

Les routes API sont dans src/routes/. Les requêtes DB utilisent Knex dans src/db/.
```

### Exclure les fichiers CLAUDE.md non pertinents

Le paramètre `claudeMdExcludes` ignore des fichiers spécifiques par chemin ou motif glob :

```json
{
  "claudeMdExcludes": [
    "**/packages/admin-dashboard/**",
    "**/packages/legacy-*/**"
  ]
}
```

Placez ce paramètre dans `.claude/settings.local.json` (gitignored) pour un usage personnel.

## Réduire ce que Claude lit

### Bloquer les lectures de code généré et vendored

Ajoutez des règles de refus `Read` dans `permissions.deny` :

```json
{
  "permissions": {
    "deny": [
      "Read(./**/dist/**)",
      "Read(./**/build/**)",
      "Read(./**/*.generated.*)",
      "Read(./vendor/**)"
    ]
  }
}
```

### Réduire les lectures de fichiers avec l'intelligence de code

Les plugins d'intelligence de code connectent Claude à un serveur de langage afin qu'il puisse sauter aux définitions, trouver les références et corriger les erreurs de type directement au lieu de parcourir l'arbre.

```shell
/plugin install typescript-lsp@claude-plugins-official
```

## Limiter les worktrees et l'accès aux fichiers

### N'extraire que les répertoires nécessaires

Le paramètre `worktree.sparsePaths` utilise la sparse-checkout git pour n'écrire que les répertoires listés sur le disque :

```json
{
  "worktree": {
    "sparsePaths": [
      ".claude",
      "packages/api",
      "packages/shared"
    ]
  }
}
```

Pour éviter de dupliquer de grands répertoires comme `node_modules` dans les worktrees, associez `sparsePaths` avec `symlinkDirectories` :

```json
{
  "worktree": {
    "sparsePaths": [
      ".claude",
      "packages/api",
      "packages/shared"
    ],
    "symlinkDirectories": [
      "node_modules"
    ]
  }
}
```

### Accorder l'accès à travers les packages ou les dépôts

Le paramètre `additionalDirectories` dans `.claude/settings.json` donne à Claude accès aux répertoires en dehors du répertoire de travail :

```json
{
  "permissions": {
    "additionalDirectories": [
      "../shared",
      "../web"
    ]
  }
}
```

Vous pouvez également accorder l'accès à l'exécution sans modifier les paramètres en passant `--add-dir` :

```bash
claude --add-dir ../shared
```

| Ajouté avec | Charge CLAUDE.md et règles | Charge les compétences |
| :---------- | :------------------------- | :--------------------- |
| Paramètre `additionalDirectories` | Jamais | Jamais |
| Drapeau `--add-dir` ou commande `/add-dir` | Uniquement avec la variable d'environnement ci-dessous | Oui |

Pour charger les fichiers CLAUDE.md et les règles depuis un répertoire ajouté avec `--add-dir` :

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared
```

## Ajouter des compétences par répertoire

Tout sous-répertoire peut définir des compétences limitées à sa propre pile. Une compétence se charge à la demande quand Claude détermine qu'elle est pertinente.

Les compétences résident sous `.claude/skills/` à l'intérieur du répertoire.

Créer un répertoire de compétences :

```bash
mkdir -p packages/api/.claude/skills/api-testing
```

Exemple de `packages/api/.claude/skills/api-testing/SKILL.md` :

```markdown
---
name: api-testing
description: Modèles de test pour le package API. Utiliser lors de l'écriture ou de la modification de tests dans packages/api/.
---

## Structure des tests

Les tests sont dans `src/__tests__/` en miroir de la structure `src/`.

## Exécuter les tests

- Tous les tests : `npm test`
- Fichier unique : `npm test -- src/__tests__/routes/users.test.ts`
- Mode watch : `npm test -- --watch`

## Patterns

- Utiliser `supertest` pour les assertions HTTP, pas fetch brut
- Toujours envelopper les tests de base de données dans une transaction qui se rembobine
```

## Tout mettre ensemble

Configuration combinée pour `packages/api/.claude/settings.json` :

```json
{
  "worktree": {
    "sparsePaths": [
      ".claude",
      "packages/api",
      "packages/shared"
    ],
    "symlinkDirectories": [
      "node_modules"
    ]
  },
  "permissions": {
    "additionalDirectories": [
      "../shared"
    ],
    "deny": [
      "Read(./**/dist/**)",
      "Read(./**/build/**)"
    ]
  }
}
```

Et dans `.claude/settings.json` à la racine du dépôt (pour les sessions de worktree) :

```json
{
  "permissions": {
    "deny": [
      "Read(./**/dist/**)",
      "Read(./**/build/**)"
    ]
  }
}
```

Disposition finale du dépôt :

```text
monorepo/
  CLAUDE.md
  .claude/settings.json                           # règles de refus pour les sessions de worktree
  packages/
    api/
      CLAUDE.md
      .claude/settings.json                       # worktree, additionalDirectories, règles de refus
      .claude/skills/api-testing/SKILL.md
    web/
      CLAUDE.md
      .claude/skills/component-patterns/SKILL.md
    shared/
      CLAUDE.md
```

## Centraliser les conventions quand la superposition ne s'adapte plus

Quand les fichiers CLAUDE.md par répertoire deviennent difficiles à gouverner, déplacez les conventions vers des mécanismes qui se chargent à la demande :

* **Compétences** : matériel de référence que Claude charge uniquement quand c'est pertinent pour la tâche
* **Plugins** : bundles versionnés de compétences, hooks et commandes qu'une équipe plateforme possède centralement
* **Serveurs MCP** : si votre organisation exécute déjà un index de recherche de code ou RAG, exposez-le comme outil MCP

## Étapes suivantes

* Utilisez les hooks pour exécuter des linters ou des vérificateurs de type par répertoire après les modifications de fichiers de Claude
* Consultez Gérer les coûts efficacement pour comprendre comment la taille de la base de code affecte l'utilisation de jetons
