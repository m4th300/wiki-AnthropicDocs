---
name: ingest
description: Intègre une source brute dans le wiki. Déclenché par "ingère [fichier]", "traite cette source", "ajoute ce document", ou "/ingest --fast [fichier]" pour le mode rapide. À utiliser dès qu'un fichier est déposé dans raw/ ou qu'une nouvelle source doit être intégrée au wiki.
---

# /ingest — Intégration d'une source

Une source bien ingérée peut toucher 10 à 15 pages existantes. L'objectif n'est pas seulement d'ajouter du contenu — c'est de réviser activement ce que le wiki affirme déjà, à la lumière de ce que la source apporte, contredit, ou nuance.

## Modes

**Mode normal** — `/ingest [fichier]` : inclut une discussion avec l'utilisateur avant d'écrire quoi que ce soit.

**Mode fast-track** — `/ingest --fast [fichier]` : l'étape de discussion est skippée, le LLM ingère directement. À utiliser pour des sources dont l'angle est évident ou quand l'utilisateur veut aller vite. Toutes les autres étapes sont identiques.

---

## Étapes (dans l'ordre strict)

### 1. Lire la source
Lire le fichier indiqué dans `raw/`. Si le fichier n'y est pas, demander à l'utilisateur de l'y déposer avant de continuer.

Après avoir lu le texte, détecter si la source contient des références à des images (syntaxe `![...](raw/assets/...)` ou équivalent). Si oui, visualiser les images référencées séparément — le LLM ne peut pas traiter texte et images en une seule passe. Ne visualiser que les images qui semblent apporter du contexte non redondant avec le texte.

### 2. Discuter avant d'écrire *(mode normal uniquement — skippée en mode --fast)*
Présenter 2-3 points clés extraits de la source. Demander :
- Ce qui résonne ou surprend
- L'angle à prioriser
- Les connexions évidentes avec le contenu existant du wiki

Ne rien écrire dans le wiki avant d'avoir eu cet échange.

### 3. Créer la page source
Créer `wiki/sources/YYYY-MM-DD_slug.md` depuis le template `wiki/meta/source.md`.
Remplir toutes les sections. Mettre à jour les champs frontmatter : `created`, `updated`, `author`, `date_source`, `raw_file`.

Si des images ont contribué à la compréhension de la source, les lister dans une section **Ressources visuelles** en bas de page :
```
## Ressources visuelles
- `raw/assets/[nom-du-fichier]` — [description courte de ce que l'image montre]
```
Ne jamais utiliser d'URLs externes pour référencer des images — elles expirent. Toujours pointer vers `raw/assets/`.

### 4. Réviser les pages existantes
Lire `wiki/index.md` pour identifier toutes les pages potentiellement concernées par la source. Lire chacune d'elles, puis pour chaque page :

- **Si la source enrichit une claim existante** → mettre à jour la section concernée et incrémenter `sources_count` dans le frontmatter
- **Si la source contredit une claim existante** → réécrire la section, signaler la tension dans les deux pages, mettre à jour `updated`
- **Si la source introduit un angle absent** → créer une nouvelle section ou une nouvelle page depuis `wiki/meta/`

Quand une section "Synthèse courante" est impactée, la réécrire entièrement — ne pas y ajouter des paragraphes supplémentaires.

### 5. Mettre à jour l'index
Ajouter dans `wiki/index.md` toutes les pages créées ou modifiées, dans la bonne catégorie, en ordre alphabétique.

### 6. Logger
Prepend dans `wiki/log.md` :
```
## [YYYY-MM-DD] ingest | Titre de la source
Mode : [normal | fast-track]
Pages créées : [liste]
Pages mises à jour : [liste]
```

### 7. Rapport final
- Lister les pages créées et mises à jour
- Signaler les tensions ou contradictions identifiées avec le contenu existant
- Proposer 1-2 questions d'exploration issues de la source
