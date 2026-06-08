# Dépannage (post-installation)

Source: https://code.claude.com/docs/fr/troubleshooting

## Tableau de redirection

| Symptôme | Page à consulter |
|---------|-----------------|
| `command not found`, installation échoue | [Dépannage installation](/fr/troubleshoot-install) |
| Boucles de connexion, OAuth, 403, identifiants cloud | [Dépannage installation - Login](/fr/troubleshoot-install#login-and-authentication) |
| Paramètres non appliqués, hooks ne se déclenchent pas | Déboguer votre configuration |
| `API Error: 5xx`, `529 Overloaded`, `429` | Référence des erreurs |
| `model not found` | Référence des erreurs - modèle |
| Extension VS Code ne connecte pas | VS Code - dépannage |
| Plugin JetBrains non détecté | JetBrains - dépannage |

**Premier réflexe** : exécuter `/doctor` (ou `claude doctor` si Claude ne démarre pas).

## Performance et stabilité

### Utilisation élevée du CPU ou de la mémoire

1. Utiliser `/compact` régulièrement
2. Fermer et redémarrer entre les tâches majeures
3. Ajouter les grands répertoires de construction au `.gitignore`
4. Si mémoire reste élevée : `/heapdump` → snapshot + ventilation sur `~/Desktop`

### Auto-compaction thrashing

Message : `Autocompact is thrashing: the context refilled to the limit...`

Solutions :
1. Lire le fichier surdimensionné en petits morceaux (plages de lignes)
2. `/compact` avec focus : `/compact keep only the plan and the diff`
3. Déléguer à un [subagent](/fr/sub-agents)
4. `/clear` si la conversation antérieure n'est pas nécessaire

### Commandes figées/gelées

1. `Ctrl+C` pour tenter d'annuler
2. Fermer le terminal si ne répond pas
3. `claude --resume` pour reprendre (la conversation n'est pas perdue)

### Texte garbled dans terminal intégré d'éditeur

(VS Code, Cursor, Devin Desktop)

Exécuter `/terminal-setup` → définit `terminal.integrated.gpuAcceleration: "off"`.
Ou définir manuellement dans les paramètres de l'éditeur + recharger la fenêtre.

## Problèmes de recherche et de découverte

Si Search, `@file`, agents/skills personnalisés ne trouvent pas les fichiers :

```bash
# macOS
brew install ripgrep

# Ubuntu/Debian
sudo apt install ripgrep

# Windows
winget install BurntSushi.ripgrep.MSVC
```

Puis définir `USE_BUILTIN_RIPGREP=0` dans l'environnement.

### Recherche lente sur WSL

Cause : pénalités de performance du système de fichiers WSL.
Solutions :
1. Recherches plus spécifiques ("Search for JWT in auth-service")
2. Déplacer le projet vers le FS Linux (`/home/`)
3. Utiliser Windows natif

## Obtenir plus d'aide

1. `/doctor` : vérification complète de la santé
2. `/feedback` : signaler directement à Anthropic
3. [GitHub issues](https://github.com/anthropics/claude-code)
4. Demander directement à Claude (a accès à sa propre documentation)
