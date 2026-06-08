# Zéro conservation des données (ZDR)

Source: https://code.claude.com/docs/fr/zero-data-retention

## Ce que c'est

ZDR = les invites et réponses du modèle générées lors des sessions Claude Code ne sont pas conservées par Anthropic après le retour de la réponse.

Disponible uniquement pour Claude Code sur Claude for Enterprise.

## Portée de ZDR

**Ce que ZDR couvre** :
- Appels d'inférence du modèle via Claude Code sur Claude for Enterprise (toutes les surfaces)

**Ce que ZDR NE couvre PAS** :
- Chat sur claude.ai (interface web Enterprise)
- Sessions Cowork
- Claude Code Analytics (collecte des métadonnées, pas les invites/réponses)
- Données administratives (emails, attributions de sièges)
- Données traitées par des outils tiers, serveurs MCP, intégrations externes

## Fonctionnalités désactivées sous ZDR

| Fonctionnalité | Raison |
|---------------|--------|
| Claude Code sur le web | Nécessite stockage côté serveur de l'historique |
| Sessions distantes (Desktop) | Nécessite données de session persistantes |
| Soumission de commentaires (`/feedback`) | Envoie données de conversation à Anthropic |

Bloquées au niveau du backend quel que soit l'affichage côté client.

## Analytics avec ZDR

Tableau de bord Analytics disponible, mais les métriques de contribution GitHub ne sont PAS disponibles.
Seules les métriques d'utilisation s'affichent.

## Conservation pour violations de politique

Même avec ZDR, Anthropic peut conserver les données jusqu'à 2 ans si la loi l'exige ou pour résoudre des violations de politique.

## ZDR par organisation

⚠️ ZDR est activée par organisation. Chaque nouvelle organisation doit être activée séparément.

## Demander ZDR

Contacter les ventes Anthropic ou votre équipe de compte.
ZDR ne s'applique qu'à la plateforme directe Anthropic (pas Bedrock, Vertex, Foundry).

## Fournisseurs cloud

Pour Bedrock, Vertex AI, Foundry → consulter les politiques de conservation des données de ces plateformes.
