# Référence des plugins

> Référence technique complète du système de plugins Claude Code, incluant les schémas, les commandes CLI et les spécifications des composants.

Cette référence fournit les spécifications techniques complètes du système de plugins Claude Code, incluant les schémas de composants, les commandes CLI et les outils de développement.

Un **plugin** est un répertoire autonome de composants qui étend Claude Code avec des fonctionnalités personnalisées. Les composants de plugin incluent les skills, les agents, les hooks, les serveurs MCP, les serveurs LSP et les moniteurs.

## Référence des composants de plugin

### Skills

Les plugins ajoutent des skills à Claude Code, créant des raccourcis `/name` que vous ou Claude pouvez invoquer.

**Emplacement** : répertoire `skills/` ou `commands/` à la racine du plugin, ou un seul fichier `SKILL.md` à la racine du plugin

**Format de fichier** : Les skills sont des répertoires avec `SKILL.md` ; les commandes sont des fichiers markdown simples

**Structure des skills** :

```text
skills/
├── pdf-processor/
│   ├── SKILL.md
│   ├── reference.md (optionnel)
│   └── scripts/ (optionnel)
└── code-reviewer/
    └── SKILL.md
```

**Comportement d'intégration** :

* Les skills et les commandes sont découverts automatiquement lors de l'installation du plugin
* Claude peut les invoquer automatiquement en fonction du contexte de la tâche
* Les skills peuvent inclure des fichiers de support à côté de SKILL.md

### Agents

Les plugins peuvent fournir des subagents spécialisés pour des tâches spécifiques que Claude peut invoquer automatiquement si approprié.

**Emplacement** : répertoire `agents/` à la racine du plugin

**Format de fichier** : Fichiers markdown décrivant les capacités de l'agent

**Structure de l'agent** :

```markdown
---
name: agent-name
description: Ce dans quoi cet agent se spécialise et quand Claude devrait l'invoquer
model: sonnet
effort: medium
maxTurns: 20
disallowedTools: Write, Edit
---

Invite système détaillée pour l'agent décrivant son rôle, son expertise et son comportement.
```

Les agents de plugin prennent en charge les champs frontmatter `name`, `description`, `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `skills`, `memory`, `background` et `isolation`. La seule valeur `isolation` valide est `"worktree"`. Pour des raisons de sécurité, `hooks`, `mcpServers` et `permissionMode` ne sont pas pris en charge pour les agents fournis par les plugins.

**Points d'intégration** :

* Les agents apparaissent dans l'interface `/agents`
* Claude peut invoquer les agents automatiquement en fonction du contexte de la tâche
* Les agents peuvent être invoqués manuellement par les utilisateurs
* Les agents de plugin fonctionnent aux côtés des agents Claude intégrés

### Hooks

Les plugins peuvent fournir des gestionnaires d'événements qui répondent automatiquement aux événements de Claude Code.

**Emplacement** : `hooks/hooks.json` à la racine du plugin, ou en ligne dans plugin.json

**Format** : Configuration JSON avec des correspondances d'événements et des actions

**Configuration des hooks** :

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

Les hooks de plugin répondent aux mêmes événements de cycle de vie que les hooks définis par l'utilisateur :

| Event                 | When it fires                                                                                                                                          |
| :-------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SessionStart`        | When a session begins or resumes                                                                                                                       |
| `Setup`               | When you start Claude Code with `--init-only`, or with `--init` or `--maintenance` in `-p` mode                                                       |
| `UserPromptSubmit`    | When you submit a prompt, before Claude processes it                                                                                                   |
| `UserPromptExpansion` | When a user-typed command expands into a prompt, before it reaches Claude. Can block the expansion                                                     |
| `PreToolUse`          | Before a tool call executes. Can block it                                                                                                              |
| `PermissionRequest`   | When a permission dialog appears                                                                                                                       |
| `PermissionDenied`    | When a tool call is denied by the auto mode classifier. Return `{retry: true}` to tell the model it may retry the denied tool call                     |
| `PostToolUse`         | After a tool call succeeds                                                                                                                             |
| `PostToolUseFailure`  | After a tool call fails                                                                                                                                |
| `PostToolBatch`       | After a full batch of parallel tool calls resolves, before the next model call                                                                         |
| `Notification`        | When Claude Code sends a notification                                                                                                                  |
| `MessageDisplay`      | While assistant message text is displayed                                                                                                              |
| `SubagentStart`       | When a subagent is spawned                                                                                                                             |
| `SubagentStop`        | When a subagent finishes                                                                                                                               |
| `TaskCreated`         | When a task is being created via `TaskCreate`                                                                                                          |
| `TaskCompleted`       | When a task is being marked as completed                                                                                                               |
| `Stop`                | When Claude finishes responding                                                                                                                        |
| `StopFailure`         | When the turn ends due to an API error. Output and exit code are ignored                                                                               |
| `TeammateIdle`        | When an agent team teammate is about to go idle                                                                                                        |
| `InstructionsLoaded`  | When a CLAUDE.md or `.claude/rules/*.md` file is loaded into context                                                                                  |
| `ConfigChange`        | When a configuration file changes during a session                                                                                                     |
| `CwdChanged`          | When the working directory changes                                                                                                                     |
| `FileChanged`         | When a watched file changes on disk. The `matcher` field specifies which filenames to watch                                                            |
| `WorktreeCreate`      | When a worktree is being created via `--worktree` or `isolation: "worktree"`                                                                           |
| `WorktreeRemove`      | When a worktree is being removed                                                                                                                       |
| `PreCompact`          | Before context compaction                                                                                                                              |
| `PostCompact`         | After context compaction completes                                                                                                                     |
| `Elicitation`         | When an MCP server requests user input during a tool call                                                                                              |
| `ElicitationResult`   | After a user responds to an MCP elicitation, before the response is sent back to the server                                                            |
| `SessionEnd`          | When a session terminates                                                                                                                              |

