# Équipes d'agents — Orchestrer des équipes de sessions Claude Code

> Coordonnez plusieurs instances Claude Code travaillant ensemble en tant qu'équipe.

Warning: Les équipes d'agents sont expérimentales et désactivées par défaut. Activer via `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` dans `settings.json` ou l'environnement. Nécessite Claude Code v2.1.32+.

## Quand utiliser les équipes d'agents

**Cas d'usage idéaux:**
- Recherche et examen (plusieurs hypothèses en parallèle)
- Nouveaux modules ou fonctionnalités (chaque coéquipier possède une partie)
- Débogage avec hypothèses concurrentes (tester différentes théories simultanément)
- Coordination inter-couches (frontend + backend + tests chacun par un coéquipier)

**Quand éviter:**
- Tâches séquentielles
- Modifications du même fichier
- Travail avec beaucoup de dépendances
- Tâches de routine simples (une seule session ou sous-agents suffisent)

## Comparer avec les sous-agents

| | Sous-agents | Équipes d'agents |
|--|------------|----------------|
| Contexte | Propre, résultats reviennent au parent | Propre, complètement indépendant |
| Communication | Parent seulement | Coéquipiers communiquent directement |
| Coordination | Parent gère tout | Liste de tâches partagée, auto-coordination |
| Meilleur pour | Tâches focalisées | Travail complexe nécessitant discussion |
| Coût en tokens | Inférieur (résumés) | Supérieur (chaque coéquipier = instance Claude) |

## Activer les équipes d'agents

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Démarrer une équipe

Demander à Claude de créer une équipe en langage naturel:
```text
Créez une équipe d'agents pour explorer cela sous différents angles :
un coéquipier sur l'UX, un sur l'architecture technique, un jouant l'avocat du diable.
```

## Contrôler l'équipe

### Modes d'affichage

| Mode | Description |
|------|-------------|
| `in-process` | Coéquipiers dans votre terminal principal. Maj+Bas pour naviguer. |
| `tmux` | Chaque coéquipier dans son propre volet divisé |
| `auto` (défaut) | Volets divisés si dans tmux, sinon in-process |

Configurer dans `~/.claude/settings.json`:
```json
{
  "teammateMode": "in-process"
}
```

### Naviguer entre coéquipiers (mode in-process)
- `Maj+Bas` : naviguer entre coéquipiers
- `Entrée` : voir session du coéquipier
- `Échap` : interrompre le tour actuel
- `Ctrl+T` : basculer la liste des tâches

### Spécifier coéquipiers et modèles
```text
Créez une équipe avec 4 coéquipiers pour refactoriser ces modules.
Utilisez Sonnet pour chaque coéquipier.
```

### Approbation du plan
```text
Générez un coéquipier architecte. Exigez l'approbation du plan avant modifications.
```

### Arrêter et nettoyer
```text
Demandez au coéquipier chercheur d'arrêter
Nettoyez l'équipe
```

Warning: Toujours utiliser le chef pour nettoyer. Les coéquipiers ne doivent pas exécuter le nettoyage.

## Architecture

| Composant | Rôle |
|-----------|------|
| Chef d'équipe | Crée l'équipe, génère coéquipiers, coordonne |
| Coéquipiers | Instances Claude Code distinctes |
| Liste de tâches | Partagée, états: en attente/en cours/terminées |
| Boîte aux lettres | Messagerie inter-agents |

Stockage:
- Configuration: `~/.claude/teams/{team-name}/config.json`
- Tâches: `~/.claude/tasks/{team-name}/`

Note: Ne pas éditer manuellement ces fichiers. Créés et mis à jour automatiquement.

## Meilleures pratiques

- **Contexte des coéquipiers** : inclure les détails spécifiques à la tâche dans le prompt de génération (pas d'historique de conversation du chef)
- **Taille d'équipe** : 3-5 coéquipiers optimaux. 5-6 tâches par coéquipier.
- **Taille des tâches** : unités autonomes produisant un livrable clair (une fonction, un fichier de test)
- **Éviter les conflits** : chaque coéquipier possède un ensemble de fichiers différent
- **Commencer par recherche/examen** avant d'essayer les équipes pour les implémentations

## Utiliser les définitions de sous-agents pour les coéquipiers

```text
Générez un coéquipier utilisant le type d'agent security-reviewer pour auditer le module auth.
```

Les outils de coordination d'équipe (`SendMessage`, gestion des tâches) sont toujours disponibles même si `tools` les restreint.

Note: Les champs `skills` et `mcpServers` d'une définition de sous-agent ne s'appliquent pas quand elle s'exécute comme coéquipier.

## Limitations connues

- Pas de reprise de session avec les coéquipiers in-process (`/resume` ne restaure pas les coéquipiers)
- L'état des tâches peut être en retard
- L'arrêt peut être lent
- Une équipe à la fois par chef
- Pas d'équipes imbriquées
- Le chef est fixe pour la durée de vie de l'équipe
- Les volets divisés nécessitent tmux ou iTerm2 (pas VS Code, Windows Terminal, Ghostty)
