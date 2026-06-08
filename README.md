# wiki-AnthropicDocs

Wiki personnel sur Claude Code et l'Agent SDK, maintenu par Claude Code (CLI).

## Principe

Le LLM maintient une wiki structurée dans `wiki/` à partir de sources brutes déposées dans `raw/`. Chaque ingestion met à jour les pages existantes, crée les nouvelles, et maintient l'index et le log. La connaissance s'accumule — elle n'est pas redécouverte à chaque session.

## Démarrage

1. Créer un nouveau repo depuis ce template
2. Ouvrir le repo dans Claude Code
3. Remplir `wiki/contexte.md` — domaine du wiki, premières informations
4. Déposer les premières sources dans `raw/`
5. Taper `/ingest [nom-du-fichier]`

## Structure

```
wiki/            ← pages maintenues par le LLM
  index.md       ← catalogue de tout le contenu
  log.md         ← journal chronologique des opérations
  contexte.md    ← état du domaine, mis à jour en fin de session
  meta/          ← templates de pages (concept, source, personne, framework)
  concepts/      ← idées, notions, théories
  sources/       ← synthèses des sources ingérées
  personnes/     ← individus
  frameworks/    ← méthodes, modèles, systèmes
raw/             ← sources brutes (immuables — le LLM ne les modifie jamais)
  assets/        ← images liées aux sources (téléchargées localement)
.claude/skills/  ← workflows Claude Code
```

## Commandes

| Commande | Usage |
|----------|-------|
| `/init` | Initialiser le wiki pour un domaine spécifique |
| `/ingest [fichier]` | Intégrer une source depuis `raw/` |
| `/ingest --fast [fichier]` | Ingestion rapide sans discussion préalable |
| `/query` | Poser une question sur le contenu du wiki |
| `/lint` | Auditer la cohérence (liens cassés, orphelins, lacunes) |
| `/update` | Mettre à jour `contexte.md` en fin de session |

## Images

Les images liées aux sources sont stockées dans `raw/assets/`. Ne jamais référencer d'URLs externes dans les pages wiki — elles expirent. Avec Obsidian : *Settings → Files and links → Attachment folder path* → définir sur `raw/assets/`.

## Git

Le wiki est un repo git. Committer après chaque session d'ingestion significative pour conserver un historique des révisions et pouvoir récupérer une version précédente en cas de réécriture accidentelle.
