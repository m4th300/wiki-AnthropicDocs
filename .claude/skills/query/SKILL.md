---
name: query
description: Consulte et synthétise le contenu du wiki. Déclenché par toute question sur le contenu du wiki, "que dit le wiki sur...", demande de synthèse, de comparaison, ou d'analyse. À utiliser dès que l'utilisateur cherche à exploiter ou croiser des connaissances déjà ingérées.
---

# /query — Consultation du wiki

Le wiki grossit par deux chemins : les ingests et les queries. Une exploration qui produit une synthèse nouvelle, une connexion non-évidente, ou une tension identifiée est aussi précieuse qu'une source ingérée — elle ne doit pas disparaître dans l'historique du chat. Mais c'est l'utilisateur qui décide ce qui mérite d'être archivé.

## Étapes

### 1. Identifier les pages pertinentes
Lire `wiki/index.md`. Identifier les pages susceptibles de contenir une réponse partielle ou complète.

### 2. Lire les pages ciblées
Lire uniquement les pages identifiées — pas le wiki en entier.

### 3. Choisir le format de réponse
Adapter la forme à la question :
- **Page markdown** — analyse, exploration d'un concept, réponse développée
- **Tableau comparatif** — comparaison entre entités, frameworks, ou sources
- **Liste synthétique** — points clés, résumé rapide, inventaire
- **Slides Marp ou graphique** — si l'utilisateur a indiqué utiliser Marp ou matplotlib, et que la question s'y prête

### 4. Synthétiser
Répondre avec des citations `[[chemin/page]]` pour chaque affirmation issue du wiki.
Signaler explicitement ce que le wiki ne contient pas sur un aspect de la question.

### 5. Proposer l'archivage si pertinent
Si la synthèse produit une relation non-triviale, une tension identifiée, ou une analyse originale, proposer de l'archiver dans le wiki. Ne jamais créer la page sans confirmation explicite.

La proposition doit inclure :
- Le titre suggéré pour la page
- Le type (`concept`, `framework`, ou autre)
- La structure proposée (sections principales)

Exemple de proposition :
> "Cette synthèse pourrait être archivée sous `wiki/concepts/biais-de-confirmation.md` avec les sections : Description, Mécanisme, Connexions avec [[...]], Sources. Tu veux que je la crée ?"

La conversation peut se poursuivre pour affiner le contenu ou la structure avant archivage.

Créer la page uniquement quand l'utilisateur dit explicitement "archive", "crée la page", "enregistre ça", ou un équivalent non ambigu.

Si la réponse est un simple lookup factuel sans synthèse nouvelle : ne pas proposer d'archivage.

### 6. Si archivage confirmé
Créer la page depuis le template `wiki/meta/` approprié.
Remplir le frontmatter : `created`, `updated`, `status: draft`, `sources_count`.
Mettre à jour `wiki/index.md`.
Logger dans `wiki/log.md` sous l'action `query` :
```
## [YYYY-MM-DD] query | Titre de la page archivée
Page créée : [[chemin/page]]
```
