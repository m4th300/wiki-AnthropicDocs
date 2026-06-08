# Explorez le répertoire .claude

> Où Claude Code lit CLAUDE.md, settings.json, hooks, skills, commands, subagents, workflows, rules et auto memory. Explorez le répertoire .claude dans votre projet et ~/.claude dans votre répertoire personnel.

Claude Code lit les instructions, les paramètres, les skills, les subagents et la mémoire à partir de votre répertoire de projet et de `~/.claude` dans votre répertoire personnel. Validez les fichiers du projet dans git pour les partager avec votre équipe ; les fichiers dans `~/.claude` sont une configuration personnelle qui s'applique à tous vos projets.

Sur Windows, `~/.claude` se résout en `%USERPROFILE%\.claude`. Si vous définissez `CLAUDE_CONFIG_DIR`, chaque chemin `~/.claude` sur cette page se trouve sous ce répertoire à la place.

La plupart des utilisateurs ne modifient que `CLAUDE.md` et `settings.json`. Le reste du répertoire est optionnel : ajoutez des skills, des rules ou des subagents selon vos besoins.

---

## Fichiers au niveau du projet

### CLAUDE.md

- **Badge :** committed
- **Résumé :** Instructions chargées à chaque session
- **Quand :** Chargé dans le contexte au début de chaque session
- **Description :** Instructions spécifiques au projet qui déterminent comment Claude travaille dans ce dépôt. Mettez vos conventions, commandes courantes et contexte architectural ici afin que Claude opère avec les mêmes hypothèses que votre équipe.
- **Conseils :**
  - Ciblez moins de 200 lignes. Les fichiers plus longs se chargent quand même en entier mais peuvent réduire l'adhérence
  - CLAUDE.md se charge dans chaque session. Si quelque chose ne concerne que des tâches spécifiques, déplacez-le dans un skill ou une règle limitée par chemin
  - Listez les commandes que vous exécutez le plus souvent, comme build, test et format
  - Exécutez `/memory` pour ouvrir et modifier CLAUDE.md depuis une session
  - Fonctionne également à `.claude/CLAUDE.md` si vous préférez garder la racine du projet propre
- **Référence :** `/en/memory`

**Exemple :**
```markdown
# Project conventions

## Commands
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`

## Stack
- TypeScript with strict mode
- React 19, functional components only

## Rules
- Named exports, never default exports
- Tests live next to source: `foo.ts` -> `foo.test.ts`
- All API routes return `{ data, error }` shape
```

---

### .mcp.json

- **Badge :** committed
- **Résumé :** Serveurs MCP limités au projet, partagés avec votre équipe
- **Quand :** Les serveurs se connectent quand la session commence. Les schémas d'outils sont différés par défaut et se chargent à la demande via la recherche d'outils
- **Description :** Configure les serveurs Model Context Protocol (MCP) qui donnent à Claude accès à des outils externes : bases de données, APIs, navigateurs, et plus. Ce fichier contient les serveurs limités au projet que toute votre équipe utilise. Les serveurs personnels vont dans `~/.claude.json` à la place.
- **Conseils :**
  - Utilisez des références de variables d'environnement pour les secrets : `${GITHUB_TOKEN}`
  - Se trouve à la racine du projet, pas dans `.claude/`
  - Pour les serveurs dont vous seul avez besoin, exécutez `claude mcp add --scope user`
- **Référence :** `/en/mcp`

**Exemple :**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

---

### .worktreeinclude

- **Badge :** committed
- **Résumé :** Fichiers gitignorés à copier dans les nouveaux worktrees
- **Quand :** Lu quand Claude crée un worktree git via `--worktree`, l'outil `EnterWorktree`, ou isolation `worktree` de sous-agent
- **Description :** Liste les fichiers gitignorés à copier de votre dépôt principal dans chaque nouveau worktree. Les worktrees sont de nouveaux checkouts, donc les fichiers non suivis comme `.env` sont manquants par défaut. Les motifs utilisent la syntaxe `.gitignore`.
- **Se trouve à :** La racine du projet, pas dans `.claude/`
- **Référence :** `/en/worktrees#copy-gitignored-files-into-worktrees`

**Exemple :**
```
# Local environment
.env
.env.local

# API credentials
config/secrets.json
```

