> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt

# Bibliothèque de prompts

> Copiez-collez des prompts pour Claude Code, étiquetés par tâche et rôle.

Ceci est une bibliothèque de prompts à copier dans Claude Code. Utilisez-la pour explorer des façons de travailler que vous n'avez pas essayées, ou quand vous ne savez pas par où commencer.

Les prompts sont collectés à partir de divers guides Anthropic : Flux de travail courants, Bonnes pratiques, et Comment les équipes Anthropic utilisent Claude Code.

## Phases SDLC et catégories

### Découvrir

**Intégration (Onboard)**
- S'orienter dans un nouveau référentiel : `give me an overview of this codebase: architecture, key directories, and how the pieces connect`

**Comprendre (Understand)**
- Expliquer du code non familier : `explain what {path} does and how data flows through it. write it up as {format}`
- Trouver où quelque chose se produit : `where do we {behavior}?`
- Vérifier ce qui se casse avant de supprimer : `what would break if I deleted {target}?`
- Tracer l'évolution du code : `look through the commit history of {path} and summarize how it evolved and why`
- Délimiter une modification (PM/Design) : `which files would I need to touch to {change}?`
- Poser une question produit à la base de code (PM) : `I am a {role}. walk me through what happens when a user {action}, from the UI down to the result`

### Concevoir (Design)

**Planifier (Plan)**
- Planifier une modification multi-fichiers : `plan how to refactor the {target} to {goal}. list the files you would change, but don't edit anything yet`
- Rédiger une spécification par entrevue (PM) : `I want to build {feature}. interview me about implementation, UX, edge cases, and tradeoffs until we have covered everything, then write the spec to SPEC.md`
- Transformer une réunion en tickets (PM) : `read {input} and write up the action items, then create a {tracker} ticket for each with acceptance criteria`
- Cartographier les cas limites (Design/PM) : `list the error states, empty states, and edge cases for {feature} that the design needs to cover`

**Prototype**
- Transformer une maquette en prototype (Design/PM/Marketing) : `here is a mockup. build a working prototype I can click through, matching the layout and states shown` [coller/glisser image]
- Implémenter à partir d'une capture d'écran et auto-vérifier (Design) : `implement this design, then take a screenshot of the result, compare it to the original, and fix any differences`

### Construire (Build)

**Implémenter**
- Suivre un modèle existant : `look at how {example} is implemented to understand the pattern, then build {new} the same way`
- Générer de la documentation : `find {scope} without {format} comments and add them, matching the style already used in the file`
- Ajouter une petite fonctionnalité : `add a {endpoint} endpoint that returns {payload}`
- Construire un petit outil interne (PM/Design/Marketing/Docs) : `create a {tool} using HTML, CSS, and vanilla JavaScript, then open it in my browser`
- Traiter un problème de bout en bout : `read issue #{issue}, implement the fix, and run the tests` [nécessite gh CLI]
- Trouver et mettre à jour le texte : `find every place we say "{copy}" or a close variant, show me each one in context, then update them all to "{new}". leave tests and the changelog alone`
- Rédiger un document à partir d'exemples passés (Docs/Marketing/PM) : `read the {examples} in {folder} to learn the structure and voice, then draft a new one for {topic}`

**Tester**
- Écrire des tests et corriger : `write tests for {path}, run them, and fix any failures`
- TDD : `write tests for {feature} first, then implement it until they pass`
- Combler les lacunes de couverture : `read {report} and add tests for the lowest-covered files until each is above {target}%`

**Refactoriser**
- Migrer un modèle : `migrate everything from {from} to {to}: identify every place that needs to change, then make the changes`
- Porter le code : `port {source} to {target}, keeping the same {keep}`
- Optimiser par rapport à une cible mesurable (Data) : `optimize {target} to bring {metric} from {current} down to under {goal}`
- Corriger un bug visuel (Design) : `the {element} extends {amount} beyond the {container} on {viewport}. fix it.`

**Examiner (Review)**
- Examiner avant de valider : `review my uncommitted changes and flag anything that looks risky before I commit`
- Examiner une PR : `review PR #{pr} and summarize what changed, then list any concerns` [nécessite gh CLI]
- Examiner les modifications Terraform (Security/Ops) : `here is my Terraform plan output. what is this going to do, and is anything here going to cause problems?` [coller plan]
- Examen de sécurité avec sous-agent (Security) : `use a subagent to review {path} for security issues and report what it finds`
- Examiner le contenu avant envoi (Marketing/Docs) : `review {file} for {concerns} and list anything I should fix before it goes to {reviewer}`

