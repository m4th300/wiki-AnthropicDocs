# Configurer les permissions

Source: https://code.claude.com/docs/fr/permissions

## Système d'autorisation

| Type d'outil | Exemple | Approbation requise | "Oui, ne pas demander à nouveau" |
|-------------|---------|--------------------|---------------------------------|
| Lecture seule | Lectures de fichiers, Grep | Non | S/O |
| Commandes Bash | Exécution shell | Oui | Permanent par répertoire et commande |
| Modification de fichier | Édition/écriture | Oui | Jusqu'à la fin de la session |

## Gérer les permissions avec `/permissions`

Règles : `Allow` (autoriser), `Ask` (demander), `Deny` (refuser)
Ordre d'évaluation : **deny → ask → allow** (premier match gagne)

Un nom d'outil simple (`Bash`) → supprime l'outil du contexte de Claude entièrement.
Une règle délimitée (`Bash(rm *)`) → laisse l'outil disponible mais bloque les appels correspondants.

## Syntaxe des règles

```json
{
  "permissions": {
    "allow": ["Bash(npm run lint)", "Bash(npm run test *)", "Read(~/.zshrc)"],
    "deny": ["Bash(curl *)", "Read(./.env)", "Read(./secrets/**)", "Bash(git push *)"]
  }
}
```

### Correspondance

- `Bash` ou `Bash(*)` → tous les usages
- `Bash(npm run build)` → commande exacte
- `Bash(npm run test *)` → préfixe avec wildcard
- `Bash(git * main)` → milieu wildcard (correspond à `git checkout main`)
- `Bash(* --version)` → suffixe wildcard

### Règles Bash avancées

**Commandes composées** : Un `*` peut s'étendre sur plusieurs arguments et espaces.
`Bash(ls *)` ≠ `Bash(ls*)` : avec espace = limite de mot (ls -la mais pas lsof).

**Wrappers supprimés** : `timeout`, `time`, `nice`, `nohup`, `stdbuf`, `xargs` (nu).
Une règle `Bash(npm test *)` match aussi `timeout 30 npm test`.

**Commandes en lecture seule** (exécutées sans invite) :
`ls`, `cat`, `echo`, `pwd`, `head`, `tail`, `grep`, `find`, `wc`, `which`, `diff`, `stat`, `du`, `cd`, et formes lecture seule de `git`.

### Règles Read/Edit

Formats de chemin (syntaxe gitignore) :
| Modèle | Signification |
|--------|--------------|
| `//path` | Chemin absolu à partir de la racine FS |
| `~/path` | Chemin depuis le répertoire home |
| `/path` | Relatif à la racine du projet |
| `path` ou `./path` | Relatif au répertoire courant |

Exemples :
- `Read(./.env)` → lit `.env` dans le répertoire courant
- `Edit(/src/**/*.ts)` → édite tous les .ts dans src/ du projet
- `Read(~/.zshrc)` → lit .zshrc du home
- `Read(//**/.env)` → lit `.env` n'importe où sur le FS

`*` = un niveau de répertoire. `**` = récursif.

### Règles WebFetch
`WebFetch(domain:example.com)` → requêtes vers example.com

### Règles MCP
- `mcp__puppeteer` → tous les outils du serveur puppeteer
- `mcp__puppeteer__puppeteer_navigate` → outil spécifique

### Règles Agent
- `Agent(Explore)` → subagent Explore
- `Agent(my-custom-agent)` → agent personnalisé

## Commandes en lecture seule auto-approuvées

`ls`, `cat`, `echo`, `pwd`, `head`, `tail`, `grep`, `find`, `wc`, `which`, `diff`, `stat`, `du`, `cd` et git en lecture seule.
`cd` dans un sous-répertoire du répertoire de travail = lecture seule.

## Répertoires de travail

Étendre l'accès :
- Au démarrage : `--add-dir <path>`
- Pendant la session : `/add-dir`
- Persistent : `additionalDirectories` dans settings.json

`--add-dir` charge aussi les skills, certains paramètres plugin, et (si activé) les CLAUDE.md.

## Modes d'autorisation

Voir aussi [Modes de permission](permission-modes.md) pour le détail complet.

| Mode | Description |
|------|-------------|
| `default` | Demande à chaque usage |
| `acceptEdits` | Auto-approuve éditions + commandes FS courantes |
| `plan` | Lecture seule, propose un plan |
| `auto` | Classificateur en arrière-plan (aperçu) |
| `dontAsk` | Refuse tout sauf pré-approuvés |
| `bypassPermissions` | Ignore tout (conteneurs isolés uniquement) |

## Hooks pour étendre les permissions

Un hook PreToolUse peut refuser, forcer une invite, ou ignorer l'invite.
Les décisions du hook ne contournent pas les règles d'autorisation :
- Règles de refus s'appliquent toujours
- Hook qui se termine avec code 2 → bloque même si une règle `allow` correspondrait

## Paramètres gérés (organisation)

Paramètres managed-only (uniquement dans managed-settings.json) :
- `allowManagedPermissionRulesOnly` : seules les règles managed s'appliquent
- `allowManagedMcpServersOnly` : seuls les MCP managed
- `allowManagedHooksOnly` : seuls les hooks managed
- `strictPluginOnlyCustomization` : bloquer customisation user/project
- `disableBypassPermissionsMode` : bloquer bypassPermissions

## Précédence des paramètres

Managed > CLI args > Local > Project > User

Si un outil est refusé à n'importe quel niveau → aucun autre ne peut l'autoriser.

## Permissions + Sandboxing

- Permissions : contrôlent quels outils Claude peut utiliser
- Sandboxing : isolation OS pour les commandes Bash

Les deux ensembles de règles se cumulent. Un refus de permission prévient que Claude essaie. Le sandbox prévient l'exécution même si Claude tente.
