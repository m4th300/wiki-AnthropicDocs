# Meilleures pratiques pour Claude Code

Source: https://code.claude.com/docs/fr/best-practices

## Contrainte fondamentale : la fenêtre de contexte

Les performances des LLM se dégradent à mesure que la context window se remplit. C'est la contrainte centrale autour de laquelle toutes les meilleures pratiques s'organisent.

## 1. Donner à Claude un moyen de vérifier son travail

La différence entre une session que vous regardez et une que vous pouvez laisser tourner : une vérification exécutable.

| Stratégie | Avant | Après |
|-----------|-------|-------|
| Critères de vérification | "implémenter validateEmail" | "écrire validateEmail. Tests: user@example.com→true, invalid→false. Exécuter les tests après" |
| Vérification visuelle | "rendre le dashboard plus beau" | "[coller screenshot] comparer avec cette conception, lister les différences et les corriger" |
| Cause profonde | "la compilation échoue" | "[coller l'erreur]. La corriger et vérifier que la compilation réussit. Traiter la cause profonde" |

Niveaux d'escalade de la vérification :
- En un seul message : demander à Claude d'itérer dans le même message
- Sur une session : `/goal` avec une condition de vérification
- Porte déterministe : hook Stop qui exécute la vérification
- Deuxième avis : subagent de vérification

## 2. Explorer d'abord, planifier, puis coder

Workflow recommandé (4 phases) :
1. **Plan mode** : Claude lit les fichiers sans modifier (`Shift+Tab`)
2. **Planifier** : demander un plan détaillé ; `Ctrl+G` pour éditer dans votre éditeur
3. **Implémenter** : quitter plan mode → Claude code en vérifiant son plan
4. **Valider** : commit avec message descriptif + créer PR

Quand sauter la planification : tâches simples où la portée est claire et la correction est petite.

## 3. Fournir un contexte spécifique

| Stratégie | Avant | Après |
|-----------|-------|-------|
| Délimiter la tâche | "ajouter des tests pour foo.py" | "écrire un test pour foo.py couvrant le cas où l'utilisateur est déconnecté. Pas de mocks." |
| Pointer vers les sources | "pourquoi l'API est-elle bizarre ?" | "parcourir l'historique git d'ExecutionFactory et résumer comment son API en est venue à être" |
| Référencer les patterns | "ajouter un widget calendrier" | "regarder HotDogWidget.php comme exemple, suivre ce pattern pour un nouveau widget calendrier" |

Fournir du contenu riche :
- `@fichier` pour référencer directement
- Coller les images (Ctrl+V, pas Cmd+V)
- Donner des URLs de documentation
- Piper des données : `cat error.log | claude`

## 4. Configurer l'environnement

### CLAUDE.md efficace

✅ Inclure : commandes Bash inconnues, règles de style qui diffèrent des valeurs par défaut, conventions de test, étiquette repo, décisions architecturales spécifiques
❌ Exclure : ce que Claude peut déduire du code, conventions de langage standard, docs API détaillées, choses qui changent souvent

Garder sous 200 lignes. Si Claude ignore des règles → fichier trop long.
```markdown
# Code style
- Use ES modules (import/export) syntax, not CommonJS (require)
- Destructure imports when possible

# Workflow
- Always typecheck after a series of code changes
- Prefer running single tests, not the whole test suite
```

### Outils CLI recommandés
- `gh` pour GitHub (créer issues, PRs, lire commentaires)
- `aws`, `gcloud` pour cloud providers
- Claude apprend les nouveaux outils CLI avec `--help`

### Permissions
3 approches pour réduire les interruptions :
- Mode auto (classificateur)
- Listes blanches de permissions (`/permissions`)
- Sandboxing (isolation OS)

## 5. Communiquer efficacement

### Laisser Claude interviewer
```text
I want to build [brief description]. Interview me in detail using AskUserQuestion.
Ask about technical implementation, UI/UX, edge cases, concerns, tradeoffs.
Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```
→ Démarrer une nouvelle session propre pour l'implémentation

## 6. Gérer la session

### Corriger la trajectoire tôt
- `Esc` : arrêter immédiatement (contexte préservé)
- `Esc + Esc` ou `/rewind` : restaurer à un checkpoint précédent
- Après 2 corrections échouées : `/clear` + meilleure invite

### Gérer le contexte agressivement
- `/clear` entre tâches non liées
- `/compact <instructions>` pour compacter avec focus
- `Esc + Esc` → Summarize from here / Summarize up to here
- `/btw` pour questions rapides qui ne doivent pas rester en contexte
- Subagents pour l'investigation (explorent dans leur propre contexte)

### Checkpoints et reprise
- Checkpoints créés automatiquement à chaque prompt
- `/rewind` : restaurer conversation + code à n'importe quel checkpoint
- `claude --continue` / `claude --resume` pour reprendre entre sessions
- `/rename` pour nommer les sessions importantes

## 7. Automatiser et mettre à l'échelle

### Mode non-interactif
```bash
claude -p "Explain what this project does"
claude -p "List all API endpoints" --output-format json
cat error.log | claude -p "find anomalies"
```

### Sessions parallèles
- Worktrees : sessions CLI isolées
- Desktop : sessions parallèles visuelles
- Claude Code sur le web : VMs isolées
- Équipes d'agents : coordination automatisée

### Pattern Writer/Reviewer
Session A (Writer) → Session B (Reviewer avec contexte frais) → Session A (Corrections)

### Fan-out sur les fichiers
```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from React to Vue. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

## 8. Anti-patterns à éviter

| Anti-pattern | Correction |
|-------------|------------|
| Session fourre-tout | `/clear` entre tâches non liées |
| Corriger encore et encore | Après 2 corrections → `/clear` + meilleure invite |
| CLAUDE.md sur-spécifié | Élaguer ; si déjà fait → supprimer ou convertir en hook |
| Confiance sans vérification | Toujours fournir une vérification exécutable |
| Exploration infinie | Délimiter les enquêtes ou utiliser des subagents |
