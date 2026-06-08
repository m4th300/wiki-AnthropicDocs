# Contrôle à distance (Remote Control)

Source: https://code.claude.com/docs/fr/remote-control

Note : Aperçu de recherche. Sur Team/Enterprise : désactivé par défaut, admin doit activer.

## Ce que c'est

Connecte claude.ai/code ou l'application Claude mobile à une session Claude Code s'exécutant sur votre machine locale. Claude continue à s'exécuter localement à tout moment.

Différent de Claude Code sur le web : code s'exécute sur votre machine (pas dans le cloud).

## Prérequis

- Abonnement Pro, Max, Team, Enterprise (pas de clé API)
- Claude Code v2.1.51+
- Sur Team/Enterprise : admin doit activer le bouton Remote Control dans l'admin console

## Démarrer une session Remote Control

### Mode serveur (recommandé pour plusieurs sessions)

```bash
claude remote-control
```

Reste en cours d'exécution, attend des connexions. Barre d'espace = afficher code QR.

Options :
- `--name "My Project"` : titre personnalisé
- `--spawn same-dir|worktree|session` : isolation des sessions
- `--capacity N` : max sessions concurrentes (défaut: 32)
- `--verbose` : journaux de connexion
- `--sandbox / --no-sandbox` : activer/désactiver sandboxing

### Session interactive avec Remote Control

```bash
claude --remote-control           # = --rc
claude --remote-control "Mon projet"
```

Session interactive normale + accès distant depuis claude.ai ou mobile.

### Depuis une session existante

```text
/remote-control
/rc Mon projet
```

### Extension VS Code

Taper `/remote-control` ou `/rc` dans la zone de saisie.
Bannière avec état de connexion + bouton "Open in browser".

## Se connecter depuis un autre appareil

- Ouvrir l'URL de session dans un navigateur
- Scanner le code QR (app Claude mobile)
- Ouvrir [claude.ai/code](https://claude.ai/code) → trouver la session par nom

Titre de session (ordre de priorité) :
1. Nom passé à `--name` ou `/remote-control`
2. Titre défini avec `/rename`
3. Dernier message significatif dans l'historique
4. Nom auto-généré (ex: `myhost-graceful-unicorn`)

## Activer pour toutes les sessions

`/config` → "Enable Remote Control for all sessions" → `true`
Desktop : Settings → Claude Code → Enable remote control by default.

## Sécurité

- Uniquement requêtes HTTPS **sortantes** (pas de ports entrants ouverts)
- S'enregistre auprès de l'API Anthropic, interroge le travail
- Trafic via API Anthropic sur TLS
- Identifiants de courte durée, chacun limité à un seul objectif

## Remote Control vs Claude Code sur le web

| | Remote Control | Claude Code sur le web |
|--|---------------|----------------------|
| Claude s'exécute sur | Votre machine | VM Anthropic |
| Accès aux MCP locaux | Oui | Non |
| Accès aux fichiers locaux | Oui | Non, GitHub seulement |
| Continue si machine fermée | Non | Oui |

## Notifications push mobiles

Claude peut envoyer des notifications push sur votre téléphone (v2.1.110+).
- Installer app Claude mobile
- Se connecter avec le même compte
- Autoriser les notifications
- `/config` → "Push when Claude decides"

## Commandes disponibles depuis mobile/web

**Fonctionnent** : `/compact`, `/clear`, `/context`, `/usage`, `/exit`, `/recap`, `/reload-plugins`
**Locales uniquement** : `/mcp`, `/plugin`, `/resume` (sélecteurs interactifs)

## Limitations

- Une session distante par processus interactif (hors mode serveur)
- Si terminal/VS Code fermé → session se termine
- Panne réseau > ~10 min → session expire
- Ultraplan déconnecte Remote Control (incompatibles en même temps)

## Dépannage

**"requires a claude.ai subscription"** : `ANTHROPIC_API_KEY` présente → `unset ANTHROPIC_API_KEY`
**"requires a full-scope login token"** : utilise `CLAUDE_CODE_OAUTH_TOKEN` → `claude auth login`
**"is disabled by your organization's policy"** :
- Authentifié avec clé API ou Console → `/login` avec compte claude.ai
- Admin Team/Enterprise n'a pas activé → admin console
- `disableRemoteControl` dans paramètres gérés

## Tableau comparatif des approches de travail à distance

| | Dispatch | Remote Control | Channels | Slack | Tâches planifiées |
|--|---------|----------------|---------|-------|------------------|
| Déclencheur | App mobile | Depuis n'importe quel appareil | Événements externes | @Claude | Calendrier |
| Claude s'exécute | Desktop | Votre machine | Votre machine (CLI) | Cloud Anthropic | Selon type |
