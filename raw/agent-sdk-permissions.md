# Permissions dans le SDK Agent

> Contrôlez comment votre agent utilise les outils avec les modes de permission, les hooks et les règles déclaratives.

## Flux d'évaluation des permissions

1. **Hooks** : exécutent d'abord, peuvent refuser ou transmettre
2. **Règles de refus** : vérifier `disallowed_tools` et `settings.json`
3. **Mode de permission** : `bypassPermissions` approuve tout ce qui reste
4. **Règles d'autorisation** : vérifier `allowed_tools` et `settings.json`
5. **Callback `canUseTool`** : décision finale (si mode n'est pas `dontAsk`)

## Règles d'autorisation et de refus

| Option | Effet |
|--------|-------|
| `allowed_tools=["Read", "Grep"]` | Auto-approuvés. Les outils non listés passent au mode de permission. |
| `disallowed_tools=["Bash"]` | Supprime l'outil du contexte de Claude entièrement |
| `disallowed_tools=["Bash(rm *)"]` | Bash reste disponible, mais `rm *` est refusé dans tous les modes |

Agent verrouillé:
```python
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Glob", "Grep"],
    permission_mode="dontAsk",  # Tout hors de la liste est refusé
)
```

Warning: `allowed_tools` ne contraint PAS `bypassPermissions`. Les outils non listés sont quand même approuvés.

## Modes de permission

| Mode | Description | Comportement |
|------|-------------|--------------|
| `default` | Standard | Non couverts → callback `canUseTool` |
| `dontAsk` | Refuser au lieu de demander | Tout hors des règles d'autorisation est refusé |
| `acceptEdits` | Auto-accepter modifications fichiers | Fichiers + commandes fs (`mkdir`, `rm`, `mv`, etc.) |
| `bypassPermissions` | Contourner tout | TOUS les outils s'exécutent (utiliser en sandbox uniquement) |
| `plan` | Mode planification | Outils lecture seule; Claude analyse sans modifier |
| `auto` (TS) | IA classe les approbations | Classificateur modèle approuve/refuse chaque outil |

Warning héritage: `bypassPermissions`, `acceptEdits`, `auto` sont hérités par les sous-agents et ne peuvent pas être remplacés par sous-agent.

## Mode acceptEdits

Auto-approuve:
- Modifications de fichiers (Edit, Write)
- Commandes fs: `mkdir`, `touch`, `rm`, `rmdir`, `mv`, `cp`, `sed`
- Uniquement dans le répertoire de travail ou `additionalDirectories`

## Mode plan

Outils en lecture seule + `AskUserQuestion`. Claude explore et planifie sans modifier les fichiers source.

## Mode dontAsk

Convertit toute invite de permission en refus. Préférable à une dépendance silencieuse à l'absence de `canUseTool`.

## Mode bypassPermissions

Tous les outils autorisés s'exécutent sans invites. Les hooks peuvent encore bloquer. Ne peut pas être utilisé comme root sur Unix.

## Définir dynamiquement le mode

```python
# Pendant streaming
await client.set_permission_mode("acceptEdits")
```

```typescript
await q.setPermissionMode("acceptEdits");
```

## Configuration via settings.json

```json
{
  "permissions": {
    "allow": ["Read", "Bash(git status)"],
    "deny": ["Bash(rm -rf *)"]
  }
}
```

Chargé si `setting_sources` inclut `"project"` (défaut).