**Diriger (Steer)**
- Corriger une mauvaise approche : `that is not right: {feedback}. try a different approach`
- Réduire la portée : `that is too much. keep only the changes to {scope} and undo your other edits`
- Transformer une correction en règle : `you keep {mistake}. add a rule to CLAUDE.md so this stops happening`

### Livrer (Ship)

**Git**
- Résoudre les conflits de fusion : `resolve the merge conflicts in this branch and explain what you kept from each side`
- Valider avec un message généré : `commit these changes with a message that summarizes what I did`
- Ouvrir une PR depuis un ticket : `find the {tracker} ticket about {topic} and open a PR that implements it` [nécessite tracker MCP]

**Sortie (Release)**
- Rédiger les notes de sortie : `compare {from} to {to} and draft release notes grouped by feature, fix, and breaking change`
- Écrire un flux de travail CI (Ops) : `write a GitHub Actions workflow that {steps} on every push to {branch}`

### Exploiter (Operate)

**Déboguer (Debug)**
- Trouver et corriger un test défaillant : `the {test} test is failing, find out why and fix it`
- Enquêter sur une erreur signalée (Ops) : `users are seeing {symptom} on {where}. investigate and tell me what is going on`
- Corriger une erreur de construction (Ops) : `here is a build error. fix the root cause and verify the build succeeds` [coller erreur]

**Incident**
- Enquêter sur un incident de production (Ops/Security) : `{symptom}. check the logs, recent deploys, and config changes, then tell me the most likely cause`
- Diagnostiquer depuis une console (Ops/Data) : `here is a screenshot of {console}. walk me through why {resource} is failing and give me the exact commands to fix it` [coller capture d'écran]
- Interroger les journaux (Security/Ops/Data) : `show me all {events} for {scope} over {timeframe}. write the query, run it, and tell me what stands out` [nécessite db MCP]

**Données (Data)**
- Analyser un fichier de données (Data/PM/Marketing) : `read {file}, summarize the key patterns, and write the results to {output}` [glisser CSV]
- Générer des variations depuis des données de performance (Marketing/Data) : `read {file}, find the underperforming {items}, and generate {n} new variations that stay under {limit} characters`

**Automatiser (Automate)**
- Transformer une tâche récurrente en compétence : `create a /{name} skill for this project that {steps}`
- Ajouter un hook : `write a hook that {action} after every {event}`
- Connecter un outil avec MCP : `set up the {server} MCP server so you can read my {data} directly`
- Capturer ce qu'il faut retenir (PM/Docs) : `summarize what we did this session and suggest what to add to CLAUDE.md`

## Ce qui rend ces prompts efficaces

**Décrivez le résultat, pas les étapes.**
```
ajouter la limitation de débit à l'API publique et s'assurer que les tests existants passent toujours
```

**Donnez-lui un moyen de vérifier son propre travail.**
```
écrire la migration, l'exécuter contre la base de données de développement et confirmer que le schéma correspond
```

**Pointez une référence.**
```
ajouter une page de paramètres qui suit la même mise en page que la page de profil
```

**Énoncez la cible mesurable.**
```
obtenir la taille du bundle sous 200 KB et montrez-moi ce que vous avez supprimé
```

**Donnez-lui l'artefact.**
```
pourquoi la construction échoue-t-elle ? @build.log
```

**Dites comment vous voulez la réponse.**
```
expliquer comment la logique de nouvelle tentative de paiement fonctionne en tant que page HTML avec un diagramme, puis l'ouvrir dans mon navigateur
```

## D'où viennent ces prompts

* [Flux de travail courants](/fr/common-workflows)
* [Bonnes pratiques](/fr/best-practices)
* [Comment les équipes Anthropic utilisent Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code) (avec approfondissements juridique, marketing, cybersécurité)
* [Guide de codage agentique à l'échelle](https://resources.anthropic.com/hubfs/Scaling%20agentic%20coding%20across%20your%20organization.pdf)
* Cours gratuit [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action) sur Anthropic Academy

## Ressources connexes

* [Compétences (skills)](/fr/skills) : enregistrer les prompts récurrents comme commandes
* [Mémoire (CLAUDE.md)](/fr/memory) : persister les conventions
* [Mode plan](/fr/permission-modes#analyze-before-you-edit-with-plan-mode)
* [Administration](/fr/admin-setup)
* [Coûts et utilisation](/fr/costs)
