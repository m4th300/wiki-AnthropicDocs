---
name: lint
description: Audite la cohérence et la complétude du wiki. Déclenché par "lint", "audite le wiki", "vérifie la cohérence", "qu'est-ce qui manque".
---

# /lint — Audit du wiki

## Étapes

### 1. Lire l'index
Lire `wiki/index.md` pour avoir la carte complète du wiki.

### 2. Lire `wiki/log.md`
Identifier la date des dernières opérations par catégorie pour repérer les synthèses potentiellement périmées.

### 3. Vérifier chaque catégorie

**Liens cassés** — `[[...]]` dans les pages qui pointent vers une page absente de l'index

**Pages orphelines** — pages dans l'index sans aucun lien entrant depuis une autre page

**Lacunes internes** — concepts, personnes ou frameworks mentionnés dans le texte mais sans page propre dans le wiki

**Lacunes externes** — thèmes ou concepts centraux dans le wiki qui s'appuient sur aucune source dans `raw/`, ou dont la couverture serait significativement améliorée par une source dédiée

**Contradictions** — assertions conflictuelles entre deux pages différentes

**Synthèses périmées** — sections "Synthèse courante" dont les informations semblent dépassées au vu des sources ingérées depuis

### 4. Produire le rapport

```
## Audit — [YYYY-MM-DD]

### Liens cassés
[liste ou "aucun"]

### Pages orphelines
[liste ou "aucune"]

### Lacunes internes
[liste ou "aucune"]

### Contradictions
[liste ou "aucune"]

### Synthèses périmées
[liste ou "aucune"]

### Lacunes externes
[concepts ou thèmes importants dans le wiki qui mériteraient une source dédiée absente de raw/ — avec pour chacun une suggestion concrète : type de source à chercher, question à investiguer, recherche web à faire]

### Suggestions
[3-5 questions ou pistes d'exploration pour la prochaine session]
```

### 5. Logger
Prepend dans `wiki/log.md` :
```
## [YYYY-MM-DD] lint | Audit wiki
Résultats : X liens cassés, X orphelins, X lacunes internes, X contradictions, X lacunes externes
```