---

## Répertoire .claude/ (niveau projet)

Tout ce que Claude Code lit spécifique à ce projet. Si vous utilisez git, validez la plupart des fichiers ici pour que votre équipe les partage ; quelques-uns, comme `settings.local.json`, sont automatiquement gitignorés.

### .claude/settings.json

- **Badge :** committed
- **Résumé :** Permissions, hooks et configuration
- **Quand :** Remplace le global `~/.claude/settings.json`. Les paramètres locaux, les flags CLI et les paramètres gérés ont priorité sur ceci
- **Description :** Paramètres que Claude Code applique directement. Les permissions contrôlent quels outils et commandes Claude peut utiliser ; les hooks exécutent vos scripts à des points spécifiques d'une session.
- **Clés communes :**
  - `permissions` : autoriser, refuser ou demander avant que Claude utilise des outils ou commandes spécifiques
  - `hooks` : exécuter vos propres scripts sur des événements comme avant un appel d'outil ou après une édition de fichier
  - `statusLine` : personnaliser la ligne affichée en bas pendant que Claude travaille
  - `model` : choisir un modèle par défaut pour ce projet
  - `env` : variables d'environnement définies dans chaque session
  - `outputStyle` : sélectionner un style de prompt système personnalisé depuis output-styles/
- **Conseils :**
  - Les motifs de permission Bash supportent les wildcards : `Bash(npm test *)` correspond à toute commande commençant par `npm test`
  - Les paramètres de tableau comme `permissions.allow` se combinent à travers toutes les portées ; les paramètres scalaires comme `model` utilisent la valeur la plus spécifique
- **Référence :** `/en/settings`

**Exemple :**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm test *)",
      "Bash(npm run *)"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ]
  },
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
      }]
    }]
  }
}
```

---

### .claude/settings.local.json

- **Badge :** gitignored
- **Résumé :** Vos remplacements de paramètres personnels pour ce projet
- **Quand :** Le plus haut des fichiers de paramètres modifiables par l'utilisateur ; les flags CLI et les paramètres gérés ont toujours la priorité
- **Description :** Paramètres personnels qui ont la priorité sur les valeurs par défaut du projet. Même format JSON que `settings.json`, mais non validé.
- **Conseils :**
  - Même schéma que `settings.json`. Les paramètres de tableau comme `permissions.allow` se combinent à travers les portées
  - Claude Code ajoute ce fichier à `~/.config/git/ignore` la première fois qu'il en écrit un
- **Référence :** `/en/settings`

**Exemple :**
```json
{
  "permissions": {
    "allow": [
      "Bash(docker *)"
    ]
  }
}
```

---

### .claude/rules/

- **Badge :** committed
- **Résumé :** Instructions limitées à un sujet, optionnellement limitées par chemins
- **Quand :** Les règles sans `paths:` se chargent au début de la session. Les règles avec `paths:` se chargent quand un fichier correspondant entre dans le contexte
- **Description :** Instructions de projet divisées en fichiers thématiques qui peuvent se charger conditionnellement selon les chemins de fichiers. Comme CLAUDE.md, les règles sont des conseils que Claude lit, pas une configuration que Claude Code impose.
- **Conseils :**
  - Utilisez le frontmatter `paths:` avec des globs pour limiter les règles à des répertoires ou types de fichiers
  - Les sous-répertoires fonctionnent : `.claude/rules/frontend/react.md` est découvert automatiquement
  - Quand CLAUDE.md approche 200 lignes, commencez à diviser en règles
- **Référence :** `/en/memory#organize-rules-with-claude/rules/`

**Exemples :**

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
---

# Testing Rules

- Use descriptive test names: "should [expected] when [condition]"
- Mock external dependencies, not internal modules
- Clean up side effects in afterEach
```

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Design Rules

- All endpoints must validate input with Zod schemas
- Return shape: { data: T } | { error: string }
- Rate limit all public endpoints
```

---

### .claude/skills/

