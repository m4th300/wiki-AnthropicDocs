> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt

# Explorez la fenêtre de contexte

> Une simulation interactive de la façon dont la fenêtre de contexte de Claude Code se remplit pendant une session. Voyez ce qui se charge automatiquement, ce que coûte chaque lecture de fichier, et quand les règles et les hooks s'exécutent.

## Ce que la chronologie montre

La session parcourt un flux réaliste avec des comptages de jetons représentatifs :

* **Avant que vous ne tapiez quoi que ce soit** : CLAUDE.md, la mémoire automatique, les noms d'outils MCP, et les descriptions de compétences se chargent tous dans le contexte.
* **Pendant que Claude travaille** : chaque lecture de fichier s'ajoute au contexte, les règles délimitées par chemin se chargent automatiquement aux côtés des fichiers correspondants, et un hook PostToolUse s'exécute après chaque modification.
* **L'invite de suivi** : un sous-agent gère la recherche dans sa propre fenêtre de contexte séparée. Seul le résumé et une petite remorque de métadonnées reviennent.
* **À la fin** : `/compact` remplace la conversation par un résumé structuré.

## Éléments chargés au démarrage (avec tailles illustratives)

| Élément | Tokens | Visibilité |
| :------ | :----- | :--------- |
| System prompt | ~4200 | Invisible dans le terminal |
| Auto memory (MEMORY.md) | ~680 | Invisible (200 lignes ou 25 Ko max) |
| Environment info | ~280 | Invisible |
| MCP tools (deferred) | ~120 | Invisible |
| Skill descriptions | ~450 | Invisible (ne survit pas au /compact) |
| ~/.claude/CLAUDE.md | ~320 | Invisible |
| Project CLAUDE.md | ~1800 | Invisible |

## Ce qui survit à la compaction

Quand une longue session se compacte, Claude Code résume l'historique de la conversation. Ce qui arrive à vos instructions dépend de la façon dont elles ont été chargées :

| Mécanisme | Après compaction |
| :-------- | :--------------- |
| Invite système et style de sortie | Inchangé ; ne fait pas partie de l'historique des messages |
| CLAUDE.md à la racine du projet et règles sans portée | Réinjectés depuis le disque |
| Mémoire automatique | Réinjectés depuis le disque |
| Règles avec frontmatter `paths:` | Perdues jusqu'à ce qu'un fichier correspondant soit lu à nouveau |
| CLAUDE.md imbriqué dans les sous-répertoires | Perdu jusqu'à ce qu'un fichier de ce sous-répertoire soit lu à nouveau |
| Corps de compétences invoqués | Réinjectés, limités à 5 000 jetons par compétence et 25 000 jetons au total ; les plus anciens sont supprimés en premier |
| Hooks | Non applicable ; les hooks s'exécutent en tant que code, pas en tant que contexte |

Les règles délimitées par chemin et les fichiers CLAUDE.md imbriqués se chargent dans l'historique des messages quand leur fichier déclencheur est lu, de sorte que la compaction les résume avec tout le reste.

Les corps de compétences sont réinjectés après compaction, mais les grandes compétences sont tronquées pour s'adapter au plafond par compétence, et les compétences invoquées les plus anciennes sont supprimées une fois le budget total dépassé.

## Vérifiez votre propre session

La visualisation utilise des nombres représentatifs. Pour voir votre utilisation réelle du contexte à tout moment :
- Exécutez `/context` pour une ventilation en direct par catégorie avec des suggestions d'optimisation.
- Exécutez `/memory` pour vérifier quels fichiers CLAUDE.md et de mémoire automatique se sont chargés au démarrage.

## Comportements des sous-agents

Un sous-agent spawné reçoit :
- Son propre system prompt (plus court que la session principale)
- Project CLAUDE.md (sa propre copie)
- MCP tools + skills
- Un task prompt de la session principale

Les lectures de fichiers du sous-agent restent dans sa fenêtre de contexte séparée. Seul le résumé final revient à la session principale (économie de contexte typique : 6100 tokens lus → 420 tokens retournés).

## Compaction automatique

La compaction automatique remplace la conversation par un résumé structuré. Ce qui survit :
- Startup content (hors historique des messages) se recharge automatiquement
- Skill descriptions : exception — ne se rechargent pas, seulement celles invoquées sont préservées
- La compaction affiche : `{preCompactTotal} → {totalTokens} tokens · freed {difference}`

## Ressources connexes

* [Étendre Claude Code](/fr/features-overview)
* [Stocker les instructions et les mémoires](/fr/memory)
* [Sous-agents](/fr/sub-agents)
* [Meilleures pratiques](/fr/best-practices)
* [Mise en cache des invites](/fr/prompt-caching)
* [Réduire l'utilisation des jetons](/fr/costs#reduce-token-usage)
