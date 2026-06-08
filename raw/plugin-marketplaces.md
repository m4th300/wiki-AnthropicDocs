# Créer et distribuer une place de marché de plugins

> Créez et hébergez des places de marché de plugins pour distribuer les extensions Claude Code dans vos équipes et communautés.

Une **place de marché de plugins** est un catalogue qui vous permet de distribuer des plugins à d'autres. Les places de marché offrent une découverte centralisée, un suivi des versions, des mises à jour automatiques et la prise en charge de plusieurs types de sources (dépôts git, chemins locaux, etc.).

## Aperçu

La création et la distribution d'une place de marché impliquent :

1. **Créer des plugins** : créez un ou plusieurs plugins avec des compétences, des agents, des hooks, des serveurs MCP ou des serveurs LSP.
2. **Créer un fichier de place de marché** : définissez un `marketplace.json` qui répertorie vos plugins et où les trouver.
3. **Héberger la place de marché** : poussez vers GitHub, GitLab ou un autre hôte git.
4. **Partager avec les utilisateurs** : les utilisateurs ajoutent votre place de marché avec `/plugin marketplace add`.

## Procédure pas à pas : créer une place de marché locale

```bash
# 1. Créer la structure de répertoires
mkdir -p my-marketplace/.claude-plugin
mkdir -p my-marketplace/plugins/quality-review-plugin/.claude-plugin
mkdir -p my-marketplace/plugins/quality-review-plugin/skills/quality-review
```

```markdown
# my-marketplace/plugins/quality-review-plugin/skills/quality-review/SKILL.md
---
description: Review code for bugs, security, and performance
disable-model-invocation: true
---

Review the code I've selected or the recent changes for:
- Potential bugs or edge cases
- Security concerns
- Performance issues
- Readability improvements

Be concise and actionable.
```

```json
// my-marketplace/plugins/quality-review-plugin/.claude-plugin/plugin.json
{
  "name": "quality-review-plugin",
  "description": "Adds a quality-review skill for quick code reviews",
  "version": "1.0.0"
}
```

```json
// my-marketplace/.claude-plugin/marketplace.json
{
  "name": "my-plugins",
  "owner": {
    "name": "Your Name"
  },
  "plugins": [
    {
      "name": "quality-review-plugin",
      "source": "./plugins/quality-review-plugin",
      "description": "Adds a quality-review skill for quick code reviews"
    }
  ]
}
```

```shell
# Ajouter et installer
/plugin marketplace add ./my-marketplace
/plugin install quality-review-plugin@my-plugins

# Essayer (les skills sont espacées avec le nom du plugin)
/quality-review-plugin:quality-review
```

## Créer le fichier de place de marché

Créez `.claude-plugin/marketplace.json` à la racine de votre dépôt.

```json
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team",
    "email": "devtools@example.com"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "./plugins/formatter",
      "description": "Automatic code formatting on save",
      "version": "2.1.0",
      "author": {
        "name": "DevTools Team"
      }
    },
    {
      "name": "deployment-tools",
      "source": {
        "source": "github",
        "repo": "company/deploy-plugin"
      },
      "description": "Deployment automation tools"
    }
  ]
}
```

## Schéma de la place de marché

### Champs obligatoires

| Champ     | Type   | Description                                                                                                                         | Exemple         |
| :-------- | :----- | :---------------------------------------------------------------------------------------------------------------------------------- | :-------------- |
| `name`    | string | Identifiant de la place de marché (kebab-case, sans espaces). Chaque utilisateur ne peut enregistrer qu'une seule place de marché par nom | `"acme-tools"`  |
| `owner`   | object | Informations du responsable                                                                                                         |                 |
| `plugins` | array  | Liste des plugins disponibles                                                                                                       |                 |