- **Badge :** committed
- **Résumé :** Invites réutilisables que vous ou Claude invoquez par nom
- **Quand :** Invoqué avec `/skill-name` ou quand Claude associe la tâche à un skill
- **Description :** Chaque skill est un dossier avec un fichier SKILL.md plus les fichiers de support nécessaires. Par défaut, vous et Claude pouvez invoquer un skill. Utilisez le frontmatter pour contrôler cela : `disable-model-invocation: true` pour les workflows utilisateur uniquement comme `/deploy`, ou `user-invocable: false` pour masquer du menu `/` pendant que Claude peut encore l'invoquer.
- **Conseils :**
  - Les skills acceptent des arguments : `/deploy staging` passe "staging" comme `$ARGUMENTS`. Utilisez `$0`, `$1`, etc. pour l'accès positionnel
  - Le frontmatter `description` détermine quand Claude auto-invoque le skill
  - Regroupez les docs de référence avec SKILL.md. Claude connaît le chemin du répertoire du skill
- **Référence :** `/en/skills`

**Exemple (SKILL.md) :**
```markdown
---
description: Reviews code changes for security vulnerabilities, authentication gaps, and injection risks
disable-model-invocation: true
argument-hint: <branch-or-path>
---

## Diff to review

!`git diff $ARGUMENTS`

Audit the changes above for:

1. Injection vulnerabilities (SQL, XSS, command)
2. Authentication and authorization gaps
3. Hardcoded secrets or credentials

Use checklist.md in this skill directory for the full review checklist.

Report findings with severity ratings and remediation steps.
```

---

### .claude/commands/

- **Badge :** committed
- **Résumé :** Invites sur un seul fichier invoquées avec `/name`
- **Quand :** L'utilisateur tape `/command-name`
- **Description :** Un fichier à `.claude/commands/deploy.md` crée `/deploy` de la même façon qu'un skill à `skills/deploy/SKILL.md`. Les skills utilisent un répertoire avec SKILL.md, permettant de regrouper des docs de référence, modèles ou scripts.
- **Note :** Commands et skills sont maintenant le même mécanisme. Pour les nouveaux workflows, utilisez `skills/` à la place.
- **Conseils :**
  - Utilisez `$ARGUMENTS` dans le fichier pour accepter des paramètres : `/fix-issue 123`
  - Si un skill et une commande partagent un nom, le skill a la priorité
  - Les nouvelles commandes devraient généralement être des skills à la place
- **Référence :** `/en/skills`

**Exemple :**
```markdown
---
argument-hint: <issue-number>
---

!`gh issue view $ARGUMENTS`

Investigate and fix the issue above.

1. Trace the bug to its root cause
2. Implement the fix
3. Write or update tests
4. Summarize what you changed and why
```

---

### .claude/output-styles/

- **Badge :** committed
- **Résumé :** Styles de sortie limités au projet, si votre équipe en partage
- **Quand :** Appliqué au début de la session quand sélectionné via le paramètre `outputStyle`
- **Description :** Les styles de sortie sont généralement personnels, donc la plupart vivent dans `~/.claude/output-styles/`. Mettez-en un ici si votre équipe partage un style.
- **Référence :** `/en/output-styles`

---

### .claude/agents/

- **Badge :** committed
- **Résumé :** Sous-agents spécialisés avec leur propre fenêtre de contexte
- **Quand :** S'exécute dans sa propre fenêtre de contexte quand vous ou Claude l'invoquez
- **Description :** Chaque fichier markdown définit un sous-agent avec son propre prompt système, accès aux outils, et optionnellement son propre modèle. Les sous-agents s'exécutent dans une fenêtre de contexte fraîche, gardant la conversation principale propre.
- **Conseils :**
  - Chaque agent obtient une fenêtre de contexte fraîche, séparée de votre session principale
  - Restreignez l'accès aux outils par agent avec le champ frontmatter `tools:`
  - Tapez @ et choisissez un agent depuis l'autocomplétion pour déléguer directement
- **Référence :** `/en/sub-agents`

**Exemple :**
```markdown
---
name: code-reviewer
description: Reviews code for correctness, security, and maintainability
tools: Read, Grep, Glob
---

You are a senior code reviewer. Review for:

1. Correctness: logic errors, edge cases, null handling
2. Security: injection, auth bypass, data exposure
3. Maintainability: naming, complexity, duplication

Every finding must include a concrete fix.
```

---

### .claude/workflows/