**Types de hooks** :

* `command` : exécuter des commandes shell ou des scripts
* `http` : envoyer l'événement JSON en tant que requête POST à une URL
* `mcp_tool` : appeler un outil sur un serveur MCP configuré
* `prompt` : évaluer une invite avec un LLM (utilise l'espace réservé `$ARGUMENTS` pour le contexte)
* `agent` : exécuter un vérificateur agentic avec des outils pour les tâches de vérification complexes

### Serveurs MCP

Les plugins peuvent regrouper des serveurs Model Context Protocol (MCP) pour connecter Claude Code avec des outils et services externes.

**Emplacement** : `.mcp.json` à la racine du plugin, ou en ligne dans plugin.json

**Format** : Configuration standard du serveur MCP

**Configuration du serveur MCP** :

```json
{
  "mcpServers": {
    "plugin-database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DB_PATH": "${CLAUDE_PLUGIN_ROOT}/data"
      }
    },
    "plugin-api-client": {
      "command": "npx",
      "args": ["@company/mcp-server", "--plugin-mode"],
      "cwd": "${CLAUDE_PLUGIN_ROOT}"
    }
  }
}
```

**Comportement d'intégration** :

* Les serveurs MCP de plugin démarrent automatiquement quand le plugin est activé
* Les serveurs apparaissent comme des outils MCP standard dans la boîte à outils de Claude
* Les capacités du serveur s'intègrent de manière transparente avec les outils existants de Claude
* Les serveurs de plugin peuvent être configurés indépendamment des serveurs MCP de l'utilisateur

### Serveurs LSP

Les plugins peuvent fournir des serveurs Language Server Protocol (LSP) pour donner à Claude une intelligence de code en temps réel lors du travail sur votre base de code.

L'intégration LSP fournit :

* **Diagnostics instantanés** : Claude voit les erreurs et les avertissements immédiatement après chaque modification
* **Navigation de code** : aller à la définition, trouver les références et les informations au survol
* **Sensibilisation au langage** : informations de type et documentation pour les symboles de code

**Emplacement** : `.lsp.json` à la racine du plugin, ou en ligne dans `plugin.json`

**Format du fichier `.lsp.json`** :

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

**En ligne dans `plugin.json`** :

```json
{
  "name": "my-plugin",
  "lspServers": {
    "go": {
      "command": "gopls",
      "args": ["serve"],
      "extensionToLanguage": {
        ".go": "go"
      }
    }
  }
}
```

**Champs obligatoires :**

| Champ                 | Description                                                 |
| :-------------------- | :---------------------------------------------------------- |
| `command`             | Le binaire LSP à exécuter (doit être dans PATH)             |
| `extensionToLanguage` | Mappe les extensions de fichier aux identifiants de langage |

**Champs optionnels :**

| Champ                   | Description                                                     |
| :---------------------- | :-------------------------------------------------------------- |
| `args`                  | Arguments de ligne de commande pour le serveur LSP              |
| `transport`             | Transport de communication : `stdio` (par défaut) ou `socket`   |
| `env`                   | Variables d'environnement à définir au démarrage du serveur     |
| `initializationOptions` | Options transmises au serveur lors de l'initialisation          |
| `settings`              | Paramètres transmis via `workspace/didChangeConfiguration`      |
| `workspaceFolder`       | Chemin du dossier de l'espace de travail pour le serveur        |
| `startupTimeout`        | Temps maximum d'attente du démarrage du serveur (millisecondes) |
| `maxRestarts`           | Nombre maximum de tentatives de redémarrage avant d'abandonner  |

**Plugins LSP disponibles :**

| Plugin              | Serveur de langage         | Commande d'installation                                                                          |
| :------------------ | :------------------------- | :----------------------------------------------------------------------------------------------- |
| `pyright-lsp`       | Pyright (Python)           | `pip install pyright` ou `npm install -g pyright`                                                |
| `typescript-lsp`    | TypeScript Language Server | `npm install -g typescript-language-server typescript`                                           |
| `rust-analyzer-lsp` | rust-analyzer              | Voir l'installation de rust-analyzer                                                             |

### Moniteurs

Les plugins peuvent déclarer des moniteurs en arrière-plan que Claude Code démarre automatiquement quand le plugin est actif. Chaque moniteur exécute une commande shell pour la durée de la session et livre chaque ligne stdout à Claude en tant que notification.

Les moniteurs de plugin nécessitent Claude Code v2.1.105 ou ultérieur.

**Emplacement** : `monitors/monitors.json` à la racine du plugin, ou en ligne dans `plugin.json`

**Format** : Tableau JSON d'entrées de moniteur

```json
[
  {
    "name": "deploy-status",
    "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/poll-deploy.sh ${user_config.api_endpoint}",
    "description": "Changements de statut de déploiement"
  },
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Journal d'erreurs de l'application",
    "when": "on-skill-invoke:debug"
  }
]
```

**Champs obligatoires :**

| Champ         | Description                                                                                                                           |
| :------------ | :------------------------------------------------------------------------------------------------------------------------------------ |
| `name`        | Identifiant unique dans le plugin                                                                                                     |
| `command`     | Commande shell exécutée en tant que processus en arrière-plan persistant                                                              |
| `description` | Résumé court de ce qui est surveillé                                                                                                  |

**Champs optionnels :**

| Champ  | Description                                                                                                                       |
| :----- | :-------------------------------------------------------------------------------------------------------------------------------- |
| `when` | `"always"` (défaut) ou `"on-skill-invoke:<skill-name>"` pour démarrer uniquement quand la skill nommée est invoquée               |

### Thèmes

Les plugins peuvent livrer des thèmes de couleur qui apparaissent dans `/theme`. Un thème est un fichier JSON dans `themes/` avec un préset `base` et une carte `overrides`.

```json
{
  "name": "Dracula",
  "base": "dark",
  "overrides": {
    "claude": "#bd93f9",
    "error": "#ff5555",
    "success": "#50fa7b"
  }
}
```

---

## Portées d'installation des plugins

Quand vous installez un plugin, vous choisissez une **portée** qui détermine où le plugin est disponible :

| Portée    | Fichier de paramètres                           | Cas d'usage                                                       |
| :-------- | :---------------------------------------------- | :---------------------------------------------------------------- |
| `user`    | `~/.claude/settings.json`                       | Plugins personnels disponibles dans tous les projets (par défaut) |
| `project` | `.claude/settings.json`                         | Plugins d'équipe partagés via le contrôle de version              |
| `local`   | `.claude/settings.local.json`                   | Plugins spécifiques au projet, ignorés par git                    |
| `managed` | Paramètres gérés                                | Plugins gérés (lecture seule, mise à jour uniquement)             |

---

## Plugins du répertoire des skills

Tout dossier sous un répertoire de skills qui contient un manifeste `.claude-plugin/plugin.json` est chargé en tant que plugin nommé `<name>@skills-dir` à la session suivante, sans marketplace et sans étape d'installation.

| Ce que vous avez                              | Ce que c'est                                                                             |
| :-------------------------------------------- | :--------------------------------------------------------------------------------------- |
| `<skills-dir>/foo/SKILL.md` sans manifeste    | Une skill simple nommée `foo`                                                            |
| `<skills-dir>/foo/.claude-plugin/plugin.json` | Un plugin `foo@skills-dir`, qui peut regrouper ses propres skills, agents, hooks et plus |
| `<plugin>/skills/bar/SKILL.md`                | Une skill `bar` emballée à l'intérieur d'un plugin                                       |

### Choisir d'où le plugin se charge

| Répertoire de skills    | Portée    | Se charge                                                                                                                   |
| :---------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------- |
| `~/.claude/skills/`     | personnel | Dans chaque projet                                                                                                          |
| `<cwd>/.claude/skills/` | projet    | Seulement après acceptation de la boîte de dialogue de confiance de l'espace de travail                                     |

---

## Schéma du manifeste du plugin

Le fichier `.claude-plugin/plugin.json` définit les métadonnées et la configuration de votre plugin. Le manifeste est optionnel — sans lui, Claude Code découvre automatiquement les composants dans les emplacements par défaut.

### Schéma complet

```json
{
  "name": "plugin-name",
  "displayName": "Plugin Name",
  "version": "1.2.0",
  "description": "Brief plugin description",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://github.com/author"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/author/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "skills": "./custom/skills/",
  "commands": ["./custom/commands/special.md"],
  "agents": ["./custom/agents/reviewer.md"],
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "outputStyles": "./styles/",
  "lspServers": "./.lsp.json",
  "experimental": {
    "themes": "./themes/",
    "monitors": "./monitors.json"
  },
  "dependencies": [
    "helper-lib",
    { "name": "secrets-vault", "version": "~2.1.0" }
  ]
}
```

### Champs obligatoires

Si vous incluez un manifeste, `name` est le seul champ obligatoire.

| Champ  | Type   | Description                                    | Exemple              |
| :----- | :----- | :--------------------------------------------- | :------------------- |
| `name` | string | Identifiant unique (kebab-case, pas d'espaces) | `"deployment-tools"` |

### Champs de métadonnées

| Champ            | Type    | Description                                                                                                                      | Exemple                                                           |
| :--------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------- |
| `$schema`        | string  | URL du schéma JSON pour l'autocomplétion                                                                                         | `"https://json.schemastore.org/claude-code-plugin-manifest.json"` |
| `displayName`    | string  | Nom lisible affiché dans le sélecteur `/plugin`. Nécessite Claude Code v2.1.143 ou ultérieur                                     | `"Deployment Tools"`                                              |
| `version`        | string  | Version sémantique. Si omise, Claude Code utilise le SHA du commit git                                                           | `"2.1.0"`                                                         |
| `description`    | string  | Explication brève de l'objectif du plugin                                                                                        | `"Deployment automation tools"`                                   |
| `author`         | object  | Informations sur l'auteur                                                                                                        | `{"name": "Dev Team", "email": "dev@company.com"}`                |
| `homepage`       | string  | URL de documentation                                                                                                             | `"https://docs.example.com"`                                      |
| `repository`     | string  | URL du code source                                                                                                               | `"https://github.com/user/plugin"`                                |
| `license`        | string  | Identifiant de licence                                                                                                           | `"MIT"`, `"Apache-2.0"`                                           |
| `keywords`       | array   | Balises de découverte                                                                                                            | `["deployment", "ci-cd"]`                                         |
| `defaultEnabled` | boolean | Si le plugin démarre activé. Nécessite Claude Code v2.1.154 ou ultérieur. Par défaut `true`                                      | `false`                                                           |

### Champs de chemin de composant

| Champ                   | Type                  | Description                                                                                                                                                             |
| :---------------------- | :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `skills`                | string\|array         | Répertoires de skills personnalisés contenant `<name>/SKILL.md`                                                                                                         |
| `commands`              | string\|array         | Fichiers de skill markdown plats personnalisés ou répertoires                                                                                                           |
| `agents`                | string\|array         | Fichiers d'agents personnalisés                                                                                                                                         |
| `hooks`                 | string\|array\|object | Chemins de configuration des hooks ou configuration en ligne                                                                                                            |
| `mcpServers`            | string\|array\|object | Chemins de configuration MCP ou configuration en ligne                                                                                                                  |
| `outputStyles`          | string\|array         | Fichiers/répertoires de styles de sortie personnalisés                                                                                                                  |
| `lspServers`            | string\|array\|object | Configurations Language Server Protocol                                                                                                                                 |
| `experimental.themes`   | string\|array         | Fichiers/répertoires de thèmes de couleur                                                                                                                               |
| `experimental.monitors` | string\|array         | Configurations de Monitor en arrière-plan                                                                                                                               |
| `userConfig`            | object                | Valeurs configurables par l'utilisateur demandées au moment de l'activation                                                                                             |
| `channels`              | array                 | Déclarations de canaux pour l'injection de messages                                                                                                                     |
| `dependencies`          | array                 | Autres plugins requis, optionnellement avec des contraintes de version semver                                                                                           |

### Configuration utilisateur

Le champ `userConfig` déclare les valeurs que Claude Code demande à l'utilisateur lors de l'activation du plugin :

```json
{
  "userConfig": {
    "api_endpoint": {
      "type": "string",
      "title": "Point de terminaison API",
      "description": "Le point de terminaison API de votre équipe"
    },
    "api_token": {
      "type": "string",
      "title": "Jeton API",
      "description": "Jeton d'authentification API",
      "sensitive": true
    }
  }
}
```

| Champ         | Obligatoire | Description                                                                                         |
| :------------ | :---------- | :-------------------------------------------------------------------------------------------------- |
| `type`        | Oui         | L'un de `string`, `number`, `boolean`, `directory`, ou `file`                                       |
| `title`       | Oui         | Étiquette affichée dans la boîte de dialogue de configuration                                       |
| `description` | Oui         | Texte d'aide affiché sous le champ                                                                  |
| `sensitive`   | Non         | Si `true`, masque l'entrée et stocke la valeur dans le stockage sécurisé                            |
| `required`    | Non         | Si `true`, la validation échoue quand le champ est vide                                             |
| `default`     | Non         | Valeur utilisée quand l'utilisateur ne fournit rien                                                 |
| `multiple`    | Non         | Pour le type `string`, autoriser un tableau de chaînes                                              |
| `min` / `max` | Non         | Limites pour le type `number`                                                                       |

Chaque valeur est disponible via `${user_config.KEY}` dans les configurations MCP/LSP, les hooks et les moniteurs. Les valeurs sont aussi exportées en tant que `CLAUDE_PLUGIN_OPTION_<KEY>`.

### Variables d'environnement

| Variable                | Description                                                                                             |
| :---------------------- | :------------------------------------------------------------------------------------------------------ |
| `${CLAUDE_PLUGIN_ROOT}` | Chemin absolu du répertoire d'installation du plugin. Change à chaque mise à jour du plugin             |
| `${CLAUDE_PLUGIN_DATA}` | Répertoire persistant pour l'état du plugin qui survit aux mises à jour (`~/.claude/plugins/data/{id}/`) |
| `${CLAUDE_PROJECT_DIR}` | La racine du projet                                                                                     |

---

## Mise en cache des plugins et résolution des fichiers

Les plugins de marketplace sont copiés dans le cache local (`~/.claude/plugins/cache`) plutôt que d'être utilisés sur place. Chaque version installée est un répertoire séparé. Les répertoires de version orphelins sont supprimés après 7 jours.

### Limitations de traversée de répertoires

Les plugins installés ne peuvent pas référencer des fichiers en dehors de leur répertoire. Les chemins `../shared-utils` ne fonctionnent pas après l'installation.

### Partager des fichiers via des liens symboliques

| Cible du lien symbolique             | Comportement lors de la copie dans le cache            |
| :------------------------------------ | :----------------------------------------------------- |
| Au sein du répertoire propre du plugin | Préservé en tant que lien symbolique relatif           |
| Ailleurs dans la même marketplace     | Déréférencé — le contenu de la cible est copié        |
| En dehors de la marketplace           | Ignoré pour des raisons de sécurité                    |

---

## Structure du répertoire des plugins

### Disposition standard des plugins

```text
enterprise-plugin/
├── .claude-plugin/           # Répertoire de métadonnées (optionnel)
│   └── plugin.json             # manifeste du plugin
├── skills/                   # Skills
│   ├── code-reviewer/
│   │   └── SKILL.md
│   └── pdf-processor/
│       ├── SKILL.md
│       └── scripts/
├── commands/                 # Skills en tant que fichiers markdown plats
│   ├── status.md
│   └── logs.md
├── agents/                   # Définitions de subagent
│   ├── security-reviewer.md
│   ├── performance-tester.md
│   └── compliance-checker.md
├── output-styles/            # Définitions de style de sortie
│   └── terse.md
├── themes/                   # Définitions de thème de couleur
│   └── dracula.json
├── monitors/                 # Configurations de moniteur en arrière-plan
│   └── monitors.json
├── hooks/                    # Configurations des hooks
│   ├── hooks.json
│   └── security-hooks.json
├── bin/                      # Exécutables de plugin ajoutés à PATH
│   └── my-tool
├── settings.json            # Paramètres par défaut pour le plugin
├── .mcp.json                # Définitions du serveur MCP
├── .lsp.json                # Configurations du serveur LSP
├── scripts/                 # Scripts de hooks et d'utilitaires
│   ├── security-scan.sh
│   ├── format-code.py
│   └── deploy.js
├── LICENSE
└── CHANGELOG.md
```

### Référence des emplacements de fichiers

| Composant            | Emplacement par défaut       | Objectif                                                     |
| :------------------- | :--------------------------- | :------------------------------------------------------------ |
| **Manifeste**        | `.claude-plugin/plugin.json` | Métadonnées et configuration du plugin (optionnel)           |
| **Skills**           | `skills/`                    | Skills avec structure `<name>/SKILL.md`                      |
| **Commandes**        | `commands/`                  | Skills en tant que fichiers Markdown plats                   |
| **Agents**           | `agents/`                    | Fichiers Markdown de subagent                                |
| **Styles de sortie** | `output-styles/`             | Définitions de style de sortie                               |
| **Thèmes**           | `themes/`                    | Définitions de thème de couleur                              |
| **Hooks**            | `hooks/hooks.json`           | Configuration des hooks                                      |
| **Serveurs MCP**     | `.mcp.json`                  | Définitions du serveur MCP                                   |
| **Serveurs LSP**     | `.lsp.json`                  | Configurations du serveur de langage                         |
| **Moniteurs**        | `monitors/monitors.json`     | Configurations de moniteur en arrière-plan                   |
| **Exécutables**      | `bin/`                       | Exécutables ajoutés au `PATH` de l'outil Bash                |
| **Paramètres**       | `settings.json`              | Configuration par défaut appliquée quand le plugin est activé |

---

## Référence des commandes CLI

### plugin init

Générez un nouveau plugin à `~/.claude/skills/<name>/`.

```bash
claude plugin init <name> [options]
```

**Options :**

| Option                   | Description                                                                                                                          | Par défaut              |
| :----------------------- | :----------------------------------------------------------------------------------------------------------------------------------- | :---------------------- |
| `--description <text>`   | Description du manifeste                                                                                                             |                         |
| `--author <name>`        | Nom de l'auteur                                                                                                                      | `git config user.name`  |
| `--author-email <email>` | Email de l'auteur                                                                                                                    | `git config user.email` |
| `--with <components...>` | Générer également des dossiers de composants : `skills`, `agents`, `hooks`, `mcp`, `lsp`, `output-style`, `channel`                  |                         |
| `-f, --force`            | Écraser un `.claude-plugin/` existant                                                                                                |                         |

**Alias :** `new`

### plugin install

```bash
claude plugin install <plugin> [options]
```

| Option                | Description                                           | Par défaut |
| :-------------------- | :---------------------------------------------------- | :--------- |
| `-s, --scope <scope>` | Portée d'installation : `user`, `project`, ou `local` | `user`     |

### plugin uninstall

```bash
claude plugin uninstall <plugin> [options]
```

| Option                | Description                                                | Par défaut |
| :-------------------- | :--------------------------------------------------------- | :--------- |
| `-s, --scope <scope>` | Désinstaller de la portée                                  | `user`     |
| `--keep-data`         | Conserver le répertoire de données persistantes            |            |
| `--prune`             | Supprimer également les dépendances auto-installées        |            |
| `-y, --yes`           | Ignorer l'invite de confirmation                           |            |

**Alias :** `remove`, `rm`

### plugin prune

Supprimez les dépendances de plugins auto-installées qui ne sont plus requises.

```bash
claude plugin prune [options]
```

| Option                | Description                                 | Par défaut |
| :-------------------- | :------------------------------------------ | :--------- |
| `-s, --scope <scope>` | Nettoyer à la portée                        | `user`     |
| `--dry-run`           | Lister ce qui serait supprimé               |            |
| `-y, --yes`           | Ignorer l'invite de confirmation            |            |

Nécessite Claude Code v2.1.121 ou ultérieur. **Alias :** `autoremove`

### plugin enable / disable

```bash
claude plugin enable <plugin> [options]
claude plugin disable <plugin> [options]
```

### plugin update

```bash
claude plugin update <plugin> [options]
```

### plugin list

```bash
claude plugin list [options]
```

| Option        | Description                                                          |
| :------------ | :------------------------------------------------------------------- |
| `--json`      | Sortie en JSON                                                       |
| `--available` | Inclure les plugins disponibles des marketplaces. Nécessite `--json` |

### plugin details

Afficher l'inventaire des composants d'un plugin et le coût en tokens projeté.

```bash
claude plugin details <name>
```

La sortie affiche deux chiffres de coût pour chaque composant :
* **Always-on :** tokens ajoutés à chaque session
* **On-invoke :** tokens qu'un composant coûte quand il se déclenche

### plugin tag

Créez une balise de version git pour le plugin dans le répertoire actuel.

```bash
claude plugin tag [options]
```

| Option        | Description                                        |
| :------------ | :------------------------------------------------- |
| `--push`      | Pousser la balise vers le serveur distant          |
| `--dry-run`   | Afficher ce qui serait balisé sans créer la balise |
| `-f, --force` | Créer la balise même si l'arborescence est sale    |

---

## Outils de débogage et de développement

Utilisez `claude --debug` pour voir les détails du chargement des plugins.

### Problèmes courants

| Problème                            | Cause                              | Solution                                                               |
| :---------------------------------- | :--------------------------------- | :--------------------------------------------------------------------- |
| Plugin ne se charge pas             | `plugin.json` invalide             | Exécutez `claude plugin validate` pour vérifier les erreurs de schéma  |
| Les skills n'apparaissent pas       | Structure de répertoire incorrecte | Assurez-vous que `skills/` est à la racine du plugin                   |
| Les hooks ne se déclenchent pas     | Le script n'est pas exécutable     | Exécutez `chmod +x script.sh`                                          |
| Le serveur MCP échoue               | `${CLAUDE_PLUGIN_ROOT}` manquant   | Utilisez la variable pour tous les chemins de plugin                   |
| LSP `Executable not found in $PATH` | Serveur de langage non installé    | Installez le binaire (ex: `npm install -g typescript-language-server`) |

### Erreurs de validation du manifeste

* `Invalid JSON syntax: Unexpected token }` : vérifiez les virgules manquantes
* `Validation errors: name: Required` : un champ obligatoire est manquant
* `JSON parse error` : erreur de syntaxe JSON

---

## Référence de distribution et de versioning

### Gestion des versions

La version est résolue à partir du premier de ces éléments qui est défini :

1. Le champ `version` dans le `plugin.json` du plugin
2. Le champ `version` dans l'entrée marketplace du plugin dans `marketplace.json`
3. Le SHA du commit git du plugin source
4. `unknown`, pour les sources `npm` ou les répertoires locaux hors git

| Approche                  | Comment                                            | Comportement de mise à jour                            |
| :------------------------ | :------------------------------------------------- | :----------------------------------------------------- |
| **Version explicite**     | Définissez `"version": "2.1.0"` dans `plugin.json` | Mises à jour uniquement quand vous augmentez le champ  |
| **Version SHA du commit** | Omettez `version`                                  | Mises à jour à chaque nouveau commit vers la source git |

## Voir aussi

* [Plugins](/fr/plugins) - Tutoriels et utilisation pratique
* [Marketplaces de plugins](/fr/plugin-marketplaces) - Création et gestion des marketplaces
* [Skills](/fr/skills) - Détails du développement des skills
* [Subagents](/fr/sub-agents) - Configuration et capacités des agents
* [Hooks](/fr/hooks) - Gestion des événements et automatisation
* [MCP](/fr/mcp) - Intégration des outils externes
* [Paramètres](/fr/settings) - Options de configuration pour les plugins
