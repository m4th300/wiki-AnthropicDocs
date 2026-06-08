# Modes de permission

Source: https://code.claude.com/docs/fr/permission-modes

## Les 6 modes

| Mode | Ce qui s'exécute sans demander | Idéal pour |
|------|-------------------------------|------------|
| `default` | Lectures uniquement | Démarrage, travaux sensibles |
| `acceptEdits` | Lectures + éditions fichiers + commandes FS courantes (mkdir, touch, mv, cp, sed) | Itération sur code |
| `plan` | Lectures uniquement | Explorer avant de modifier |
| `auto` | Tout, avec vérifications sécurité en arrière-plan | Tâches longues, moins de fatigue |
| `dontAsk` | Uniquement outils pré-approuvés | CI verrouillé |
| `bypassPermissions` | Tout | Conteneurs/VMs isolés UNIQUEMENT |

Chemins protégés : jamais auto-approuvés sauf en `bypassPermissions` (.git, .config/git, .vscode, .claude, .bashrc, .gitconfig, etc.)

## Changer de mode

**CLI** : `Shift+Tab` pour parcourir default → acceptEdits → plan (+ auto et bypass si activés)
**Au démarrage** : `claude --permission-mode plan`
**Par défaut** : `settings.json` → `permissions.defaultMode: "acceptEdits"`

## Mode acceptEdits

Auto-approuve : éditions de fichiers dans le répertoire de travail + commandes Bash courantes (mkdir, touch, rm, rmdir, mv, cp, sed) + leurs équivalents PowerShell.
Les chemins hors scope et chemins protégés invitent toujours.

## Mode plan

Claude lit et explore mais ne modifie pas. Après le plan :
- Approuver en mode auto
- Approuver et accepter les modifications
- Approuver et examiner manuellement
- Continuer la planification avec des commentaires
- `Ctrl+G` → ouvrir le plan dans l'éditeur

## Mode auto (aperçu de recherche)

Classificateur séparé examine chaque action en arrière-plan. Nécessite :
- Claude Code v2.1.83+
- Tous les plans (Pro, Max, Team, Enterprise)
- Sur Team/Enterprise : admin doit l'activer dans l'admin console
- Modèle : Sonnet 4.6 ou Opus 4.6+
- Sur Bedrock/Vertex/Foundry : définir `CLAUDE_CODE_ENABLE_AUTO_MODE=1`, Opus 4.7+ uniquement

### Ce que le classificateur bloque par défaut
- `curl | bash` (download + execute)
- Envoi données sensibles vers serveurs externes
- Déploiements et migrations de production
- Suppression en masse sur cloud storage
- Octroi permissions IAM
- Push direct vers main

### Ce qui est autorisé par défaut
- Opérations fichiers locaux dans répertoire de travail
- Installation dépendances des manifestes
- Lecture `.env` + envoi credentials à l'API correspondante
- Requêtes HTTP en lecture seule
- Push vers la branche sur laquelle vous avez commencé

Retour en arrière si 3 blocages consécutifs ou 20 au total.

## Mode dontAsk

Refuse automatiquement tout outil qui invoquerait sinon. Uniquement les règles `allow` et commandes Bash en lecture seule peuvent s'exécuter.

```bash
claude --permission-mode dontAsk
```

## Mode bypassPermissions

Désactive toutes les vérifications. À partir de v2.1.126 : inclut les chemins protégés.
Exception : `rm -rf /` et `rm -rf ~` invitent toujours.

**Uniquement pour environnements isolés : conteneurs, VMs, devcontainers sans accès Internet.**

```bash
claude --permission-mode bypassPermissions
# ou
claude --dangerously-skip-permissions
```

Ne fonctionne pas en tant que root/sudo sur Linux/macOS.

## Chemins protégés (jamais auto-approuvés sauf bypassPermissions)

Répertoires : `.git`, `.config/git`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.yarn`, `.mvn`, `.claude` (sauf commands/agents/skills/worktrees)
Fichiers : `.gitconfig`, `.bashrc`, `.zshrc`, `.npmrc`, `.yarnrc`, `.mcp.json`, etc.