- **Badge :** committed
- **Résumé :** Scripts de flux de travail dynamiques qui orchestrent de nombreux sous-agents
- **Quand :** Chargé au démarrage ; chaque fichier devient une commande `/<name>`
- **Description :** Chaque fichier `.js` est un workflow dynamique : un script que le runtime exécute pour créer et coordonner de nombreux sous-agents. Les workflows sont écrits par Claude et sauvegardés ici depuis `/workflows`.
- **Conseils :**
  - Sauvegardez une exécution depuis `/workflows` avec `s` pour en créer un
  - Un workflow de projet a la priorité sur un workflow personnel dans `~/.claude/workflows/` avec le même nom
- **Référence :** `/en/workflows`

---

### .claude/agent-memory/

- **Badge :** committed, Claude writes
- **Résumé :** Mémoire persistante des sous-agents, séparée de votre auto memory de session principale
- **Quand :** Les 200 premières lignes (limitées à 25KB) de MEMORY.md chargées dans le prompt système du sous-agent quand il s'exécute
- **Description :** Les sous-agents avec `memory: project` dans leur frontmatter obtiennent un répertoire de mémoire dédié ici. Distinct de votre auto memory de session principale à `~/.claude/projects/`.
- **Conseils :**
  - Créé uniquement pour les sous-agents qui définissent le champ frontmatter `memory:`
  - Pour la mémoire hors contrôle de version, utilisez `memory: local`, qui écrit dans `.claude/agent-memory-local/`
  - Pour la mémoire inter-projets, utilisez `memory: user`, qui écrit dans `~/.claude/agent-memory/`
- **Référence :** `/en/sub-agents#enable-persistent-memory`

---

## Fichiers globaux (~/.claude/)

Les fichiers ici s'appliquent à chaque projet dans lequel vous travaillez et ne sont jamais validés dans un dépôt.

### ~/.claude.json

- **Badge :** local only
- **Résumé :** État de l'application et préférences d'interface utilisateur
- **Quand :** Lu au début de la session pour vos préférences et serveurs MCP. Claude Code écrit dedans quand vous changez les paramètres dans `/config` ou approuvez les invites de confiance
- **Description :** Contient l'état qui n'appartient pas à `settings.json` : thème, session OAuth, décisions de confiance par projet, vos serveurs MCP personnels et bascules d'interface utilisateur.
- **Conseils :**
  - Les bascules IDE comme `autoConnectIde` et `externalEditorContext` vivent ici, pas dans `settings.json`
  - La clé `projects` suit l'état par projet comme l'acceptation du dialogue de confiance
  - Les serveurs MCP ici sont les vôtres uniquement : la portée utilisateur s'applique à tous les projets, la portée locale est par projet mais non validée
- **Référence :** `/en/settings#global-config-settings`

**Exemple :**
```json
{
  "autoConnectIde": true,
  "externalEditorContext": true,
  "mcpServers": {
    "my-tools": {
      "command": "npx",
      "args": ["-y", "@example/mcp-server"]
    }
  }
}
```

---

### ~/.claude/CLAUDE.md

- **Badge :** local only
- **Résumé :** Préférences personnelles dans chaque projet
- **Quand :** Chargé au début de chaque session, dans chaque projet
- **Description :** Votre fichier d'instructions global. Chargé aux côtés du CLAUDE.md du projet au début de la session, donc les deux sont dans le contexte ensemble. Quand les instructions entrent en conflit, les instructions au niveau du projet ont la priorité.
- **Conseils :**
  - Gardez-le court car il se charge dans le contexte pour chaque projet
  - Bon pour le style de réponse, le format de commit et les conventions personnelles
- **Référence :** `/en/memory`

**Exemple :**
```markdown
# Global preferences

- Keep explanations concise
- Use conventional commit format
- Show the terminal command to verify changes
- Prefer composition over inheritance
```

---

### ~/.claude/settings.json

- **Badge :** local only
- **Résumé :** Paramètres par défaut pour tous les projets
- **Quand :** Vos valeurs par défaut. Les `settings.json` de projet et locaux remplacent tous les clés que vous définissez également là
- **Description :** Mêmes clés que le `settings.json` du projet : permissions, hooks, modèle, variables d'environnement, etc. Mettez ici les paramètres que vous voulez dans chaque projet.
- **Référence :** `/en/settings`

