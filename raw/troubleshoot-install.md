# Dépanner l'installation et la connexion

Source: https://code.claude.com/docs/fr/troubleshoot-install

## Table d'erreurs rapide

| Erreur | Solution |
|--------|----------|
| `command not found: claude` | Corriger PATH |
| `syntax error near unexpected token '<'` | Script install retourne HTML |
| `curl: (22) 403` | Script install retourne HTML |
| `curl: (23)` ou `(56)` | Vérifier connectivité |
| `Killed` pendant install Linux | Ajouter espace d'échange |
| `TLS connect error` | Mettre à jour certs CA |
| `Failed to fetch version` | Vérifier réseau/proxy |
| `irm is not recognized` | Mauvaise commande (CMD vs PowerShell) |
| `'bash' is not recognized` | Commande macOS/Linux exécutée sur Windows |
| `Error loading shared library` | Mauvaise variante binaire (musl/glibc) |
| `Illegal instruction` | Incompatibilité architecture/CPU |
| `Exec format error` dans WSL | Régression binaire WSL1 |
| `OAuth error: Invalid code` | Code expiré → réessayer vite |
| `403 Forbidden` | Abonnement/rôle/proxy |
| `Could not load credentials from any providers` | Credentials Bedrock/Vertex/Foundry |

## Vérifications de diagnostic

### Connectivité réseau
```bash
curl -sI https://downloads.claude.ai/claude-code-releases/latest
# → HTTP/2 200 = OK
```

Avec proxy :
```bash
export HTTPS_PROXY=http://proxy.example.com:8080
curl -fsSL https://claude.ai/install.sh | bash
```

### Vérifier PATH
```bash
# macOS/Linux
echo $PATH | tr ':' '\n' | grep -Fx "$HOME/.local/bin"
# Corriger :
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

### Installations en conflit
```bash
which -a claude
ls -la ~/.local/bin/claude
ls -la ~/.claude/local/
npm -g ls @anthropic-ai/claude-code 2>/dev/null
```

Supprimer installation npm obsolète :
```bash
npm uninstall -g @anthropic-ai/claude-code
rm -rf ~/.claude/local
```

## Problèmes courants

### Script install retourne HTML
→ App indisponible dans votre région, ou problème réseau/proxy.
Alternative : Homebrew (`brew install --cask claude-code`) ou WinGet.

### `command not found: claude`
→ `~/.local/bin` pas dans PATH. Voir correction PATH ci-dessus.

### Erreurs TLS/SSL
```bash
# Ubuntu/Debian
sudo apt-get install ca-certificates
# macOS : mettre à jour macOS
# Windows
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
# Proxy d'entreprise
export NODE_EXTRA_CA_CERTS=/path/to/corporate-ca.pem
```

### Install interrompue (Linux, `Killed`)
→ Manque de RAM. Ajouter espace d'échange :
```bash
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
```

### WSL2/SSH/Conteneurs : navigateur s'ouvre sur mauvaise machine
→ Copier l'URL affichée dans le terminal, l'ouvrir dans le navigateur local. Ou :
```bash
export BROWSER="/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"
```

### Erreur OAuth
→ Réinitialiser : `/logout` → fermer → `claude` → `/login`.
→ Appuyer sur `c` pour copier l'URL OAuth si le navigateur ne s'ouvre pas.

### 403 Forbidden après connexion
- Pro/Max : vérifier abonnement sur claude.ai/settings
- Console : vérifier rôle "Claude Code" ou "Developer"

### `ANTHROPIC_API_KEY` remplace les identifiants OAuth
```bash
unset ANTHROPIC_API_KEY
# Supprimer de ~/.zshrc ou ~/.bashrc
```

### Credentials Bedrock/Vertex/Foundry
```bash
# Bedrock
aws sts get-caller-identity
# Vertex
gcloud auth application-default login
# Foundry
az login
```

### WSL1 : `Exec format error`
→ Convertir en WSL2 : `wsl --set-version <DistroName> 2`
→ Ou wrapper dans `~/.bashrc` : `/lib64/ld-linux-x86-64.so.2 "$HOME/.local/bin/claude" "$@"`

### Docker : fige pendant installation
→ Définir `WORKDIR /tmp` avant d'installer (évite analyse du filesystem entier).

## Toujours bloqué

1. GitHub Issues : https://github.com/anthropics/claude-code/issues
2. `claude doctor` pour diagnostic automatisé
3. `/feedback` dans Claude Code
