# Utilisation de l'ordinateur (CLI)

Source: https://code.claude.com/docs/fr/computer-use

Note : Aperçu de recherche. macOS uniquement. Pro ou Max seulement. v2.1.85+. Pas disponible sur Team/Enterprise ni en mode non-interactif.

## Ce que c'est

Claude peut ouvrir des applications, contrôler votre écran, cliquer, taper et faire des captures d'écran.

Utile pour : applications natives, simulateurs iOS, outils GUI sans API.

Claude utilise d'abord : serveur MCP > Bash > Chrome > puis utilisation de l'ordinateur (en dernier recours).

## Activer

```text
/mcp    # Trouver "computer-use" → Activer
```

Puis accorder les autorisations macOS :
- **Accessibilité** : cliquer, taper, faire défiler
- **Enregistrement d'écran** : voir l'écran

## Utiliser

```text
Compilez la cible MenuBarStats, lancez-la, ouvrez les préférences
et vérifiez que le curseur d'intervalle fonctionne.

La modale coupe son pied de page sur les fenêtres étroites.
Redimensionnez, reproduisez le bogue, faites une capture d'écran, corrigez le CSS.

Ouvrez le simulateur iOS, lancez l'application, appuyez sur les écrans
d'intégration et dites-moi si un écran prend plus d'une seconde.
```

## Fonctionnement

- **Verrou à l'échelle de la machine** : une seule session à la fois
- **Applications masquées** : pendant que Claude travaille, les autres apps sont masquées
- **Terminal exclu** : la fenêtre terminal n'apparaît pas dans les captures d'écran
- **Captures d'écran réduites** : automatique (ex: 3456×2234 → 1372×887 sur MacBook Pro Retina)
- **Approbation par application** : Claude ne contrôle que les apps approuvées par session

Appuyer sur `Esc` n'importe où = arrêter immédiatement.

## Avertissements d'applications sensibles

| Avertissement | Concerne |
|-------------|---------|
| Équivalent à l'accès shell | Terminal, iTerm, VS Code, Warp, IDE |
| Peut lire/écrire n'importe quel fichier | Finder |
| Peut modifier les paramètres système | Paramètres système |

## Niveaux de contrôle par catégorie

- Navigateurs et plateformes de trading : lecture seule
- Terminaux et IDE : clic uniquement
- Autres : contrôle complet

## Différences CLI vs Desktop

| | Desktop | CLI |
|--|---------|-----|
| Plateformes | macOS + Windows | macOS uniquement |
| Activer | Paramètres > Général | `/mcp` → computer-use |
| Plans | Pro, Max, Team, Enterprise | Pro, Max uniquement |

## Dépannage

**"L'utilisation de l'ordinateur est utilisée par une autre session"** : terminer ou quitter l'autre session.
**Invite de permissions réapparaît** : redémarrer Claude Code complètement après avoir accordé l'enregistrement d'écran.
**`computer-use` n'apparaît pas dans `/mcp`** : vérifier macOS, v2.1.85+, plan Pro/Max, authentification claude.ai (pas fournisseur tiers), session interactive.