**Exemple :**
```json
{
  "permissions": {
    "allow": [
      "Bash(git log *)",
      "Bash(git diff *)"
    ]
  }
}
```

---

### ~/.claude/keybindings.json

- **Badge :** local only
- **Résumé :** Raccourcis clavier personnalisés
- **Quand :** Lu au début de la session et rechargé à chaud quand vous éditez le fichier
- **Description :** Rebindez les raccourcis clavier dans le CLI interactif. Exécutez `/keybindings` pour créer ou ouvrir ce fichier. Ctrl+C, Ctrl+D, Ctrl+M et Caps Lock sont réservés et ne peuvent pas être rebindés.
- **Référence :** `/en/keybindings`

**Exemple :**
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

---

### ~/.claude/themes/

- **Badge :** local only
- **Résumé :** Thèmes de couleurs personnalisés
- **Quand :** Lu au début de la session et rechargé à chaud quand les fichiers changent. Listé dans `/theme`
- **Description :** Chaque fichier `.json` définit un thème de couleur personnalisé : un preset `base` intégré plus une carte `overrides` de tokens de couleur.
- **Référence :** `/en/terminal-config#create-a-custom-theme`

**Exemple :**
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

### ~/.claude/projects/

- **Badge :** local only, Claude writes
- **Résumé :** Auto memory : notes de Claude à lui-même, par projet
- **Quand :** MEMORY.md chargé au début de la session ; les fichiers thématiques lus à la demande
- **Description :** L'auto memory permet à Claude d'accumuler des connaissances entre les sessions sans que vous n'écriviez quoi que ce soit. Claude sauvegarde des notes pendant qu'il travaille : commandes de build, insights de débogage, notes d'architecture. Chaque projet obtient son propre répertoire de mémoire indexé par le chemin du dépôt.
- **Conseils :**
  - Activé par défaut. Basculer avec `/memory` ou `autoMemoryEnabled` dans les paramètres
  - MEMORY.md est l'index chargé à chaque session. Les 200 premières lignes, ou 25KB, sont lues
  - Les fichiers thématiques comme `debugging.md` sont lus à la demande, pas au démarrage
  - Ce sont des markdowns simples. Éditez ou supprimez-les à tout moment
- **Référence :** `/en/memory#auto-memory`

**Structure :**
```
~/.claude/projects/<project>/memory/
  MEMORY.md          ← index chargé à chaque session
  debugging.md       ← fichier thématique lu à la demande
  architecture.md    ← fichier thématique lu à la demande
```

---

### ~/.claude/rules/

- **Badge :** local only
- **Résumé :** Règles au niveau utilisateur qui s'appliquent à chaque projet
- **Quand :** Les règles sans `paths:` se chargent au début de la session. Les règles avec `paths:` se chargent quand un fichier correspondant entre dans le contexte
- **Description :** Même structure que le `.claude/rules/` du projet mais s'applique partout. Utilisez ceci pour les conventions que vous voulez dans tout votre travail.
- **Référence :** `/en/memory#organize-rules-with-claude/rules/`

---

### ~/.claude/skills/

- **Badge :** local only
- **Résumé :** Skills personnels disponibles dans chaque projet
- **Quand :** Invoqué avec `/skill-name` dans n'importe quel projet
- **Description :** Skills que vous avez construits pour vous-même qui fonctionnent partout. Même structure que les skills du projet.
- **Référence :** `/en/skills`

---

### ~/.claude/output-styles/

- **Badge :** local only
- **Résumé :** Sections de prompt système personnalisées qui ajustent comment Claude fonctionne
- **Quand :** Appliqué au début de la session quand sélectionné via le paramètre `outputStyle`
- **Description :** Chaque fichier markdown définit un style de sortie : une section ajoutée au prompt système qui, par défaut, supprime également les instructions intégrées de tâches d'ingénierie logicielle.
- **Conseils :**
  - Les styles intégrés Explanatory et Learning sont inclus avec Claude Code
  - Définissez `keep-coding-instructions: true` dans le frontmatter pour garder les instructions de tâche par défaut
  - Les changements prennent effet à la session suivante car le prompt système est fixé au démarrage
- **Référence :** `/en/output-styles`

