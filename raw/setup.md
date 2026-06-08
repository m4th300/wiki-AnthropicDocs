# Configuration avancée

Source: https://code.claude.com/docs/fr/setup

## Configuration requise

- **OS** : macOS 13.0+, Windows 10 1809+/Server 2019+, Ubuntu 20.04+, Debian 10+, Alpine 3.19+
- **Matériel** : 4 Go+ RAM, x64 ou ARM64
- **Réseau** : connexion Internet requise
- **Shell** : Bash, Zsh, PowerShell ou CMD
- **Localisation** : pays supportés par Anthropic

## Installation (méthodes)

### Native (recommandé)
```bash
# macOS, Linux, WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### Homebrew
```bash
brew install --cask claude-code          # canal stable (~1 semaine de retard)
brew install --cask claude-code@latest   # canal latest (immédiat)
```

### WinGet
```powershell
winget install Anthropic.ClaudeCode
```

### npm
```bash
npm install -g @anthropic-ai/claude-code
```

### apt / dnf / apk (Linux)
Voir la documentation pour les dépôts signés. Clé : `31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`.

## Windows : Natif vs WSL

| Option | Sandboxing | Quand l'utiliser |
|--------|-----------|-----------------|
| Windows natif | Non supporté | Projets Windows natifs |
| WSL 2 | Supporté | Chaînes Linux, sandbox |
| WSL 1 | Non supporté | Si WSL 2 indisponible |

Git for Windows optionnel mais recommandé (active l'outil Bash). Sans lui → PowerShell.

## Alpine Linux (musl)

Nécessite `libgcc`, `libstdc++`, `ripgrep` + `USE_BUILTIN_RIPGREP=0` dans settings.json.

## Vérification

```bash
claude --version
claude doctor    # diagnostic détaillé
```

## Authentification

Plans requis : Pro, Max, Team, Enterprise, Console. Pas le plan gratuit.
Fournisseurs cloud : Amazon Bedrock, Google Vertex AI, Microsoft Foundry.

## Mises à jour

- Native : auto-update en arrière-plan
- Homebrew/WinGet : manuelles par défaut
- Canal de version : `autoUpdatesChannel: "stable"` ou `"latest"` dans settings.json
- Version minimale : `minimumVersion: "2.1.100"` dans settings.json
- Désactiver auto-update : `DISABLE_AUTOUPDATER: "1"` dans `env` de settings.json
- Mise à jour manuelle : `claude update`

## Installation version spécifique

```bash
curl -fsSL https://claude.ai/install.sh | bash -s 2.1.89
curl -fsSL https://claude.ai/install.sh | bash -s stable
```

## Intégrité des binaires

Chaque version publie `manifest.json` avec checksums SHA256, signé GPG.
```bash
gpg --fingerprint security@anthropic.com
# → 31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE
```

Signatures de code plateforme :
- macOS : signé "Anthropic PBC", notarié Apple. `codesign --verify --verbose ./claude`
- Windows : signé "Anthropic, PBC". `Get-AuthenticodeSignature .\claude.exe`
- Linux : via manifeste ou gestionnaire de paquets

## Désinstallation

Native : `rm -f ~/.local/bin/claude && rm -rf ~/.local/share/claude`
Supprimer config : `rm -rf ~/.claude && rm ~/.claude.json` (supprime paramètres, sessions, MCP)
