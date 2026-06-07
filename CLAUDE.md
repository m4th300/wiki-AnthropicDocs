# CLAUDE.md — Contrat opérationnel

Ce fichier est lu en début de chaque session. Il définit l'architecture du wiki, les conventions, et les workflows disponibles.

**Ce fichier co-évolue.** Si une convention ne fonctionne pas ou si un nouveau workflow émerge, réviser ce fichier en session et logger la modification dans `wiki/log.md` sous l'action `schema`.

---

## Pourquoi ce wiki existe

La plupart des usages de LLM avec des documents fonctionnent comme du RAG : on récupère des fragments à la volée, la synthèse est refaite à chaque question. Rien ne s'accumule.

Ce wiki fonctionne différemment. **La valeur est dans les connexions pré-compilées** — les liens entre pages, les contradictions déjà signalées, les synthèses déjà construites. Quand une nouvelle source arrive, le LLM ne l'indexe pas : il l'intègre dans la structure existante, révise les claims existants, et met à jour les connexions. La connaissance est compilée une fois et maintenue à jour — pas redérivée à chaque query.

Le wiki grossit par deux chemins :
- **Ingest** — une source externe enrichit le wiki
- **Query** — une exploration produit une synthèse qui rejoint le wiki

**Le rôle humain** : choisir les sources, poser les bonnes questions, diriger l'analyse, penser à ce que ça signifie.
**Le rôle du LLM** : tout le reste — résumer, relier, classer, maintenir la cohérence.

---

## Architecture

```
wiki/            ← pages maintenues par le LLM (lecture + écriture)
raw/             ← sources brutes (lecture seule — ne jamais modifier)
  assets/        ← images téléchargées localement (liées aux sources)
.claude/skills/  ← workflows externalisés (ne jamais modifier en session)
```

**Règle fondamentale** : `raw/` est immuable. Lire les sources et les images, ne jamais les modifier.

**Images** : les images liées à une source sont stockées dans `raw/assets/`. Quand une source contient des images, le LLM lit d'abord le texte, puis visualise les images référencées séparément pour enrichir le contexte. Les liens vers les images dans les pages wiki pointent vers `raw/assets/[nom-du-fichier]`.

---

## Démarrage de session

Lire dans cet ordre :
1. Ce fichier (`CLAUDE.md`)
2. `wiki/contexte.md` — état actuel du domaine
3. `wiki/index.md` — carte du contenu existant
4. Les 5 dernières entrées de `wiki/log.md` — ce qui a été fait récemment

---

## Conventions

**Slugs** — kebab-case, minuscules, sans accents (`e` pour `é/è/ê`, `a` pour `à`, `c` pour `ç`)

**Dates** — ISO 8601 (`YYYY-MM-DD`)

**Nommage des sources** — préfixe date obligatoire : `YYYY-MM-DD_slug.md`

**Liens internes** — `[[chemin/page]]` (sans `.md`, relatif à `wiki/`)

**Sections "Synthèse courante"** — réécrites à chaque mise à jour, jamais accumulées

**Sections "Sources"** — append-only, ne jamais supprimer une entrée

**Log** — prepend (entrée la plus récente en haut), format strict :
```
## [YYYY-MM-DD] action | Titre
```
Actions valides : `ingest`, `query`, `lint`, `update`, `init`, `schema`

**Index** — une entrée par page, format :
```
- [[chemin/page]] — description en une ligne
```
Classé par catégorie, alphabétique dans chaque catégorie.

---

## Opérations

| Commande | Déclencheur | Rôle |
|----------|-------------|------|
| `/ingest` | "ingère [fichier]", "traite cette source" | Intègre une source brute dans le wiki |
| `/query`  | Question sur le contenu du wiki | Consulte, synthétise, et archive si pertinent |
| `/lint`   | "lint", "audite le wiki" | Vérifie la cohérence interne et suggère des sources externes |
| `/update` | Fin de session significative | Met à jour `contexte.md` |

Les workflows complets sont dans `.claude/skills/`.

---

## Types de pages

Les templates sont dans `wiki/meta/`.

| Type | Dossier | Usage |
|------|---------|-------|
| `concept`   | `wiki/concepts/`   | Idée, notion, théorie |
| `source`    | `wiki/sources/`    | Synthèse d'une source ingérée |
| `personne`  | `wiki/personnes/`  | Individu (auteur, sujet, collaborateur) |
| `framework` | `wiki/frameworks/` | Méthode, système, modèle d'analyse |

---

## Règles de qualité

- Ne jamais inventer de faits, citations ou sources
- Signaler explicitement toute incertitude
- Chaque page créée → référencée dans `wiki/index.md`
- Chaque opération → loguée dans `wiki/log.md`
- Signaler les contradictions entre pages plutôt que les ignorer