**Noms réservés** : `claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `anthropic-agent-skills`, `knowledge-work-plugins`, `life-sciences`, `claude-for-legal`, `claude-for-financial-services`, `financial-services-plugins`.

### Champs du propriétaire

| Champ   | Type   | Obligatoire | Description          |
| :------ | :----- | :---------- | :------------------- |
| `name`  | string | Oui         | Nom du responsable   |
| `email` | string | Non         | Email de contact     |

### Champs optionnels de la place de marché

| Champ                                 | Type   | Description                                                                                                         |
| :------------------------------------ | :----- | :------------------------------------------------------------------------------------------------------------------ |
| `$schema`                             | string | URL du schéma JSON pour l'autocomplétion                                                                            |
| `description`                         | string | Brève description de la place de marché                                                                             |
| `version`                             | string | Version du manifeste                                                                                                |
| `metadata.pluginRoot`                 | string | Répertoire de base ajouté aux chemins de source relatifs                                                            |
| `allowCrossMarketplaceDependenciesOn` | array  | Autres places de marché sur lesquelles les plugins peuvent dépendre                                                  |

## Entrées de plugin

### Champs obligatoires des entrées de plugin

| Champ    | Type           | Description                               |
| :------- | :------------- | :---------------------------------------- |
| `name`   | string         | Identifiant du plugin (kebab-case)        |
| `source` | string\|object | Où récupérer le plugin                    |

### Champs de plugin optionnels

| Champ            | Type    | Description                                                                                              |
| :--------------- | :------ | :------------------------------------------------------------------------------------------------------- |
| `displayName`    | string  | Nom lisible affiché dans l'UI. Nécessite v2.1.143+                                                       |
| `description`    | string  | Brève description du plugin                                                                              |
| `version`        | string  | Version épinglée. Omettez pour utiliser le SHA du commit git                                             |
| `author`         | object  | Informations sur l'auteur                                                                                |
| `homepage`       | string  | URL de documentation                                                                                     |
| `repository`     | string  | URL du code source                                                                                       |
| `license`        | string  | Identifiant de licence SPDX                                                                              |
| `keywords`       | array   | Balises pour la découverte                                                                               |
| `category`       | string  | Catégorie du plugin                                                                                      |
| `tags`           | array   | Balises pour la recherche                                                                                |
| `strict`         | boolean | Si `plugin.json` est l'autorité (par défaut true). Voir Mode strict                                      |
| `defaultEnabled` | boolean | Si le plugin est activé après installation (par défaut true). Nécessite v2.1.154+                        |

## Sources de plugin

| Source         | Type                                   | Champs                             | Notes                                                                                      |
| -------------- | -------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------ |
| Chemin relatif | `string` (ex: `"./my-plugin"`)         | —                                  | Répertoire local dans le dépôt. Doit commencer par `./`                                    |
| `github`       | object                                 | `repo`, `ref?`, `sha?`             |                                                                                            |
| `url`          | object                                 | `url`, `ref?`, `sha?`              | Source d'URL Git                                                                           |
| `git-subdir`   | object                                 | `url`, `path`, `ref?`, `sha?`      | Sous-répertoire dans un dépôt git. Clone partiel pour minimiser la bande passante          |
| `npm`          | object                                 | `package`, `version?`, `registry?` | Installé via `npm install`                                                                 |

### Chemins relatifs

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin"
}
```

Les chemins relatifs ne fonctionnent que quand les utilisateurs ajoutent la place de marché via Git. Pour la distribution basée sur les URL, utilisez GitHub, npm ou URL git.

### Dépôts GitHub

