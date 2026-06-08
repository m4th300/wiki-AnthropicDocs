# Sandboxing (outil Bash en sandbox)

Source: https://code.claude.com/docs/fr/sandboxing

## Ce que c'est

Isolation du système de fichiers et du réseau pour les commandes Bash. Vous définissez les limites ; le système d'exploitation les applique pour chaque commande et ses processus enfants.

**Plateformes** : macOS, Linux, WSL2. Windows natif = NON SUPPORTÉ.

## Démarrer

```text
/sandbox
```

Ouvre un panneau avec 3 onglets : Mode, Overrides, Config.

### Dépendances Linux/WSL2

```bash
# Ubuntu/Debian
sudo apt-get install bubblewrap socat

# Fedora
sudo dnf install bubblewrap socat

# Seccomp (optionnel, pour bloquer sockets Unix)
npm install -g @anthropic-ai/sandbox-runtime
```

### Ubuntu 24.04+ : autoriser bubblewrap

```bash
sudo tee /etc/apparmor.d/bwrap > /dev/null <<'EOF'
abi <abi/4.0>,
include <tunables/global>
profile bwrap /usr/bin/bwrap flags=(unconfined) {
  userns,
  include if exists <local/bwrap>
}
EOF
sudo systemctl reload apparmor
```

## Modes sandbox

**Mode auto-allow** : commandes sandboxées exécutées sans invite. Commandes nécessitant des hôtes non autorisés → flux de permission régulier.

Exceptions qui invitent toujours même en auto-allow :
- Règles de refus explicites
- `rm`/`rmdir` ciblant `/`, répertoire personnel, ou chemins système critiques
- Règles Ask pour commandes revenant au flux de permission régulier

**Mode permissions régulières** : garde les invites de permission même pour les commandes sandboxées.

**`allowUnsandboxedCommands`** : si une commande échoue sous le sandbox, Claude peut réessayer sans sandbox (avec votre approbation). Désactiver = mode strict.

## Configuration (`settings.json`)

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": ["~/.kube", "/tmp/build"],
      "denyRead": ["~/"],
      "allowRead": ["."],
      "denyWrite": ["~/Documents"]
    },
    "network": {
      "allowedDomains": ["api.github.com", "*.npmjs.org"],
      "deniedDomains": ["example.com"]
    }
  }
}
```

### Syntaxe des chemins

| Préfixe | Signification |
|---------|--------------|
| `/` | Absolu depuis la racine FS |
| `~/` | Depuis le répertoire home |
| `./` ou sans préfixe | Relatif à la racine du projet (dans settings.json du projet) |

Note : DIFFÉRENT de la syntaxe des règles de permission Read/Edit (qui utilisent `//path` pour absolu).

### Comportement réseau par défaut

- Aucun domaine pré-autorisé
- Première connexion à un nouveau domaine → Claude demande approbation
- `allowManagedDomainsOnly` dans les paramètres gérés → bloquer automatiquement les domaines non autorisés

### Fichiers d'identifiants

Par défaut, le sandbox PERMET la lecture de `~/.aws/credentials`, `~/.ssh/`, etc.
Pour bloquer : ajouter à `denyRead`.

## Comportement du système de fichiers

- **Écriture par défaut** : uniquement dans le répertoire de travail courant
- **Lecture par défaut** : tout l'ordinateur, sauf quelques répertoires refusés
- **Git worktrees** : autorise les écritures dans le répertoire `.git` partagé (sauf `hooks/` et `config`)
- **Fichiers settings.json Claude** : refus d'écriture automatique (auto-protection)

## Pour les organisations (paramètres gérés)

```json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "allowUnsandboxedCommands": false,
    "excludedCommands": ["docker *", "terraform *"],
    "filesystem": {
      "denyRead": ["~/.aws", "~/.ssh"]
    }
  }
}
```

- `failIfUnavailable: true` → erreur dure si sandbox non disponible (vs avertissement)
- `allowManagedReadPathsOnly: true` → seules les `allowRead` gérées honorées (les locales ignorées)
- `allowManagedDomainsOnly: true` → idem pour les domaines réseau

## Dépannage courant

| Problème | Solution |
|---------|---------|
| `jest` se bloque | Exécuter `jest --no-watchman` |
| CLI Go (gh, gcloud, terraform) échouent TLS sur macOS | Ajouter à `excludedCommands` |
| `docker` échoue | Ajouter `docker *` à `excludedCommands` |
| Bubblewrap échoue dans un conteneur | `enableWeakerNestedSandbox: true` |

## Limitations de sécurité

- **Réseau** : le proxy ne termine pas TLS → le contenu des connexions HTTPS n'est pas inspecté
- **Domain fronting** : possible sur les domaines larges (ex: `github.com`)
- **Sockets Unix** : autoriser `/var/run/docker.sock` = accès hôte via Docker
- **Variables d'environnement** : les commandes héritent de l'environnement parent (incluant les identifiants)
  → Supprimer : `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB`
- **WSL2** : ne peut pas lancer les binaires Windows (cmd.exe, powershell.exe)

## Portée du sandbox

UNIQUEMENT les sous-processus Bash. Pas les outils fichiers intégrés (Read/Edit/Write), ni l'utilisation ordinateur, ni les subagents (utilisent la même config sandbox que la session parent).

## Proxy réseau personnalisé

```json
{
  "sandbox": {
    "network": {
      "httpProxyPort": 8080,
      "socksProxyPort": 8081
    }
  }
}
```
