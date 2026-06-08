# Révision de code (Code Review)

Source: https://code.claude.com/docs/fr/code-review

Note : Aperçu de recherche. Team et Enterprise uniquement. Pas disponible avec Zero Data Retention.

## Ce que c'est

Analyse automatique des PRs GitHub via une flotte d'agents spécialisés. Publie des commentaires en ligne sur les lignes de code où des problèmes ont été trouvés.

Pour exécuter dans votre propre CI : voir [GitHub Actions] ou [GitLab CI/CD].

## Niveaux de gravité

| Marqueur | Gravité | Signification |
|----------|---------|---------------|
| 🔴 | Important | Bug à corriger avant la fusion |
| 🟡 | Nit | Problème mineur, non bloquant |
| 🟣 | Préexistant | Bug existant non introduit par cette PR |

Les résultats incluent un raisonnement étendu réductible.

## Évaluer les résultats

👍 / 👎 sur chaque commentaire → Anthropic collecte les réactions après la fusion pour affiner le réviseur.

Répondre aux commentaires NE déclenche PAS une re-révision. Pour agir : corriger le code et pousser.
Pour re-révision sans pousser : `@claude review once`.

## Configuration (par un admin)

1. [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code) → section Code Review → Setup
2. Installer la GitHub App Claude → permissions : Contents (R/W), Issues (R/W), Pull requests (R/W)
3. Sélectionner les référentiels
4. Définir le comportement de révision par référentiel :
   - **Once after PR creation** : une révision par PR
   - **After every push** : révise à chaque push, résout automatiquement les threads résolus
   - **Manual** : seulement via `@claude review`

## Déclencher manuellement

| Commande | Effet |
|---------|-------|
| `@claude review` | Démarre une révision + abonne la PR aux pushes futurs |
| `@claude review once` | Démarre une seule révision, pas d'abonnement |

Conditions :
- Commentaire PR de haut niveau (pas une ligne de diff)
- Commande au début du commentaire
- Accès propriétaire/membre/collaborateur au repo
- PR ouverte

Les déclencheurs manuels s'exécutent sur les PR brouillon (contrairement aux déclencheurs automatiques).

## Personnalisation

### `CLAUDE.md`

Code Review lit les fichiers CLAUDE.md. Les violations nouvellement introduites = nits.
Bidirectionnel : si votre PR rend une déclaration CLAUDE.md obsolète, Claude le signale.

### `REVIEW.md`

Fichier à la racine du repo, injecté en priorité la plus élevée dans chaque agent.
Instructions directes (pas de syntaxe @import). Limitez à ce qui change le comportement.

#### Ce que `REVIEW.md` peut contrôler

- **Gravité** : redéfinir ce que 🔴 Important signifie pour votre repo
- **Volume de nits** : "signaler au maximum 5 nits, mentionner le reste comme comptage"
- **Règles de saut** : chemins, branches, catégories (code généré, lockfiles, lint déjà dans CI)
- **Vérifications spécifiques** : "les nouveaux itinéraires API doivent avoir un test d'intégration"
- **Barre de vérification** : "les affirmations ont besoin d'une citation `file:line`"
- **Convergence** : "après la première révision, supprimer les nits, publier Important uniquement"
- **Forme du résumé** : "commencer par un comptage comme `2 factual, 4 style`"

Exemple `REVIEW.md` :
```markdown
## Ce que Important signifie ici
Réserver Important aux bugs qui cassent le comportement, fuient les données, ou bloquent un rollback.
Le style et les suggestions de refactorisation sont Nit au maximum.

## Limiter les nits
Signaler au maximum cinq Nits. Dire « plus N éléments similaires » dans le résumé pour le reste.

## Ne pas signaler
- Tout ce que CI applique déjà : lint, formatage, erreurs de type
- Fichiers sous `src/gen/` et fichiers `*.lock`

## Toujours vérifier
- Les nouveaux itinéraires API ont un test d'intégration
- Les lignes de log n'incluent pas les emails ou ID utilisateur
```

## Tarification

15-25$ par révision en moyenne (variable selon taille PR et complexité).
Facturé via crédits d'utilisation, séparement de l'utilisation du plan.
Configurer un plafond mensuel : [claude.ai/admin-settings/usage](https://claude.ai/admin-settings/usage).

## Sortie de l'exécution de vérification

- **Check run "Claude Code Review"** → Details = tableau de gravité + annotations
- Conclusion toujours neutre (ne bloque jamais la fusion)
- Parse la conclusion :
```bash
gh api repos/OWNER/REPO/check-runs/CHECK_RUN_ID \
  --jq '.output.text | split("bughunter-severity: ")[1] | split(" -->")[0] | fromjson'
```
Retourne : `{"normal": 2, "nit": 1, "pre_existing": 0}`

## Commande locale `/code-review`

Dans n'importe quelle session Claude Code, exécuter `/code-review` pour analyser la diff actuelle.
- `--comment` : publier en commentaires PR en ligne
- `--fix` : appliquer les résultats à l'arborescence de travail
- `/code-review ultra --fix` : ultrareview profonde dans le cloud

## Dépannage

**Redéclencher** après échec : `@claude review once` (bouton Re-run GitHub ne fonctionne PAS).
**Plafond de dépenses atteint** : révisions reprennent au début de la prochaine période de facturation.
**Problèmes pas visibles en inline** : voir Check run Details et onglet Files changed.
