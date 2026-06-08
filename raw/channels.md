# Canaux (Channels)

Source: https://code.claude.com/docs/fr/channels

Note : Aperçu de recherche. Nécessite v2.1.80+. Authentification claude.ai ou Console API. PAS disponible sur Bedrock, Vertex, Foundry.

## Ce que c'est

Un canal = serveur MCP qui envoie des événements dans votre session Claude Code active.
Claude réagit aux messages arrivant pendant que vous êtes absent du terminal.
Bidirectionnel : Claude peut répondre via le même canal.

Nécessite Bun installé.

## Canaux supportés

- **Telegram** : bot via BotFather
- **Discord** : bot Discord
- **iMessage** : macOS uniquement, lecture de la base Messages

## Démarrage rapide (fakechat)

```
/plugin install fakechat@claude-plugins-official
```

Puis redémarrer avec :
```bash
claude --channels plugin:fakechat@claude-plugins-official
```

Ouvrir http://localhost:8787 pour une interface de chat localhost.

## Configuration Telegram (exemple)

1. Créer un bot via BotFather → copier le jeton
2. `/plugin install telegram@claude-plugins-official`
3. `/reload-plugins`
4. `/telegram:configure <token>`
5. Redémarrer : `claude --channels plugin:telegram@claude-plugins-official`
6. Envoyer un message au bot → code d'appairage
7. `/telegram:access pair <code>`
8. `/telegram:access policy allowlist`

## Sécurité

- Liste blanche d'expéditeurs : seuls les identifiants appairés peuvent envoyer
- Telegram/Discord : appairage via code
- iMessage : auto-chat (vous vous envoyer un message) contourne le contrôle d'accès
- La liste blanche contrôle aussi les relais de permission

## Contrôles d'entreprise

| Plan | Statut par défaut |
|------|-----------------|
| claude.ai Team/Enterprise | Bloqué jusqu'à activation par l'admin |
| Console API | Autorisé par défaut |

Activer via : claude.ai/admin-settings/claude-code → Channels toggle, ou `channelsEnabled: true` dans les paramètres gérés.

Restreindre les plugins via `allowedChannelPlugins` dans les paramètres gérés.

## Comparaison avec les autres approches

| Approche | Claude s'exécute | Bonne pour |
|---------|-----------------|-----------|
| Canaux | Votre machine (session active) | Réagir aux événements dans une session déjà ouverte |
| Claude Code sur le web | VM Anthropic | Déléguer du travail asynchrone |
| Claude dans Slack | VM Anthropic | Démarrer des tâches depuis le chat d'équipe |
| Remote Control | Votre machine | Piloter une session en cours depuis un autre appareil |

## Utilisation en production

Exécuter Claude dans un terminal persistant (tmux, screen) ou processus d'arrière-plan pour que le canal reste actif.

Les événements arrivent UNIQUEMENT quand la session est ouverte.
