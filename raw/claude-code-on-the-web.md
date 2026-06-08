> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt

# Utiliser Claude Code sur le web

> Configurez les environnements cloud, les scripts de configuration, l'accès réseau et Docker dans le sandbox d'Anthropic. Déplacez les sessions entre le web et le terminal avec `--remote` et `--teleport`.

**Note:** Claude Code sur le web est en aperçu de recherche pour les utilisateurs Pro, Max et Team, ainsi que pour les utilisateurs Enterprise disposant de sièges premium ou de sièges Chat + Claude Code.

Claude Code sur le web exécute les tâches sur l'infrastructure cloud gérée par Anthropic à [claude.ai/code](https://claude.ai/code). Les sessions persistent même si vous fermez votre navigateur, et vous pouvez les surveiller depuis l'application mobile Claude.

## Options d'authentification GitHub

| Méthode | Comment ça marche | Idéal pour |
| :------ | :---------------- | :--------- |
| **Application GitHub** | Autorisez l'application Claude GitHub lors de l'intégration web. | Intégration web ; équipes qui veulent Auto-fix |
| **`/web-setup`** | Exécutez `/web-setup` dans votre terminal pour synchroniser votre jeton CLI `gh` local vers votre compte Claude. | Développeurs individuels qui utilisent déjà `gh` |

Les administrateurs Team et Enterprise peuvent désactiver `/web-setup` avec le bouton bascule Quick web setup sur [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code).

Les organisations avec Zéro rétention de données activée ne peuvent pas utiliser `/web-setup`.

## L'environnement cloud

Chaque session s'exécute dans une VM fraîche gérée par Anthropic avec votre référentiel cloné.

### Ce qui est disponible dans les sessions cloud

| | Disponible dans les sessions cloud | Pourquoi |
| :-- | :-- | :-- |
| Votre `CLAUDE.md` du référentiel | Oui | Fait partie du clone |
| Vos hooks `.claude/settings.json` du référentiel | Oui | Fait partie du clone |
| Vos serveurs MCP `.mcp.json` du référentiel | Oui | Fait partie du clone |
| Votre `.claude/rules/` du référentiel | Oui | Fait partie du clone |
| Votre `.claude/skills/`, `.claude/agents/`, `.claude/commands/` | Oui | Fait partie du clone |
| Plugins déclarés dans `.claude/settings.json` | Oui | Installés au démarrage |
| Votre `~/.claude/CLAUDE.md` utilisateur | Non | Vit sur votre machine |
| Plugins activés uniquement dans vos paramètres utilisateur | Non | Vivent dans `~/.claude/settings.json` |
| Serveurs MCP ajoutés avec `claude mcp add` | Non | Configuration utilisateur locale |
| Jetons API statiques et identifiants | Non | Aucun magasin de secrets dédié |
| Authentification interactive comme AWS SSO | Non | Non pris en charge |

### Outils installés

| Catégorie | Inclus |
| :-------- | :----- |
| **Python** | Python 3.x avec pip, poetry, uv, black, mypy, pytest, ruff |
| **Node.js** | 20, 21 et 22 via nvm, avec npm, yarn, pnpm, bun¹, eslint, prettier, chromedriver |
| **Ruby** | 3.1, 3.2, 3.3 avec gem, bundler, rbenv |
| **PHP** | 8.4 avec Composer |
| **Java** | OpenJDK 21 avec Maven et Gradle |
| **Go** | dernière version stable |
| **Rust** | rustc et cargo |
| **C/C++** | GCC, Clang, cmake, ninja, conan |
| **Docker** | docker, dockerd, docker compose |
| **Bases de données** | PostgreSQL 16, Redis 7.0 |
| **Utilitaires** | git, jq, yq, ripgrep, tmux, vim, nano |

¹ Bun a des problèmes de compatibilité proxy connus pour la récupération de paquets.

Pour les versions exactes, demandez à Claude d'exécuter `check-tools` dans une session cloud.

### Travailler avec les problèmes et demandes de tirage GitHub

Les sessions cloud incluent des outils GitHub intégrés (lire les issues, lister les PRs, récupérer les diffs, publier des commentaires) sans configuration. Le CLI `gh` n'est pas pré-installé.

### Lier les artefacts à la session

Variable d'environnement `CLAUDE_CODE_REMOTE_SESSION_ID` disponible dans chaque session. URL de transcription :

