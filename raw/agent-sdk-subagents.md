# Sous-agents dans le SDK Agent

> Définissez et invoquez des sous-agents pour isoler le contexte, exécuter des tâches en parallèle et appliquer des instructions spécialisées

## Aperçu

Trois façons de créer des sous-agents:
1. **Par programmation** : paramètre `agents` dans `query()` (recommandé pour SDK)
2. **Fichiers système** : markdown dans `.claude/agents/` 
3. **Général intégré** : Claude peut invoquer `general-purpose` automatiquement via l'outil Agent

## Avantages

- **Isolation du contexte** : appels d'outils intermédiaires restent dans le sous-agent
- **Parallélisation** : plusieurs sous-agents simultanément
- **Instructions spécialisées** : prompts système adaptés avec expertise spécifique
- **Restrictions d'outils** : limiter les outils disponibles par sous-agent

## Définition programmatique

```python
async for message in query(
    prompt="Use the code-reviewer agent to review this codebase",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Glob", "Grep", "Agent"],  # "Agent" requis pour sous-agents
        agents={
            "code-reviewer": AgentDefinition(
                description="Expert code reviewer for quality and security reviews.",
                prompt="Analyze code quality and suggest improvements.",
                tools=["Read", "Glob", "Grep"],
                model="sonnet",
            )
        },
    ),
):
    if hasattr(message, "result"):
        print(message.result)
```

## Configuration AgentDefinition

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `description` | `string` | Oui | Quand utiliser cet agent |
| `prompt` | `string` | Oui | Prompt système de l'agent |
| `tools` | `string[]` | Non | Outils autorisés (hérite si omis) |
| `disallowedTools` | `string[]` | Non | Outils interdits |
| `model` | `string` | Non | Surcharge du modèle (alias: `sonnet`, `opus`, `haiku`, `inherit`) |
| `skills` | `string[]` | Non | Skills à précharger au démarrage |
| `memory` | `string` | Non | Source de mémoire |
| `mcpServers` | `(string|object)[]` | Non | Serveurs MCP disponibles |
| `maxTurns` | `number` | Non | Tours maximum |
| `background` | `boolean` | Non | Tâche de fond non bloquante |
| `effort` | `string|number` | Non | Niveau d'effort |
| `permissionMode` | `PermissionMode` | Non | Mode de permission |

Note: Les sous-agents **ne peuvent pas** créer leurs propres sous-agents. Ne pas inclure `Agent` dans `tools` d'un sous-agent.

## Ce que les sous-agents héritent

| Le sous-agent reçoit | Le sous-agent ne reçoit pas |
|---------------------|----------------------------|
| Son propre prompt système + invite de l'outil Agent | Historique de conversation du parent |
| CLAUDE.md du projet | Contenu des skills non listés dans `skills` |
| Définitions d'outils (héritées ou subset) | Prompt système du parent |

## Invocation

### Automatique
Claude décide selon la tâche et la `description` du sous-agent.

### Explicite
```text
"Use the code-reviewer agent to check the authentication module"
```

### Dynamique (factory pattern)
```python
def create_security_agent(security_level: str) -> AgentDefinition:
    is_strict = security_level == "strict"
    return AgentDefinition(
        description="Security code reviewer",
        prompt=f"You are a {'strict' if is_strict else 'balanced'} security reviewer...",
        tools=["Read", "Grep", "Glob"],
        model="opus" if is_strict else "sonnet",
    )

options = ClaudeAgentOptions(
    allowed_tools=["Read", "Grep", "Glob", "Agent"],
    agents={"security-reviewer": create_security_agent("strict")},
)
```

## Détection d'invocation

Les sous-agents sont invoqués via l'outil `Agent` (ancien nom: `Task`).

```python
for block in message.content:
    if isinstance(block, ToolUseBlock) and block.name in ("Task", "Agent"):
        print(f"Subagent invoked: {block.input.get('subagent_type')}")
```

Les messages depuis le contexte d'un sous-agent incluent `parent_tool_use_id`.

## Reprise des sous-agents

Possible en capturant le `session_id` et l'`agentId`, puis en reprenant la session avec `resume: sessionId` et en incluant l'ID d'agent dans l'invite.

## Combinaisons d'outils courantes

| Cas d'usage | Outils |
|-------------|--------|
| Analyse lecture seule | `Read`, `Grep`, `Glob` |
| Exécution de tests | `Bash`, `Read`, `Grep` |
| Modification de code | `Read`, `Edit`, `Write`, `Grep`, `Glob` |
| Accès complet | Hérite de tous les outils (omettre `tools`) |

## Workflows dynamiques

Pour coordonner des dizaines à centaines d'agents, utiliser l'outil `Workflow` (TypeScript SDK v0.3.149+).
