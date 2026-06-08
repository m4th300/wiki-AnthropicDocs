# Orchestrer des sous-agents à grande échelle avec des workflows dynamiques

> Les workflows dynamiques orchestrent de nombreux sous-agents à partir d'un script que Claude écrit et que vous pouvez réexécuter. Utilisez-les pour des audits de base de code, des migrations importantes et des recherches croisées.

Les workflows dynamiques sont en aperçu de recherche. Ils nécessitent Claude Code v2.1.154 ou une version ultérieure et sont disponibles sur tous les plans payants, avec l'accès API Anthropic, et sur Amazon Bedrock, Google Cloud Vertex AI et Microsoft Foundry. Sur Pro, activez-les depuis la ligne Dynamic workflows dans `/config`.

Un workflow dynamique est un script JavaScript qui orchestre des sous-agents à grande échelle. Claude écrit le script pour la tâche que vous décrivez, et un runtime l'exécute en arrière-plan pendant que votre session reste réactive.

Utilisez un workflow quand une tâche nécessite plus d'agents qu'une seule conversation peut coordonner, ou quand vous voulez que l'orchestration soit codifiée comme un script que vous pouvez lire et réexécuter. Exemples : un balayage de bugs à l'échelle de la base de code, une migration de 500 fichiers, une question de recherche nécessitant des sources croisées, ou un plan difficile valant la peine d'être rédigé sous plusieurs angles indépendants.

## Quand utiliser un workflow

Les sous-agents, compétences, équipes d'agents et workflows peuvent tous exécuter une tâche multi-étapes. La différence est qui détient le plan :

|  | Sous-agents | Compétences | Équipes d'agents | Workflows |
| :- | :---------- | :---------- | :--------------- | :-------- |
| Ce que c'est | Un travailleur que Claude génère | Instructions que Claude suit | Un agent principal supervisant des sessions pairs | Un script que le runtime exécute |
| Qui décide ce qui s'exécute ensuite | Claude, tour par tour | Claude, suivant l'invite | L'agent principal, tour par tour | Le script |
| Où vivent les résultats intermédiaires | Fenêtre de contexte de Claude | Fenêtre de contexte de Claude | Une liste de tâches partagée | Variables du script |
| Ce qui est répétable | La définition du travailleur | Les instructions | La définition de l'équipe | L'orchestration elle-même |
| Échelle | Quelques tâches déléguées par tour | Identique aux sous-agents | Une poignée de pairs de longue durée | Des dizaines à des centaines d'agents par exécution |
| Interruption | Redémarre le tour | Redémarre le tour | Les coéquipiers continuent à fonctionner | Repris dans la même session |

Un workflow déplace le plan dans du code. Avec les sous-agents, les compétences et les équipes d'agents, Claude est l'orchestrateur : il décide tour par tour quoi générer ou assigner, et chaque résultat atterrit dans une fenêtre de contexte. Un script de workflow détient lui-même la boucle, les branchements et les résultats intermédiaires, donc le contexte de Claude ne contient que la réponse finale.

## Exécuter un workflow groupé

La façon la plus rapide de voir un workflow en action est d'exécuter `/deep-research`, le workflow intégré que Claude Code inclut pour investiguer une question sur de nombreuses sources.

### Workflows groupés

| Commande | Ce qu'elle fait |
| :------- | :-------------- |
| `/deep-research <question>` | Déploie des recherches web sur une question sous plusieurs angles, récupère et croise les sources trouvées, vote sur chaque affirmation, et retourne un rapport cité |

### Étapes pour exécuter `/deep-research`

1. Exécutez `/deep-research` avec une question à investiguer :
   ```text
   /deep-research What changed in the Node.js permission model between v20 and v22?
   ```

2. Claude Code demande si le workflow doit être autorisé. Sélectionnez **Yes** pour continuer.

3. L'exécution démarre en arrière-plan. Exécutez `/workflows` pour voir la progression :
   ```text
   /workflows
   ```

4. Quand l'exécution se termine, le rapport atterrit dans votre session avec les sources citées.

### Regarder l'exécution

| Touche | Action |
| :----- | :----- |
| `↑` / `↓` | Sélectionner une phase ou un agent |
| `Enter` ou `→` | Explorer la phase sélectionnée, puis un agent |
| `Esc` | Revenir d'un niveau |
| `j` / `k` | Faire défiler dans les détails de l'agent |
| `p` | Mettre en pause ou reprendre l'exécution |
| `x` | Arrêter l'agent sélectionné, ou tout le workflow |
| `r` | Redémarrer l'agent sélectionné en cours d'exécution |
| `s` | Sauvegarder le script de l'exécution comme commande |

## Demander à Claude d'écrire un workflow

Vous pouvez demander à Claude d'écrire un workflow de deux façons :

### Demander un workflow dans votre invite

Pour exécuter une seule tâche comme un workflow sans changer le niveau d'effort de la session, incluez le mot-clé `ultracode` dans votre invite :

