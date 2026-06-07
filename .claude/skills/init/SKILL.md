---
name: init
description: Initialise un nouveau wiki depuis le template llm-wiki-template. Déclenché par "/init", "initialise le wiki", "configure le wiki", ou quand l'utilisateur ouvre un repo wiki fraîchement cloné et demande par où commencer. À utiliser obligatoirement avant toute ingestion sur un wiki vierge.
---

# /init — Initialisation du wiki

Un wiki vierge cloné depuis le template est générique par construction. Ce skill le transforme en un outil adapté à un domaine précis : il comprend le projet, propose des adaptations structurelles, les valide avec l'utilisateur, puis les implémente. Rien n'est écrit sans validation explicite.

## Phase 1 — Comprendre le domaine

Poser ces questions, une par une ou regroupées selon le contexte :

1. **Domaine** — Quel est le sujet ou domaine couvert par ce wiki ? (ex: cours de machine learning, veille concurrentielle, documentation API interne)
2. **Objectif** — Quel est l'objectif principal ? Choisir parmi : apprendre et consolider, prendre de meilleures décisions, produire des livrables, référencer une documentation.
3. **Langue** — Quelle est la langue de rédaction du wiki ? Toutes les pages seront dans cette langue.
4. **Sources** — Quel type de sources sera ingéré ? (articles web, PDFs, notes personnelles, transcripts vidéo, documentation technique, autre) Et à quelle fréquence approximative ?
5. **Types d'entités** — Quels types d'objets ce domaine manipule-t-il naturellement ? (ex: pour un wiki de cours → chapitres, exercices, auteurs ; pour un wiki business → marchés, concurrents, décisions)

## Phase 2 — Proposer les adaptations

Analyser les réponses et construire une proposition complète couvrant :

**Structure des pages**
- Les 4 types de base (concept, source, personne, framework) sont-ils suffisants ?
- Si non, proposer des types supplémentaires adaptés au domaine avec leur template de sections

**Taxonomie de tags**
- Proposer un jeu de tags de base cohérent avec le domaine
- Format : `domaine/[nom-du-wiki]` pour le tag de domaine, tags thématiques libres pour le reste

**Adaptations de CLAUDE.md**
- Des conventions spécifiques au domaine sont-elles nécessaires ? (nommage particulier, types de pages ajoutés, règles métier)
- Si oui, lister les lignes à modifier

**Skills à adapter ou créer**
- Les 4 skills existants (ingest, query, lint, update) couvrent-ils le domaine ?
- Si un workflow spécifique au domaine serait utile (ex: `/transcript` pour des vidéos, `/compare` pour une veille concurrentielle), le signaler sans le créer maintenant

**Squelette de pages**
- Proposer 3 à 5 premières pages wiki pertinentes pour le domaine (titre + type + liste des sections)
- Ces pages ne doivent pas être des sources — elles représentent les premières entrées conceptuelles ou structurelles naturelles du domaine

Présenter l'ensemble de cette proposition dans un récapitulatif clair avant toute implémentation.

## Phase 3 — Validation

Attendre la confirmation explicite de l'utilisateur sur :
- Les types de pages (garder les 4 de base, ajouter les types proposés, modifier)
- La taxonomie de tags
- Les adaptations de CLAUDE.md
- Le squelette de pages (valider titre par titre)

Intégrer les retours avant d'écrire quoi que ce soit.

## Phase 4 — Implémentation (sur confirmation)

Exécuter dans cet ordre :

1. **Remplir `wiki/contexte.md`**
   - Section Domaine : description en 2-3 phrases
   - Section Situation actuelle : état initial du projet ou de l'apprentissage
   - Laisser les autres sections en `*(à compléter au fil des sessions)*`

2. **Adapter `CLAUDE.md`** si des modifications ont été validées
   - Ajouter les types de pages supplémentaires dans la table Types de pages
   - Ajouter les conventions spécifiques dans la section Conventions
   - Logger le changement sous l'action `schema`

3. **Créer les templates de pages supplémentaires** dans `wiki/meta/` si de nouveaux types ont été validés
   - Utiliser la même structure que les templates existants
   - Frontmatter identique + sections adaptées au type

4. **Créer le squelette de pages validé**
   - Créer chaque page depuis le template correspondant
   - Remplir uniquement le frontmatter (created, updated, status: draft) et les titres de sections — laisser le contenu vide
   - `status: draft` sur toutes les pages squelette

5. **Mettre à jour `wiki/index.md`**
   - Ajouter toutes les pages créées dans les bonnes catégories

6. **Logger dans `wiki/log.md`**
```
## [YYYY-MM-DD] init | Initialisation — [nom du domaine]
Langue : [langue]
Types de pages : [liste]
Pages squelette créées : [liste]
Adaptations CLAUDE.md : [oui/non — détail si oui]
```

## Phase 5 — Amorçage

Conclure en indiquant :
- Les 2-3 premières sources à déposer dans `raw/` pour commencer à alimenter le wiki
- La commande à taper pour démarrer : `/ingest [nom-du-fichier]`
- Un rappel sur le mode fast-track si l'utilisateur a des sources nombreuses : `/ingest --fast [fichier]`
