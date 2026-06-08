# Utilisation des données

Source: https://code.claude.com/docs/fr/data-usage

## Politique de formation aux données

**Utilisateurs grand public (Free, Pro, Max)** :
- Avec paramètre activé : données utilisées pour améliorer les modèles futurs
- Avec paramètre désactivé : conservation 30 jours, pas de formation

**Utilisateurs commerciaux (Team, Enterprise, API, Bedrock, Vertex, Foundry)** :
- Anthropic NE forme PAS de modèles avec votre code ou vos invites
- Sauf si vous avez explicitement opté via le Programme de partenariat pour développeurs

Modifier les préférences : [claude.ai/settings/data-privacy-controls](https://claude.ai/settings/data-privacy-controls)

## Conservation des données

| Type de compte | Avec formation activée | Sans formation |
|---------------|----------------------|----------------|
| Free/Pro/Max | 5 ans | 30 jours |
| Team/Enterprise/API | N/A | 30 jours |
| Enterprise ZDR | N/A | Zéro persistance |

**Conservation locale** : Transcriptions de session dans `~/.claude/projects/` pendant 30 jours (configurable avec `cleanupPeriodDays`).

## Sondages de qualité de session

"Comment Claude s'en sort-il ?" → enregistre uniquement votre note.

Suivi optionnel : "Anthropic peut-il consulter votre transcription ?" :
- **Oui** : télécharge la transcription, sous-agents et logs (conservés 6 mois)
- **Non** / **Ne plus demander** : rien n'est envoyé

Désactiver : `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY=1`

## Télémétrie et rapports d'erreurs

| Service | Claude API | Vertex/Bedrock/Foundry |
|---------|-----------|----------------------|
| Métriques (latence, fiabilité) | Activé | Désactivé |
| Sentry (erreurs) | Activé | Désactivé |
| Rapports `/feedback` | Activé | Désactivé |
| Sondages qualité | Activé | Activé |

Désactiver tout le trafic non essentiel : `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`
Désactiver télémétrie : `DISABLE_TELEMETRY`
Désactiver erreurs : `DISABLE_ERROR_REPORTING`
Désactiver feedback : `DISABLE_FEEDBACK_COMMAND=1`

## Vérification de sécurité WebFetch

Avant chaque fetch, le nom d'hôte est vérifié contre une liste de blocage Anthropic.
S'exécute quel que soit le fournisseur.
Désactiver : `skipWebFetchPreflight: true` dans les paramètres.

## Chiffrement

- **En transit** : TLS 1.2+
- **Au repos** :
  - API Anthropic : AES-256 + Zero Data Retention possible
  - Bedrock : AES-256 avec clés AWS KMS (CMEK disponible)
  - Vertex AI : clés Google + CMEK disponible
  - Foundry : acheminé vers infra Anthropic avec AES-256

## Exécution cloud (Claude Code sur le web)

- Code cloné sur VM isolée Anthropic
- Identifiants GitHub via proxy sécurisé (jamais dans le sandbox)
- Trafic sortant via proxy de sécurité pour audit
- Suppression d'une session = suppression définitive des données d'événement