```text
ultracode: audit every API endpoint under src/routes/ for missing auth checks
```

Claude Code met en évidence le mot-clé dans votre saisie et Claude écrit un script de workflow pour la tâche au lieu de la traiter tour par tour.

### Laisser Claude décider avec ultracode

Ultracode est un paramètre Claude Code qui combine un `xhigh` niveau de raisonnement avec une orchestration automatique de workflow. Avec lui activé, Claude planifie un workflow pour chaque tâche substantielle au lieu d'attendre que vous le demandiez :

```text
/effort ultracode
```

Ultracode dure pour la session courante et se réinitialise quand vous en démarrez une nouvelle. Revenez avec `/effort high` quand vous retournez au travail de routine.

### Approuver le plan avant qu'il s'exécute

Dans le CLI, l'invite par exécution montre les phases planifiées et ces options :

* **Yes, run it** : démarrer l'exécution
* **Yes, and don't ask again for `<name>` in `<path>`** : démarrer, et ignorer cette invite pour ce workflow dans ce projet à l'avenir
* **View raw script** : lire le script avant de décider
* **No** : annuler

Le comportement selon le mode de permission :

| Mode de permission | Quand vous êtes invité |
| :----------------- | :--------------------- |
| Par défaut, accepter les modifications | Chaque exécution, sauf si vous avez sélectionné **Yes, and don't ask again** |
| Auto | Première exécution uniquement |
| Contournement des permissions, `claude -p`, Agent SDK | Jamais |

### Sauvegarder le workflow pour réutilisation

Quand Claude écrit un workflow pour une tâche que vous allez répéter, vous pouvez sauvegarder le script de cette exécution comme commande. Exécutez `/workflows`, sélectionnez l'exécution à conserver, et appuyez sur `s`.

Deux emplacements de sauvegarde :

* `.claude/workflows/` dans votre projet : partagé avec tous ceux qui clonent le dépôt
* `~/.claude/workflows/` dans votre répertoire home : disponible dans chaque projet, visible uniquement par vous

Le workflow s'exécute comme `/<name>` dans les sessions futures.

### Passer des entrées à un workflow sauvegardé

Un workflow sauvegardé peut accepter des entrées via le paramètre `args`. Le script le lit comme un global nommé `args` :

```text
> Run /triage-issues on issues 1024, 1025, and 1030
```

## Comment fonctionne un workflow

Le runtime de workflow exécute le script dans un environnement isolé, séparé de votre conversation. Les résultats intermédiaires restent dans les variables du script au lieu d'atterrir dans le contexte de Claude.

### Comportement et limites

| Contrainte | Pourquoi |
| :--------- | :------- |
| Pas de saisie utilisateur en cours d'exécution | Seules les invites de permission d'agent peuvent mettre une exécution en pause |
| Pas d'accès direct au système de fichiers ou au shell depuis le workflow lui-même | Les agents lisent, écrivent et exécutent des commandes. Le script coordonne les agents |
| Jusqu'à 16 agents simultanés, moins sur les machines avec des cœurs CPU limités | Limite l'utilisation des ressources locales |
| 1 000 agents au total par exécution | Empêche les boucles incontrôlées |

## Gérer les exécutions

### Reprendre après une pause

Si vous arrêtez une exécution, vous pouvez la reprendre : les agents déjà terminés retournent leurs résultats mis en cache, et le reste s'exécute en direct. Reprenez depuis `/workflows` en sélectionnant l'exécution et en appuyant sur `p`.

La reprise fonctionne dans la même session Claude Code. Si vous quittez Claude Code pendant qu'un workflow est en cours d'exécution, la prochaine session démarre le workflow à zéro.

### Coût

Un workflow génère de nombreux agents, donc une seule exécution peut utiliser significativement plus de jetons que de travailler sur la même tâche en conversation. Les exécutions comptent vers les limites d'utilisation et de débit de votre plan comme n'importe quelle autre session.

Pour évaluer la dépense avant de s'engager dans une grande tâche, exécutez d'abord le workflow sur une petite tranche : un répertoire au lieu du dépôt entier.

### Désactiver les workflows

Pour vous :

* Désactivez Dynamic workflows dans `/config`
* Définissez `"disableWorkflows": true` dans `~/.claude/settings.json`
* Définissez `CLAUDE_CODE_DISABLE_WORKFLOWS=1`

Pour toute votre organisation, définissez `"disableWorkflows": true` dans les paramètres gérés, ou utilisez le bouton sur la page des paramètres d'administration Claude Code.

## Ressources connexes

* Exécuter des agents en parallèle : comparer les sous-agents, la vue des agents, les équipes d'agents et les workflows
* Créer des sous-agents personnalisés : la primitive de travail que les workflows orchestrent
* Gérer les coûts : comment les exécutions multi-agents comptent vers les limites d'utilisation