```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo",
    "ref": "v2.0.0",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

### Dépôts Git

```json
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git",
    "ref": "main"
  }
}
```

### Sous-répertoires Git

```json
{
  "name": "my-plugin",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/acme-corp/monorepo.git",
    "path": "tools/claude-plugin",
    "ref": "v2.0.0"
  }
}
```

### Paquets npm

```json
{
  "name": "my-npm-plugin",
  "source": {
    "source": "npm",
    "package": "@acme/claude-plugin",
    "version": "2.1.0",
    "registry": "https://npm.example.com"
  }
}
```

### Mode strict

| Valeur              | Comportement                                                                                                                                            |
| :------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `true` (par défaut) | `plugin.json` est l'autorité. L'entrée de la place de marché peut la compléter avec des composants supplémentaires                                       |
| `false`             | L'entrée de la place de marché est la définition complète. Si le plugin a un `plugin.json` qui déclare des composants, c'est un conflit (plugin non chargé) |

## Héberger et distribuer les places de marché

### Héberger sur GitHub (recommandé)

1. Créer un dépôt
2. Ajouter `.claude-plugin/marketplace.json`
3. Partager avec `/plugin marketplace add owner/repo`

### Dépôts privés

Pour les mises à jour automatiques en arrière-plan, définissez le jeton approprié :

| Fournisseur | Variables d'environnement    | Notes                                       |
| :---------- | :--------------------------- | :------------------------------------------ |
| GitHub      | `GITHUB_TOKEN` ou `GH_TOKEN` | Jeton d'accès personnel ou jeton GitHub App |
| GitLab      | `GITLAB_TOKEN` ou `GL_TOKEN` | Jeton d'accès personnel ou jeton de projet  |
| Bitbucket   | `BITBUCKET_TOKEN`            | Mot de passe d'application                  |

### Exiger des places de marché pour votre équipe

`.claude/settings.json` :

```json
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "code-formatter@company-tools": true
  }
}
```

### Pré-remplir les plugins pour les conteneurs

```bash
# Construire le répertoire de seed
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed claude plugin marketplace add your-org/plugins
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed claude plugin install my-tool@your-plugins
```

Définissez `CLAUDE_CODE_PLUGIN_SEED_DIR=/opt/claude-seed` dans l'environnement d'exécution.

**Comportement du seed** :
- Lecture seule : jamais écrit
- Les mises à jour automatiques sont désactivées pour les places de marché de seed
- Les entrées de seed ont la priorité sur la configuration utilisateur
- Les opérations de suppression/mise à jour échouent sur les seeds (contacter l'administrateur)

### Restrictions des places de marché gérées

Le paramètre `strictKnownMarketplaces` dans les paramètres gérés contrôle les sources autorisées :

| Valeur                  | Comportement                                                        |
| ----------------------- | ------------------------------------------------------------------- |
| Non défini (par défaut) | Aucune restriction                                                  |
| Tableau vide `[]`       | Verrouillage complet — aucune nouvelle place de marché              |
| Liste de sources        | Seules les places de marché correspondant à la liste sont autorisées |

Configurations :

```json
// Désactiver tous les ajouts
{ "strictKnownMarketplaces": [] }

// Autoriser uniquement des sources spécifiques
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/approved-plugins" },
    { "source": "url", "url": "https://plugins.example.com/marketplace.json" }
  ]
}

// GitHub Enterprise Server (hostPattern)
{
  "strictKnownMarketplaces": [
    { "source": "hostPattern", "hostPattern": "^github\\.example\\.com$" }
  ]
}

// Chemin local approuvé
{
  "strictKnownMarketplaces": [
    { "source": "pathPattern", "pathPattern": "^/opt/approved/" }
  ]
}
```

`strictKnownMarketplaces` restreint l'ajout mais n'enregistre pas. Associez avec `extraKnownMarketplaces` pour rendre les places de marché autorisées disponibles automatiquement.

### Résolution des versions et canaux de publication

Claude Code résout la version à partir du premier élément défini :

1. `version` dans le `plugin.json` du plugin
2. `version` dans l'entrée de la place de marché
3. SHA du commit git de la source du plugin

Pour les canaux de publication stable/latest, configurez deux places de marché pointant vers différentes refs :

```json
// stable-tools/marketplace.json
{
  "name": "stable-tools",
  "plugins": [{
    "name": "code-formatter",
    "source": { "source": "github", "repo": "acme-corp/code-formatter", "ref": "stable" }
  }]
}

