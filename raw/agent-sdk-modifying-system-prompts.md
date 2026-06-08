# Modification des invites système dans le SDK Agent

> Choisissez entre le préréglage `claude_code` et une invite système personnalisée.

## Points de départ

1. **Défaut minimal** : invite minimaliste, appel d'outils basique sans directives Claude Code
2. **Préréglage `claude_code`** : invite complète de Claude Code (outils, style, formatage, sécurité, contexte)
3. **Chaîne personnalisée** : invite entièrement personnalisée

## Décider d'un point de départ

| Cas | Utiliser |
|-----|---------|
| Outil CLI/IDE avec humain qui observe | Préréglage `claude_code` |
| Même + règles spécifiques (normes, format) | Préréglage + `append` |
| Agent avec surface/identité/permission différentes | Chaîne personnalisée |
| Boucle d'appel d'outils mince | Pas d'option `systemPrompt` |

"Différent de Claude Code" = surface non-terminal, identité propre, modèle de permission autonome, ou tâches non-codage.

## Méthodes de personnalisation

### 1. CLAUDE.md (instructions niveau projet)

Injecté dans la conversation (pas dans le prompt système). Fonctionne avec n'importe quelle config de prompt.

Chargé quand `"project"` (CLAUDE.md du cwd) ou `"user"` (~/.claude/CLAUDE.md) est dans `settingSources`.

```typescript
options: {
  systemPrompt: { type: "preset", preset: "claude_code" },
  settingSources: ["project"]
}
```

### 2. Styles de sortie (configurations persistantes)

Fichiers markdown avec frontmatter dans `~/.claude/output-styles/` ou `.claude/output-styles/`.

```markdown
---
name: Code Reviewer
description: Thorough code review assistant
keep-coding-instructions: true
---

You are an expert code reviewer...
```

- `keep-coding-instructions: true` : conserver les instructions Claude Code + ajouter les vôtres
- `keep-coding-instructions: false` (défaut) : remplacer les instructions de codage

### 3. Append au préréglage

```python
options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": "Always include detailed docstrings and type hints in Python code.",
    }
)
```

### Améliorer la mise en cache avec `exclude_dynamic_sections`

```python
system_prompt={
    "type": "preset",
    "preset": "claude_code",
    "append": "You operate Acme's internal triage workflow.",
    "exclude_dynamic_sections": True,  # Rend le prompt système identique entre sessions
}
```

Note: Le contexte par session (cwd, os, shell) se déplace dans le premier message utilisateur plutôt que dans le prompt système. Légèrement moins autoritaire pour ces détails.

Nécessite SDK Python v0.1.58+ ou TypeScript v0.2.98+.

### 4. Invite personnalisée (contrôle complet)

```python
options = ClaudeAgentOptions(
    system_prompt="You are a Python coding specialist. Follow PEP 8 style guidelines."
)
```

Vous êtes responsable de toute la guidance d'outils et des règles de sécurité.

## Comparaison

| | CLAUDE.md | Styles de sortie | Append | Personnalisé |
|--|-----------|-----------------|--------|--------------|
| Persistance | Par projet | Fichiers | Session | Session |
| Outils par défaut | Préservés | Préservés | Préservés | Perdus sauf si inclus |
| Sécurité intégrée | Maintenue | Maintenue | Maintenue | À ajouter |
| Niveau personnalisation | Ajouts | Remplace/étend | Ajouts | Contrôle complet |

## Combiner les approches

```python
options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": "For this review, prioritize OAuth 2.0 compliance.",
    },
    setting_sources=["project"],  # Charge aussi CLAUDE.md
)
```