```bash
echo "https://claude.ai/code/${CLAUDE_CODE_REMOTE_SESSION_ID/#cse_/session_}"
```

### Limites de ressources

* 4 vCPU
* 16 Go de RAM
* 30 Go de disque

### Configurer votre environnement

| Action | Comment |
| :----- | :------- |
| Ajouter un environnement | Sélectionnez l'environnement actuel → **Ajouter un environnement** |
| Modifier un environnement | Icône cloud → icône des paramètres |
| Archiver un environnement | Ouvrez l'environnement → **Archiver** |
| Définir la valeur par défaut pour `--remote` | Exécutez `/remote-env` dans votre terminal |

Variables d'environnement : format `.env`, une paire `KEY=value` par ligne, sans guillemets.

## Scripts de configuration

Un script Bash qui s'exécute au démarrage d'une nouvelle session cloud, avant le lancement de Claude Code. Scripts exécutés en tant que root sur Ubuntu 24.04.

```bash
#!/bin/bash
apt update && apt install -y gh
```

Conseils :
- Si le script se termine avec un code non nul, la session ne démarre pas.
- Gardez le temps d'exécution total en dessous de ~5 minutes.
- Exécutez les installations indépendantes en parallèle avec `&` et `wait`.

### Mise en cache de l'environnement

Après la première exécution du script de configuration, Anthropic crée un snapshot du système de fichiers. Les nouvelles sessions commencent avec vos dépendances déjà installées. Le cache est invalidé quand :
- Vous modifiez le script de configuration
- Vous modifiez les hôtes réseau autorisés
- Le cache atteint son expiration (~7 jours)

### Scripts de configuration vs. hooks SessionStart

| | Scripts de configuration | Hooks SessionStart |
| -- | -- | -- |
| Attaché à | L'environnement cloud | Votre référentiel |
| Configuré dans | Interface utilisateur cloud | `.claude/settings.json` |
| S'exécute | Avant le lancement de Claude Code (avec cache) | Après le lancement, sur chaque session |
| Portée | Environnements cloud uniquement | Local et cloud |

### Installer les dépendances avec un hook SessionStart

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/scripts/install_pkgs.sh"
          }
        ]
      }
    ]
  }
}
```

Script avec détection cloud :

```bash
#!/bin/bash
if [ "$CLAUDE_CODE_REMOTE" != "true" ]; then
  exit 0
