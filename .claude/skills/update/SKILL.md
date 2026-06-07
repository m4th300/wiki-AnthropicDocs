---
name: update
description: Met à jour contexte.md après une session significative. Déclenché manuellement par "/update" ou "mets à jour le contexte". Aussi déclenché automatiquement et silencieusement par le LLM en fin de session si des informations nouvelles sur le domaine ont émergé.
---

# /update — Mise à jour du contexte

## Règle fondamentale
N'écrire que ce qui a été réellement observé ou appris dans la session courante.
Ne rien inventer. Ne rien extrapoler.

## Modes

**Mode manuel** — déclenché par `/update` ou "mets à jour le contexte". Le LLM exécute le workflow complet et présente les changements à l'utilisateur.

**Mode automatique silencieux** — déclenché par le LLM lui-même en fin de session, sans que l'utilisateur le demande. Le LLM exécute le même workflow mais signale brièvement en fin de réponse : *"contexte.md mis à jour — sections modifiées : [liste]."* Sans interrompre la conversation.

## Conditions pour le déclenchement automatique
Le mode automatique s'active quand au moins une de ces conditions est vraie dans la session :
- Au moins une source a été ingérée
- Au moins une page wiki a été créée via query
- Un échange a produit de nouvelles informations sur l'état du domaine, du projet, ou des questions ouvertes

Si aucune de ces conditions n'est remplie, ne pas déclencher le mode automatique.

## Étapes (communes aux deux modes)

### 1. Relire `wiki/contexte.md`
Identifier les sections dont le contenu a évolué durant la session.

### 2. Mettre à jour les sections pertinentes

- **Situation actuelle** — nouveaux faits sur le domaine ou l'état du projet
- **Projets en cours** — avancement, nouveaux projets, projets terminés
- **Fils conducteurs actifs** — thèmes émergents depuis les dernières sources, thèmes abandonnés à retirer
- **Questions ouvertes** — nouvelles questions soulevées, questions résolues à supprimer

Chaque section modifiée est réécrite, pas accumulée.
Les sections sans nouvelle information restent inchangées.
Les sections vides (`*(à compléter)*`) restent vides si rien ne justifie de les remplir.

### 3. Logger
Prepend dans `wiki/log.md` :
```
## [YYYY-MM-DD] update | contexte.md
Mode : [manuel | automatique]
Sections modifiées : [liste]
```
