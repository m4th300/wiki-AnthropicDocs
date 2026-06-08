# Exécuter Claude Code par programmation (mode non-interactif)

Source: https://code.claude.com/docs/fr/headless

Note : À partir du 15 juin 2026, l'utilisation de `claude -p` sur plans abonnement tirera depuis un nouveau crédit mensuel Agent SDK séparé.

## Mode basique

```bash
claude -p "Find and fix the bug in auth.py" --allowedTools "Read,Edit,Bash"
```

Flag `-p` ou `--print` = mode non-interactif.
Toutes les options CLI fonctionnent avec `-p`.

## Mode bare (recommandé pour CI)

```bash
claude --bare -p "Summarize this file" --allowedTools "Read"
```

`--bare` ignore : hooks, skills, plugins, MCP, auto memory, CLAUDE.md.
Résultat identique sur chaque machine indépendamment de la configuration locale.

Contexte en mode bare :
| Pour charger | Utiliser |
|-------------|---------|
| Additions prompt système | `--append-system-prompt`, `--append-system-prompt-file` |
| Paramètres | `--settings <file-or-json>` |
| Serveurs MCP | `--mcp-config <file-or-json>` |
| Agents personnalisés | `--agents <json>` |
| Plugin | `--plugin-dir <path>`, `--plugin-url <url>` |

Note : mode bare ignore OAuth et le trousseau → utiliser `ANTHROPIC_API_KEY` ou `apiKeyHelper` dans `--settings`.

`--bare` sera le mode par défaut pour `-p` dans une version future.

## Piper des données

```bash
cat build-error.txt | claude -p 'concisely explain the root cause' > output.txt
```

Limite stdin : 10 Mo (v2.1.128+). Au-delà → erreur avec statut non nul.

## Ajouter Claude dans un script de compilation

```json
{
  "scripts": {
    "lint:claude": "git diff main | claude -p \"you are a typo linter. for each typo, report filename:line and the issue.\""
  }
}
```

## Formats de sortie

| Format | Utilisation |
|--------|------------|
| `text` (défaut) | Sortie texte brut |
| `json` | JSON structuré avec résultat, ID session, métadonnées |
| `stream-json` | JSON délimité par sauts de ligne pour streaming |

```bash
claude -p "Summarize this project" --output-format json | jq -r '.result'
```

## Sortie structurée avec schéma JSON

```bash
claude -p "Extract the main function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
  | jq '.structured_output'
```

## Réponses en streaming

```bash
claude -p "Write a poem" --output-format stream-json --verbose --include-partial-messages | \
  jq -rj 'select(.type == "stream_event" and .event.delta.type? == "text_delta") | .event.delta.text'
```

Événements spéciaux :
- `system/api_retry` : avant chaque réessai API
- `system/init` : métadonnées de session (modèle, outils, MCP, plugins chargés)
- `system/plugin_install` : progression de l'installation des plugins

## Approuver automatiquement les outils

```bash
claude -p "Run the test suite and fix any failures" \
  --allowedTools "Bash,Read,Edit"

# Mode de permission pour CI verrouillé
claude -p "Apply the lint fixes" --permission-mode acceptEdits
```

## Créer un commit

```bash
claude -p "Look at my staged changes and create an appropriate commit" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"
```

## Personnaliser le prompt système

```bash
gh pr diff "$1" | claude -p \
  --append-system-prompt "You are a security engineer. Review for vulnerabilities." \
  --output-format json
```

## Continuer les conversations

```bash
# Première requête
claude -p "Review this codebase for performance issues"

# Continuer la plus récente
claude -p "Now focus on the database queries" --continue

# Continuer une session spécifique
session_id=$(claude -p "Start a review" --output-format json | jq -r '.session_id')
claude -p "Continue that review" --resume "$session_id"
```