// latest-tools/marketplace.json
{
  "name": "latest-tools",
  "plugins": [{
    "name": "code-formatter",
    "source": { "source": "github", "repo": "acme-corp/code-formatter", "ref": "latest" }
  }]
}
```

## Validation et test

```bash
# Valider la syntaxe
claude plugin validate .
# ou dans Claude Code :
/plugin validate .

# Ajouter pour test
/plugin marketplace add ./path/to/marketplace

# Installer un plugin de test
/plugin install test-plugin@marketplace-name
```

## Gérer les places de marché depuis la CLI

### Plugin marketplace add

```bash
claude plugin marketplace add <source> [options]
```

**Options :**

| Option                | Description                                                     | Par défaut |
| :-------------------- | :-------------------------------------------------------------- | :--------- |
| `--scope <scope>`     | `user`, `project` ou `local`                                    | `user`     |
| `--sparse <paths...>` | Limiter le checkout à des répertoires spécifiques (monodépôts)  |            |

```bash
# GitHub raccourci
claude plugin marketplace add acme-corp/claude-plugins

# Épingler à une branche
claude plugin marketplace add acme-corp/claude-plugins@v2.0

# URL git non-GitHub
claude plugin marketplace add https://gitlab.example.com/team/plugins.git

# URL directe vers marketplace.json
claude plugin marketplace add https://example.com/marketplace.json

# Local
claude plugin marketplace add ./my-marketplace

# Portée projet
claude plugin marketplace add acme-corp/claude-plugins --scope project

# Monodépôt sparse
claude plugin marketplace add acme-corp/monorepo --sparse .claude-plugin plugins
```

### Plugin marketplace list

```bash
claude plugin marketplace list [--json]
```

### Plugin marketplace remove

```bash
claude plugin marketplace remove <name> [--scope <scope>]
```

La suppression de la dernière portée restante désinstalle aussi tous les plugins installés à partir de cette place de marché.

### Plugin marketplace update

```bash
claude plugin marketplace update [name]
```

## Dépannage

| Problème                                          | Cause                                                              | Solution                                                                                   |
| :------------------------------------------------ | :----------------------------------------------------------------- | :----------------------------------------------------------------------------------------- |
| La place de marché ne se charge pas               | `.claude-plugin/marketplace.json` manquant                         | Créez le fichier avec les champs obligatoires                                              |
| `Duplicate plugin name "x" found in marketplace`  | Deux plugins partagent le même nom                                 | Donnez à chaque plugin une valeur `name` unique                                            |
| `plugins[0].source: Path contains ".."`           | Traversée de répertoire interdite                                  | Utilisez des chemins sans `..`                                                             |
| Les plugins avec chemins relatifs échouent en URL | Seul `marketplace.json` est téléchargé, pas les plugins            | Utilisez des sources GitHub/npm/URL git                                                    |
| Mises à jour échouent hors ligne                  | `git pull` efface le cache obsolète                               | Définissez `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1`                              |
| Opérations git expirent                           | Grand dépôt ou connexion lente                                     | Augmentez `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` (ex: `300000` pour 5 minutes)               |
| Authentification dépôt privé échoue               | Credentials manquants pour les mises à jour automatiques           | Définissez `GITHUB_TOKEN`, `GITLAB_TOKEN` ou `BITBUCKET_TOKEN`                             |

## Voir aussi

* [Découvrir et installer des plugins](/fr/discover-plugins)
* [Plugins](/fr/plugins) - Création de plugins
* [Référence des plugins](/fr/plugins-reference) - Spécifications techniques complètes
* [Paramètres des plugins](/fr/settings#plugin-settings)
* [Référence strictKnownMarketplaces](/fr/settings#strictknownmarketplaces)