fi
npm install
pip install -r requirements.txt
exit 0
```

Variable d'environnement `CLAUDE_CODE_REMOTE=true` dans les sessions cloud.

## Accès réseau

| Niveau | Connexions sortantes |
| :----- | :------------------- |
| **None** | Aucun accès réseau sortant |
| **Trusted** | Domaines autorisés uniquement : registres de paquets, GitHub, SDK cloud |
| **Full** | N'importe quel domaine |
| **Custom** | Votre propre liste d'autorisation |

### Domaines autorisés par défaut (niveau Trusted)

Catégories incluses :
- Services Anthropic (api.anthropic.com, claude.ai, etc.)
- Contrôle de version (github.com, gitlab.com, bitbucket.org)
- Registres de conteneurs (docker.io, gcr.io, ghcr.io, mcr.microsoft.com, public.ecr.aws)
- Plateformes cloud (Google, Azure, AWS, Oracle)
- Gestionnaires de paquets : npm, PyPI, RubyGems, crates.io, Go, Maven/Gradle, Composer (PHP), NuGet (.NET), pub.dev (Dart), hex.pm (Elixir)
- Distributions Linux (ubuntu.com, nixos.org)
- Outils de développement (Kubernetes, HashiCorp, Anaconda, Apache)
- Services cloud et surveillance (statsig, sentry, datadog, honeycomb)
- Model Context Protocol (*.modelcontextprotocol.io)

### Proxy GitHub

Toutes les opérations GitHub passent par un service proxy dédié qui :
- Gère l'authentification GitHub de manière sécurisée
- Restreint les opérations de poussée git à la branche de travail actuelle
- Permet le clonage, la récupération et les opérations PR

### Proxy de sécurité

Tout le trafic Internet sortant passe par un proxy HTTP/HTTPS.

## Déplacer les tâches entre le web et le terminal

### Du terminal au web (`--remote`)

```bash
claude --remote "Fix the authentication bug in src/auth/login.ts"
```

Crée une nouvelle session cloud. La VM clone depuis GitHub (poussez d'abord les commits locaux).

Conseils :
- **Planifiez localement, exécutez à distance** : `claude --permission-mode plan` puis `claude --remote "Execute the plan in docs/migration-plan.md"`
- **Tâches parallèles** : chaque `--remote` crée une session indépendante
- **Sans GitHub** : `CCR_FORCE_BUNDLE=1 claude --remote "..."` (limites : repo git avec commits, <100 Mo)

### Du web au terminal (`--teleport`)

```bash
claude --teleport          # sélecteur interactif
claude --teleport <id>     # session spécifique
```

Ou dans une session existante : `/teleport` (ou `/tp`).

Exigences de téléportation :
| Exigence | Détails |
| -------- | ------- |
| État git propre | Aucune modification non validée |
| Référentiel correct | Même référentiel, pas une fourche |
| Branche disponible | Branche poussée vers le serveur distant |
| Même compte | Même compte claude.ai |

## Travailler avec les sessions

### Gestion du contexte

| Commande | Fonctionne | Notes |
| :------- | :--------- | :---- |
| `/compact` | Oui | Résume la conversation |
| `/context` | Oui | Affiche le contenu de la fenêtre de contexte |
| `/clear` | Non | Démarrez une nouvelle session à la place |

Variables env pour la compaction : `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` et `CLAUDE_CODE_AUTO_COMPACT_WINDOW`.

### Partage des sessions

- Enterprise/Team : **Private** ou **Team** (vérification d'accès au référentiel activée par défaut)
- Max/Pro : **Private** ou **Public** (vérification d'accès non activée par défaut)

### Archiver / Supprimer

- **Archiver** : masque de la liste mais les sessions existantes continuent
- **Supprimer** : suppression définitive et irréversible

## Correction automatique des demandes de tirage (Auto-fix)

Claude surveille une PR et répond automatiquement aux défaillances CI et aux commentaires d'examen.

Nécessite l'application Claude GitHub installée sur le référentiel.

Façons d'activer :
- **PR créées dans Claude Code sur le web** : bouton **Auto-fix** dans la barre d'état CI
- **Depuis le terminal** : `/autofix-pr` sur la branche de la PR
- **Depuis l'application mobile** : demander à Claude de corriger la PR
- **N'importe quelle PR existante** : coller l'URL de la PR

### Comment Claude répond à l'activité PR

- **Corrections claires** : apporte la modification, pousse et explique
- **Demandes ambiguës** : demande avant d'agir
- **Événements en double ou sans action** : le note et continue

**Avertissement** : les réponses de Claude sont publiées sous votre compte GitHub (étiquetées Claude Code). Peut déclencher l'automatisation basée sur les commentaires (Atlantis, Terraform Cloud, etc.).

## Sécurité et isolation

* **Machines virtuelles isolées** : chaque session dans une VM isolée
* **Contrôles d'accès réseau** : limités par défaut
* **Protection des identifiants** : identifiants git gérés via proxy sécurisé, jamais dans le sandbox
* **Analyse sécurisée** : code analysé dans des VM isolées avant la création de PR

## Dépannage

### Échec de la création de session

- Vérifiez [status.claude.com](https://status.claude.com)
- Réessayez après une minute
- Vérifiez l'accès au référentiel GitHub

### Session Remote Control expirée ou accès refusé

- Exécutez `/login` localement
- Confirmez même compte proprietaire de la session

### Environnement expiré

Rouvrez depuis [claude.ai/code](https://claude.ai/code) pour un environnement frais avec l'historique restauré.

## Limitations

* **Limites de débit** : partagées avec tous les usages Claude/Claude Code
* **Authentification du référentiel** : nécessite le même compte pour web↔local
* **Restrictions de plateforme** : nécessite GitHub (GitLab/Bitbucket en lecture uniquement via bundle)
* **Liste d'autorisation IP** : les sessions cloud appellent depuis l'infrastructure Anthropic, pas votre réseau

## Ressources connexes

* [Ultraplan](/fr/ultraplan)
* [Ultrareview](/fr/ultrareview)
* [Routines](/fr/routines)
* [Configuration des hooks](/fr/hooks)
* [Référence des paramètres](/fr/settings)
* [Sécurité](/fr/security)
* [Utilisation des données](/fr/data-usage)
