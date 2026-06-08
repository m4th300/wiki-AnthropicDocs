# Flux de travail courants

Source: https://code.claude.com/docs/fr/common-workflows

## Recettes de prompts

### Explorer une nouvelle base de code
```text
give me an overview of this codebase
explain the main architecture patterns
what are the key data models?
how is authentication handled?
```

### Trouver du code
```text
find the files that handle user authentication
how do these authentication files work together?
trace the login process from front-end to database
```

### Corriger des bugs
```text
I'm seeing an error when I run npm test
suggest a few ways to fix the @ts-ignore in user.ts
update user.ts to add the null check you suggested
```

### Refactoriser
```text
find deprecated API usage in our codebase
suggest how to refactor utils.js to use modern JavaScript features
refactor utils.js to use ES2024 features while maintaining the same behavior
run tests for the refactored code
```

### Tests
```text
find functions in NotificationsService.swift that are not covered by tests
add tests for the notification service
add test cases for edge conditions in the notification service
run the new tests and fix any failures
```

### Créer une PR
```text
summarize the changes I've made to the authentication module
create a pr
enhance the PR description with more context
```

Reprendre une PR plus tard : `claude --from-pr <number>` ou coller l'URL dans `/resume`.

### Documentation
```text
find functions without proper JSDoc comments in the auth module
add JSDoc comments to the undocumented functions in auth.js
check if the documentation follows our project standards
```

### Images
- Glisser-déposer dans Claude Code
- Copier + Ctrl+V (pas Cmd+V)
- Fournir un chemin : "Analyze this image: /path/to/image.png"

### Références de fichiers avec @
- `@src/utils/auth.js` → contenu complet du fichier
- `@src/components/` → liste du répertoire
- `@github:repos/owner/repo/issues` → ressources MCP
- Les références @ chargent aussi les CLAUDE.md dans le répertoire du fichier

## Reprendre les conversations

```bash
claude --continue    # Dernière session du répertoire
claude --resume      # Choisir dans une liste
```

## Sessions parallèles avec worktrees

```bash
claude --worktree feature-auth
```

Chaque worktree = checkout séparé sur sa propre branche. Voir [Worktrees].

## Planifier avant de modifier

```bash
claude --permission-mode plan
# ou Shift+Tab pendant une session
```

## Déléguer la recherche à des subagents

```text
use a subagent to investigate how our auth system handles token refresh
```

Le subagent lit les fichiers dans son propre contexte, rapporte seulement les conclusions.

## Piping et scripts

```bash
git log --oneline -20 | claude -p "summarize these recent commits"
tail -200 app.log | claude -p "find any anomalies"
git diff main --name-only | claude -p "review these changed files for security issues"
```

## Planification des tâches

| Option | Où s'exécute | Idéal pour |
|--------|-------------|-----------|
| Routines | Infrastructure Anthropic | Tâches même si ordinateur éteint |
| Tâches planifiées Desktop | Votre machine | Accès fichiers locaux |
| GitHub Actions | Pipeline CI | Événements repo |
| `/loop` | Session CLI actuelle | Sondage rapide |

## Questions sur les capacités

```text
can Claude Code create pull requests?
how does Claude Code handle permissions?
what skills are available?
how do I use MCP with Claude Code?
```

Claude a accès à sa propre documentation à jour.
