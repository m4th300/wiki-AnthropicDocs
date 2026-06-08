# Gérer les coûts efficacement

Source: https://code.claude.com/docs/fr/costs

## Coûts moyens observés

Dans les déploiements entreprise :
- ~13 $/développeur/jour actif en moyenne
- 150-250 $/développeur/mois
- 90% des utilisateurs : < 30 $/jour actif

## Suivre vos coûts

### `/usage` dans la session
```text
Total cost:            $0.55
Total duration (API):  6m 19.7s
Total duration (wall): 6h 33m 10.2s
Total code changes:    0 lines added, 0 lines removed
```

Note : chiffre en $ = estimation locale à partir des token counts ≠ facture réelle.
Plans Pro/Max/Team/Enterprise : barres d'utilisation du plan + ventilation par skills/subagents/plugins.
`d` / `w` : basculer 24h / 7 jours.

Pour facturation fiable : Console Claude → page Utilisation.

## Gérer les coûts pour les équipes

### API Claude
- Définir [limites de dépenses workspace](https://platform.claude.com/docs/fr/build-with-claude/workspaces#workspace-limits) dans la Console
- Workspace "Claude Code" créé automatiquement à la première connexion

### Plans Pro/Max
- `/usage-credits` pour définir une limite de dépenses mensuelle en crédits

### Bedrock/Vertex/Foundry
- Utiliser LiteLLM pour le suivi des dépenses par clé (open-source, non affilié Anthropic)

## Recommandations de limites de débit

| Taille équipe | TPM par utilisateur | RPM par utilisateur |
|--------------|--------------------|--------------------|
| 1-5 | 200k-300k | 5-7 |
| 5-20 | 100k-150k | 2.5-3.5 |
| 20-50 | 50k-75k | 1.25-1.75 |
| 50-100 | 25k-35k | 0.62-0.87 |
| 100-500 | 15k-20k | 0.37-0.47 |
| 500+ | 10k-15k | 0.25-0.35 |

TPM par utilisateur diminue avec la taille (moindre concurrence dans les grandes orgas).

## Coûts équipes d'agents

~7x plus de tokens qu'une session standard (chaque coéquipier = sa propre context window).
- Utiliser Sonnet pour les coéquipiers
- Garder les équipes petites
- Nettoyer les équipes quand le travail est terminé

## Réduire l'utilisation des tokens

Claude Code optimise automatiquement via :
- **Mise en cache des invites** : réduit les coûts pour le contenu répété
- **Compaction automatique** : résume l'historique en approchant les limites

### Stratégies manuelles

**Gestion du contexte**
- `/clear` entre tâches non liées
- `/compact Focus on code samples and API usage`
- Dans CLAUDE.md : `# Compact instructions: When compacting, focus on test output and code changes`

**Choix du modèle**
- Sonnet pour la plupart des tâches de codage
- Opus pour les décisions architecturales complexes
- Haiku pour les tâches simples de subagent
- `/model` pour changer en cours de session

**Serveurs MCP**
- Schémas différés par défaut (MCP Tool Search)
- Préférer les outils CLI (`gh`, `aws`, `gcloud`) aux serveurs MCP quand disponibles
- Désactiver les serveurs inutilisés via `/mcp`

**Plugins d'intelligence de code**
- Donnent à Claude une navigation de symboles précise
- Un appel "aller à la définition" remplace grep + lecture de fichiers candidats
- Signalent automatiquement les erreurs de type après modifications

**Hooks de prétraitement**
Un hook qui filtre la sortie avant que Claude ne la voie peut réduire massivement les tokens :
```bash
# Filtrer la sortie des tests pour afficher uniquement les échecs
if [[ "$cmd" =~ ^(npm test|pytest|go test) ]]; then
  filtered_cmd="$cmd 2>&1 | grep -A 5 -E '(FAIL|ERROR|error:)' | head -100"
  # retourner la commande filtrée
fi
```

**CLAUDE.md vs Skills**
- CLAUDE.md : chargé à chaque session (tous les tokens)
- Skills : chargées à la demande (tokens uniquement quand utilisées)
Déplacer les instructions spécialisées (review PR, migration DB) dans des skills.
Garder CLAUDE.md < 200 lignes.

**Réflexion étendue**
- Activée par défaut → dizaines de milliers de tokens de réflexion par requête
- Pour tâches simples : baisser le niveau d'effort avec `/effort`
- Ou : `MAX_THINKING_TOKENS=8000`

**Subagents**
- Déléguer les opérations détaillées (tests, récupération docs, logs)
- Seul le résumé revient dans votre conversation principale

**Prompts spécifiques**
- "améliorer cette base de code" → analyse large, coûteux
- "ajouter la validation des entrées à la fonction de connexion dans auth.ts" → efficace

**Habitudes pour les tâches complexes**
- Plan Mode avant l'implémentation
- `Esc` + `/rewind` pour corriger la trajectoire tôt
- Fournir des cibles de vérification
- Tester de manière progressive (un fichier à la fois)

## Utilisation en arrière-plan

Claude Code utilise des tokens même inactif :
- Résumé des conversations : fonctionnalité `claude --resume`
- Traitement de certaines commandes comme `/usage`

Coût : généralement < 0,04 $/session.
