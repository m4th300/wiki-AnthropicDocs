# Détecter les problèmes de sécurité au fur et à mesure que Claude écrit du code

> Installez le plugin security-guidance pour que Claude examine ses propres modifications de code à la recherche de vulnérabilités et les corrige dans la même session.

Le plugin de conseils en sécurité fait en sorte que Claude examine ses propres modifications de code à la recherche de vulnérabilités courantes pendant qu'il travaille et corrige ce qu'il trouve dans la même session. Le plugin détecte des problèmes tels que l'injection, la désérialisation non sécurisée et les API DOM non sécurisées avant que le code n'atteigne une demande de tirage.

Une fois installé, le plugin s'exécute automatiquement. Il n'y a rien à invoquer et aucune commande séparée à retenir.

Le plugin est le compagnon en session de Code Review, qui s'exécute sur les demandes de tirage. Ce plugin réduit ce qui atteint la PR. Code Review attrape ce qui le fait.

## Conditions préalables

* Claude Code CLI version 2.1.144 ou ultérieure
* Python 3.8 ou ultérieur sur votre `PATH`. Le plugin essaie `python3`, `python` et `py -3` dans cet ordre
* Un référentiel git pour le répertoire dans lequel vous travaillez

À la première exécution, le plugin crée un environnement virtuel sous `~/.claude/security/` et installe le Claude Agent SDK dedans, ce qui nécessite `pip` et un accès réseau. Si cette installation échoue, l'examen de commit revient à un examen unique au lieu d'un examen agentique. Sur Windows, l'étape d'environnement virtuel est ignorée.

## Installer le plugin

Dans une session Claude Code, installez à partir du marketplace officiel Anthropic :

```text
/plugin install security-guidance@claude-plugins-official
```

L'installation demande une portée. Choisissez la portée utilisateur pour que le plugin se charge dans chaque nouvelle session locale.

Si Claude Code signale que le marketplace n'est pas trouvé, exécutez d'abord :
```text
/plugin marketplace add anthropics/claude-plugins-official
```

Activez dans la session actuelle :

```text
/reload-plugins
```

### Activer dans les sessions cloud et les référentiels partagés

Pour les sessions Claude Code sur le web ou pour tous ceux qui clonent un référentiel, déclarez le plugin dans les paramètres vérifiés du projet :

```json
// .claude/settings.json
{
  "enabledPlugins": {
    "security-guidance@claude-plugins-official": true
  }
}
```

Les administrateurs peuvent activer le plugin à l'échelle de l'organisation en définissant `enabledPlugins` dans les paramètres gérés.

## Ce que le plugin vérifie

Le plugin examine le travail de Claude à trois points :

### À chaque modification de fichier