**Exemple :**
```markdown
---
description: Explains reasoning and asks you to implement small pieces
keep-coding-instructions: true
---

After completing each task, add a brief "Why this approach" note
explaining the key design decision.

When a change is under 10 lines, ask the user to implement it
themselves by leaving a TODO(human) marker instead of writing it.
```

---

### ~/.claude/agents/

- **Badge :** local only
- **Résumé :** Sous-agents personnels disponibles dans chaque projet
- **Quand :** Claude délègue ou vous @-mentionnez dans n'importe quel projet
- **Description :** Les sous-agents définis ici sont disponibles dans tous vos projets. Même format que les agents du projet.
- **Référence :** `/en/sub-agents`

---

### ~/.claude/workflows/

- **Badge :** local only
- **Résumé :** Workflows dynamiques personnels disponibles dans chaque projet
- **Quand :** Chargé au démarrage ; chaque fichier devient une commande `/<name>`
- **Description :** Les scripts de workflow sauvegardés ici sont disponibles dans tous vos projets. Un workflow de projet avec le même nom dans `.claude/workflows/` a la priorité.
- **Référence :** `/en/workflows`

---

### ~/.claude/agent-memory/

- **Badge :** local only, Claude writes
- **Résumé :** Mémoire persistante pour les sous-agents avec `memory: user`
- **Quand :** Chargé dans le prompt système du sous-agent quand il démarre
- **Description :** Les sous-agents avec `memory: user` dans leur frontmatter stockent ici des connaissances qui persistent à travers tous les projets.
- **Référence :** `/en/sub-agents#enable-persistent-memory`

---

## Ce qui n'est pas affiché

| Fichier | Emplacement | Objectif |
| ----------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------- |
| `managed-settings.json` | Au niveau du système, varie selon le système d'exploitation | Paramètres appliqués par l'entreprise que vous ne pouvez pas remplacer |
| `CLAUDE.local.md` | Racine du projet | Vos préférences privées pour ce projet, chargées aux côtés de CLAUDE.md |
| Plugins installés | `~/.claude/plugins` | Marchés clonés, versions de plugins installées et données par plugin |

`~/.claude` contient également les données que Claude Code écrit au fur et à mesure que vous travaillez : transcriptions, historique des invites, instantanés de fichiers, caches et journaux.

---

## Choisissez le bon fichier

| Vous voulez | Modifier | Portée | Référence |
| :---------------------------------------------------------------- | :--------------------------------------- | :---------------- | :---------------------------------------------------- |
| Donner à Claude le contexte et les conventions du projet | `CLAUDE.md` | projet ou global | Mémoire |
| Autoriser ou bloquer des appels d'outils spécifiques | `settings.json` `permissions` ou `hooks` | projet ou global | Permissions, Hooks |
| Exécuter un script avant ou après les appels d'outils | `settings.json` `hooks` | projet ou global | Hooks |
| Définir les variables d'environnement pour la session | `settings.json` `env` | projet ou global | Paramètres |
| Garder les remplacements personnels hors de git | `settings.local.json` | projet uniquement | Portées des paramètres |
| Ajouter une invite ou une capacité que vous invoquez avec `/name` | `skills/<name>/SKILL.md` | projet ou global | Skills |
| Définir un subagent spécialisé avec ses propres outils | `agents/*.md` | projet ou global | Subagents |
| Orchestrer de nombreux subagents à partir d'un script | `workflows/*.js` | projet ou global | Workflows dynamiques |
| Connecter des outils externes via MCP | `.mcp.json` | projet uniquement | MCP |
| Modifier la façon dont Claude formate les réponses | `output-styles/*.md` | projet ou global | Styles de sortie |

---

## Référence des fichiers

