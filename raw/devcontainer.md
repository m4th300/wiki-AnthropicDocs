# Conteneurs de développement

Source: https://code.claude.com/docs/fr/devcontainer

## Ce que c'est

Dev container = environnement identique et isolé pour toute l'équipe.
Claude Code s'exécute à l'intérieur → les commandes s'exécutent dans le conteneur, pas sur l'hôte.

## Ajouter Claude Code à votre dev container

Méthode recommandée : utiliser la [Fonctionnalité Claude Code Dev Container](https://github.com/anthropics/devcontainer-features/tree/main/src/claude-code).

```json
// .devcontainer/devcontainer.json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/anthropics/devcontainer-features/claude-code:1.0": {}
  }
}
```

Ensuite : **Dev Containers: Rebuild Container** → `claude` dans le terminal.

Note : la balise `:1.0` épingle le script d'installation, pas la version de Claude Code (installe toujours la dernière).

## Persister l'authentification entre les reconstructions

```json
"mounts": [
  "source=claude-code-config,target=/home/node/.claude,type=volume"
]
```

Pour isoler l'état par projet : `source=claude-code-config-${devcontainerId}`.

Dans GitHub Codespaces : stocker `ANTHROPIC_API_KEY` ou `CLAUDE_CODE_OAUTH_TOKEN` en tant que secret Codespaces.

## Appliquer la politique organisationnelle

```dockerfile
RUN mkdir -p /etc/claude-code
COPY managed-settings.json /etc/claude-code/managed-settings.json
```

Variables d'environnement :
```json
"containerEnv": {
  "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
  "DISABLE_AUTOUPDATER": "1"
}
```

Note : les ingénieurs peuvent modifier les fichiers du référentiel. Pour une politique incontournable → [paramètres gérés par le serveur](/fr/server-managed-settings).

Pour épingler une version spécifique de Claude Code :
```dockerfile
RUN npm install -g @anthropic-ai/claude-code@X.Y.Z
```
+ `DISABLE_AUTOUPDATER=1`

## Serveurs MCP dans le dev container

Définir à portée projet dans `.mcp.json` à la racine du référentiel.
Installer les binaires stdio dans le Dockerfile.
Ajouter les domaines des serveurs distants à la liste blanche réseau.

## Restreindre la sortie réseau

Référence : [`init-firewall.sh`](https://github.com/anthropics/claude-code/blob/main/.devcontainer/init-firewall.sh)
Nécessite `runArgs` avec `NET_ADMIN` et `NET_RAW`.
Voir [Exigences d'accès réseau](/fr/network-config).

## Exécuter sans invites de permission

```bash
claude --dangerously-skip-permissions
```

Possible car le conteneur confine l'exécution. Claude Code doit s'exécuter en tant que non-root.

⚠️ Claude peut toujours modifier les fichiers dans l'espace de travail monté en bind (apparaissent sur l'hôte).
Associer avec les restrictions réseau ci-dessus.

Pour empêcher les utilisateurs d'utiliser cet indicateur :
```json
{"permissions": {"disableBypassPermissionsMode": "disable"}}
```

## Conteneur de référence

`anthropics/claude-code/.devcontainer` = exemple avec CLI, pare-feu, volumes persistants, Zsh.
Cloner + **Dev Containers: Reopen in Container**.

| Fichier | Objectif |
|---------|---------|
| `devcontainer.json` | Montages, runArgs, extensions, containerEnv |
| `Dockerfile` | Image de base, outils, Claude Code |
| `init-firewall.sh` | Bloc trafic sortant sauf domaines autorisés |

## Avertissement de sécurité

Claude Code a accès aux fichiers dans l'espace de travail monté.
Avec `--dangerously-skip-permissions` : ne pas monter `~/.ssh` ou credentials cloud dans le conteneur.
Préférer les jetons limités au référentiel ou à courte durée de vie.
