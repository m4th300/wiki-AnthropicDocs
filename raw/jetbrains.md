# JetBrains IDEs

Source: https://code.claude.com/docs/fr/jetbrains

## IDEs supportés

IntelliJ IDEA, PyCharm, Android Studio, WebStorm, PhpStorm, GoLand

## Fonctionnalités

- Lancement rapide : `Cmd+Esc` (Mac) / `Ctrl+Esc` (Win/Linux)
- Affichage des différences dans la visionneuse native de l'IDE
- Contexte de sélection : la sélection actuelle est automatiquement partagée avec Claude
  (Note : les règles de refus `Read` bloquent ce partage pour les fichiers correspondants)
- Raccourcis référence de fichier : `Cmd+Option+K` (Mac) / `Alt+Ctrl+K` (Linux/Win) → `@src/auth.ts#L1-99`
- Partage des diagnostics : erreurs lint/syntaxe partagées automatiquement

## Installation

Plugin : [Claude Code Beta](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-) depuis la Marketplace JetBrains.

Redémarrer l'IDE après installation (peut nécessiter plusieurs redémarrages).

## Utilisation

Depuis le terminal intégré : `claude` puis fonctionnalités actives automatiquement.
Depuis terminal externe : exécuter `/ide` dans Claude Code pour connecter à l'IDE.

## Configuration

### Paramètres Claude Code
Dans Claude Code : `/config` → définir outil de différence sur `auto` (affichage dans IDE) ou `terminal`.

### Paramètres du plugin
Settings → Outils → Claude Code [Beta]

| Paramètre | Description |
|-----------|-------------|
| Commande Claude | Chemin personnalisé vers le binaire Claude |
| Option+Entrée multi-lignes (macOS) | Désactiver si Option est capturée de manière inattendue |
| Mises à jour automatiques | Vérification et installation auto des updates |

**Pour WSL** : définir `wsl -d Ubuntu -- bash -lic "claude"` comme commande Claude.

### Configuration de la touche ESC

Si ESC n'interrompt pas les opérations :
Settings → Outils → Terminal → Décocher "Déplacer le focus vers l'éditeur avec Échap"

## Configurations spéciales

### Développement à distance

⚠️ Installer le plugin sur l'HÔTE DISTANT via Settings → Plugin (Hôte), pas sur la machine cliente.

### Configuration WSL (problème "Aucun IDE détecté")

**Cause** : réseau NAT WSL2 ou Pare-feu Windows bloque la connexion.
WSL1 utilise directement le réseau de l'hôte (non affecté).

**Solution 1 : Règle Pare-feu Windows**
```powershell
New-NetFirewallRule -DisplayName "Allow WSL2 Internal Traffic" -Direction Inbound -Protocol TCP -Action Allow -RemoteAddress 172.21.0.0/16 -LocalAddress 172.21.0.0/16
```

**Solution 2 : Mise en réseau en miroir (Windows 11 22H2+)**
Dans `.wslconfig` :
```ini
[wsl2]
networkingMode=mirrored
```
Puis `wsl --shutdown`.

## Considérations de sécurité

Avec permissions d'édition automatique activées, Claude Code peut modifier les fichiers de configuration IDE (qui peuvent être exécutés automatiquement par l'IDE). Utiliser le mode d'approbation manuelle recommandé.