| Fichier | Portée | Valider | Ce qu'il fait | Référence |
| --------------------------------------------------- | ----------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `CLAUDE.md` | Projet et global | ✓ | Instructions chargées à chaque session | Mémoire |
| `rules/*.md` | Projet et global | ✓ | Instructions limitées à un sujet, optionnellement limitées par chemin | Rules |
| `settings.json` | Projet et global | ✓ | Permissions, hooks, variables d'environnement, paramètres par défaut du modèle | Paramètres |
| `settings.local.json` | Projet uniquement | | Vos remplacements personnels, auto-gitignorés | Portées des paramètres |
| `.mcp.json` | Projet uniquement | ✓ | Serveurs MCP partagés par l'équipe | Portées MCP |
| `.worktreeinclude` | Projet uniquement | ✓ | Fichiers gitignorés à copier dans les nouveaux worktrees | Worktrees |
| `skills/<name>/SKILL.md` | Projet et global | ✓ | Invites réutilisables invoquées avec `/name` ou auto-invoquées | Skills |
| `commands/*.md` | Projet et global | ✓ | Invites sur un seul fichier ; même mécanisme que les skills | Skills |
| `output-styles/*.md` | Projet et global | ✓ | Sections de système-prompt personnalisées | Styles de sortie |
| `agents/*.md` | Projet et global | ✓ | Définitions de subagents avec leur propre invite et outils | Subagents |
| `workflows/*.js` | Projet et global | ✓ | Scripts de flux de travail dynamiques ; chaque fichier devient une commande `/<name>` | Flux de travail dynamiques |
| `agent-memory/<name>/` | Projet et global | ✓ | Mémoire persistante pour les subagents | Mémoire persistante |
| `~/.claude.json` | Global uniquement | | État de l'application, OAuth, bascules d'interface utilisateur, serveurs MCP personnels | Configuration globale |
| `projects/<project>/memory/` | Global uniquement | | Auto memory : notes de Claude à lui-même entre les sessions | Auto memory |
| `keybindings.json` | Global uniquement | | Raccourcis clavier personnalisés | Keybindings |
| `themes/*.json` | Global uniquement | | Thèmes de couleurs personnalisés | Thèmes personnalisés |

---

## Données d'application

Au-delà de la configuration que vous créez, `~/.claude` contient les données que Claude Code écrit pendant les sessions.

### Nettoyés automatiquement

Les fichiers dans les chemins ci-dessous sont supprimés au démarrage une fois qu'ils sont plus anciens que `cleanupPeriodDays` (défaut : 30 jours).

| Chemin sous `~/.claude/` | Contenu |
| -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `projects/<project>/<session>.jsonl` | Transcription complète de la conversation |
| `projects/<project>/<session>/subagents/` | Transcriptions de conversation des sous-agents |
| `projects/<project>/<session>/tool-results/` | Les grandes sorties d'outils |
| `file-history/<session>/` | Instantanés pré-édition des fichiers |
| `plans/` | Fichiers de plan du mode plan |
| `debug/` | Journaux de débogage par session |
| `paste-cache/`, `image-cache/` | Contenu des grands collages et images jointes |
| `session-env/` | Métadonnées d'environnement par session |
| `tasks/` | Listes de tâches par session |
| `shell-snapshots/` | Environnement shell capturé |
| `backups/` | Copies de `~/.claude.json` avant migrations |
| `feedback-bundles/` | Archives de transcription expurgées |

### Conservés jusqu'à ce que vous les supprimiez

| Chemin sous `~/.claude/` | Contenu |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `history.jsonl` | Chaque invite que vous avez tapée, avec horodatage et chemin du projet |
| `stats-cache.json` | Nombres de tokens et de coûts agrégés affichés par `/usage` |
| `remote-settings.json` | Copie en cache des paramètres gérés par le serveur |

### Effacer les données locales

Exécutez `claude project purge` pour supprimer l'état que Claude Code maintient pour un projet (nécessite Claude Code v2.1.124+).

```bash
# Prévisualiser sans supprimer
claude project purge ~/work/my-repo --dry-run

# Supprimer avec confirmation
claude project purge ~/work/my-repo

# Supprimer sans confirmation
claude project purge ~/work/my-repo --yes

# Purger tous les projets
claude project purge --all
```

Ne supprimez pas `~/.claude.json`, `~/.claude/settings.json` ou `~/.claude/plugins/` : ceux-ci contiennent votre authentification, vos préférences et vos plugins installés.

---

## Ressources connexes

* Gérez la mémoire de Claude : `/fr/memory`
* Configurez les paramètres : `/fr/settings`
* Créez des skills : `/fr/skills`
* Configurez les subagents : `/fr/sub-agents`
* Déboguez votre configuration : `/fr/debug-your-config`