Correspondance de motif rapide sans appel de modèle (pas de coût d'utilisation). Exemples de catégories de motifs :

* Exécution de code dynamique : `eval(`, `new Function`, `os.system`, `child_process.exec`
* Désérialisation non sécurisée : `pickle`
* Injection DOM : `dangerouslySetInnerHTML`, `.innerHTML =`, `document.write`
* Fichiers de flux de travail : modifications sous `.github/workflows/`

La vérification s'exécute après que la modification soit appliquée et ajoute l'avertissement au contexte de Claude. Chaque avertissement se déclenche une fois par motif par fichier par session.

### À la fin de chaque tour

Un tour est un cycle de réponse complet. Après chaque tour, le plugin calcule un diff git de tout ce qui a changé dans l'arborescence de travail et l'envoie à un examen Claude séparé axé sur la sécurité. L'examen s'exécute en arrière-plan.

Cela détecte les problèmes qu'une correspondance de chaîne ne peut pas :

* Contournement d'autorisation
* Références d'objet direct non sécurisées
* Injection
* Falsification de requête côté serveur
* Cryptographie faible

L'examen couvre jusqu'à 30 fichiers modifiés par tour et se déclenche au maximum trois fois de suite avant de vous rendre la main.

### À chaque commit ou push que Claude effectue

Lorsque Claude exécute `git commit` ou `git push` via son outil Bash, le plugin exécute un examen agentique plus approfondi. Cet examen lit le code environnant, y compris les appelants, les désinfectants et les fichiers connexes, pour décider si une conclusion est réelle avant de la signaler.

Cette couche ne se déclenche que sur les commits et les pushes que Claude effectue via son outil Bash. Les commits que vous exécutez à partir de votre propre shell ne sont pas examinés. Les examens sont limités à 20 par heure glissante.

### Indépendance et limites de l'examen

Le plugin ne demande pas à la même instance Claude qui a écrit le code de se noter elle-même :
- La vérification par modification est une correspondance de chaîne déterministe sans modèle
- Les examens de fin de tour et de commit s'exécutent en tant qu'appel Claude séparé avec un contexte frais

Aucune des couches ne bloque les écritures ou les commits. Traitez le plugin comme une couche de défense en profondeur, pas une solution de sécurité complète.

## Ajouter vos propres règles

### Ajouter des conseils pour les examens soutenus par le modèle

Créez `.claude/claude-security-guidance.md` dans votre projet :

```markdown
# Conseils en sécurité pour ce référentiel

- Ne pas enregistrer `customer_id` ou `account_number` au niveau INFO ou supérieur.
- Toutes les routes sous `/admin` doivent appeler `require_role("admin")` avant toute lecture de base de données.
- Utilisez `crypto.timingSafeEqual` pour la comparaison de jetons au lieu de `===`.
```

Ces règles sont des conseils, pas des garde-fous déterministes. Une règle qui dit d'ignorer une classe de vulnérabilité ne supprime pas ces conclusions (additif uniquement).

### Ajouter des motifs personnalisés par modification

Créez `.claude/security-patterns.yaml` :

```yaml
patterns:
  - rule_name: internal_api_key
    substrings: ["sk_live_", "AKIA"]
    reminder: "Clé API codée en dur. Chargez les identifiants à partir du gestionnaire de secrets."
  - rule_name: tenant_unfiltered_query
    regex: "\\.objects\\.all\\(\\)"
    paths: ["**/src/tenants/**"]
    reminder: "Le code multi-locataire doit filtrer par org_id."
```

| Champ           | Type   | Description                                                                      |
| :-------------- | :----- | :------------------------------------------------------------------------------- |
| `rule_name`     | string | Identifiant affiché dans l'avertissement                                         |
| `reminder`      | string | Texte d'avertissement ajouté au contexte de Claude, limité à 1 KB               |
| `regex`         | string | Expression régulière Python                                                      |
| `substrings`    | list   | Sous-chaînes littérales ; fournissez ceci ou `regex`                             |
| `paths`         | list   | Motifs glob optionnels ; préfixez avec `**/` pour les chemins relatifs au projet |
| `exclude_paths` | list   | Motifs glob à ignorer                                                            |

Le plugin lit aussi `.claude/security-patterns.yml` et `.claude/security-patterns.json`. Limite de 50 règles personnalisées.

### Emplacements de recherche des fichiers de règles

| Portée       | Chemin                                      | Notes                                        |
| :----------- | :------------------------------------------ | :------------------------------------------- |
| Utilisateur  | `~/.claude/claude-security-guidance.md`     | S'applique à chaque projet sur votre machine |
| Projet       | `.claude/claude-security-guidance.md`       | Vérifié avec le référentiel                  |
| Projet local | `.claude/claude-security-guidance.local.md` | Ignoré par Git                               |

Plafond combiné de 8 KB pour le fichier de conseils. Les mêmes chemins s'appliquent à `security-patterns.yaml`.

## Coût d'utilisation

* La vérification de motif par modification : aucun appel de modèle, aucun coût
* Les examens de fin de tour et de commit : dépensent chacun une utilisation de modèle supplémentaire
* L'examen de commit est agentique et peut prendre plusieurs tours de modèle par commit, limité à 20 par heure glissante
* Les deux examens soutenus par modèle utilisent **Claude Opus 4.7** par défaut
* Définissez `SECURITY_REVIEW_MODEL` pour l'examen de fin de tour et `SG_AGENTIC_MODEL` pour l'examen de commit

Le plugin est disponible sur tous les plans.

## Désactiver ou désinstaller

### Variables d'environnement pour désactiver des couches individuelles

| Variable                        | Effet                                                           |
| :------------------------------ | :-------------------------------------------------------------- |
| `ENABLE_PATTERN_RULES=0`        | Désactiver la vérification de motif par modification           |
| `ENABLE_STOP_REVIEW=0`          | Désactiver l'examen diff de fin de tour                         |
| `ENABLE_COMMIT_REVIEW=0`        | Désactiver l'examen de commit et de push                        |
| `ENABLE_CODE_SECURITY_REVIEW=0` | Désactiver tous les examens soutenus par le modèle              |
| `SECURITY_GUIDANCE_DISABLE=1`   | Désactiver le plugin entièrement sans le désinstaller           |

### Commandes de désactivation/désinstallation

```text
# Mettre en pause
/plugin disable security-guidance@claude-plugins-official

# Supprimer
/plugin uninstall security-guidance@claude-plugins-official
```

Si le plugin a été activé via `.claude/settings.json` d'un projet, le désactiver écrit un remplacement dans `.claude/settings.local.json` plutôt que de modifier le fichier vérifié. S'il a été activé via les paramètres gérés, seul un administrateur peut le désactiver.

## Comment le plugin s'intègre avec Claude Code

Le plugin est entièrement construit sur hooks. Il enregistre :

| Événement Hook                                                  | Objectif                                                                     |
| :-------------------------------------------------------------- | :--------------------------------------------------------------------------- |
| `SessionStart`                                                  | Amorcer l'environnement Python du plugin                                     |
| `UserPromptSubmit`                                              | Capturer la ligne de base de l'arborescence de travail                        |
| `PostToolUse` sur `Edit`, `Write` et `NotebookEdit`             | Correspondance de motif par modification                                      |
| `Stop`                                                          | Examen diff de fin de tour, exécuté en arrière-plan                          |
| `PostToolUse` sur `Bash`, filtré sur `git commit` et `git push` | Examen de commit et de push, exécuté en arrière-plan                         |

## Comment cela s'intègre avec d'autres outils de sécurité

Une pile typique de défense en profondeur :

| Étape                 | Outil                                                             | Ce qu'il couvre                                                                         |
| :-------------------- | :---------------------------------------------------------------- | :-------------------------------------------------------------------------------------- |
| En session            | Plugin de conseils en sécurité                                    | Vulnérabilités courantes dans le code que Claude écrit, corrigées dans la même session  |
| À la demande          | `/security-review`                                                | Passage de sécurité unique sur la branche actuelle                                      |
| Sur demande de tirage | Code Review (plans Team et Enterprise)                            | Examen multi-agent avec contexte complet de la base de code                             |
| En CI                 | Vos analyseurs statiques existants et scanners de dépendances     | Règles spécifiques au langage, vérifications de la chaîne d'approvisionnement           |

## Dépannage

Le plugin écrit les diagnostics d'exécution dans `~/.claude/security/log.txt`.

Raisons courantes pour lesquelles une couche d'examen s'ignore sans message :

* Le répertoire n'est pas un référentiel git (examens de fin de tour et de commit nécessitent l'état git)
* La session n'a pas d'authentification Anthropic (seule la vérification de motif s'exécute)
* PyYAML non importable avec un `security-patterns.yaml` (utilisez `security-patterns.json` à la place)

## Ressources connexes

* [Code Review](/fr/code-review) : configurer l'examen multi-agent au moment de la PR
* [Automatiser les flux de travail avec des hooks](/fr/hooks-guide) : créer vos propres vérifications
* [Découvrir et installer des plugins](/fr/discover-plugins#official-anthropic-marketplace) : parcourir d'autres plugins officiels
