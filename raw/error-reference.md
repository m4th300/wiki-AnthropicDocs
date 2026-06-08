# Référence des erreurs

> Recherchez les messages d'erreur d'exécution de Claude Code avec ce que chacun signifie et comment le corriger.

Cette page liste les erreurs d'exécution que Claude Code affiche et comment récupérer de chacune, plus ce qu'il faut vérifier quand les réponses semblent incorrectes sans erreur. Pour les erreurs d'installation comme `command not found` ou les échecs TLS lors de la configuration, voir Résolution des problèmes d'installation et de connexion.

Ces erreurs et commandes de récupération s'appliquent au CLI, à l'application Desktop et à Claude Code sur le web, puisque tous trois encapsulent le même CLI Claude Code.

Claude Code appelle l'API Claude pour les réponses du modèle, donc la plupart des erreurs d'exécution correspondent à un code d'erreur API sous-jacent.

## Trouver votre erreur

| Message | Section |
| :------ | :------- |
| `API Error: 500 Internal server error` | Erreurs de serveur |
| `API Error: Repeated 529 Overloaded errors` | Erreurs de serveur |
| `Request timed out` | Erreurs de serveur, ou réseau si le message mentionne votre connexion internet |
| `<model> is temporarily unavailable, so auto mode cannot determine the safety of...` | Erreurs de serveur |
| `Auto mode could not evaluate this action and is blocking it for safety` | Erreurs de serveur |
| `Auto mode classifier transcript exceeded context window` | Erreurs de serveur |
| `You've hit your session limit` / `You've hit your weekly limit` | Limites d'utilisation |
| `Server is temporarily limiting requests` | Limites d'utilisation |
| `Request rejected (429)` | Limites d'utilisation |
| `Credit balance is too low` | Limites d'utilisation |
| `Not logged in · Please run /login` | Authentification |
| `Invalid API key` | Authentification |
| `This organization has been disabled` | Authentification |
| `Your organization has disabled Claude subscription access` | Authentification |
| `Routines are disabled by your organization's policy` | Authentification |
| `OAuth token revoked` / `OAuth token has expired` | Authentification |
| `does not meet scope requirement user:profile` | Authentification |
| `Unable to connect to API` | Réseau |
| `SSL certificate verification failed` | Réseau |
| `403` avec `x-deny-reason: host_not_allowed` dans une session cloud ou routine | Réseau |
| `Prompt is too long` | Erreurs de requête |
| `Error during compaction: Conversation too long` | Erreurs de requête |
| `Request too large` | Erreurs de requête |
| `Image was too large` | Erreurs de requête |
| `Unable to resize image` | Erreurs de requête |
| `PDF too large` / `PDF is password protected` | Erreurs de requête |
| `Extra inputs are not permitted` | Erreurs de requête |
| `There's an issue with the selected model` | Erreurs de requête |
| `Claude Opus is not available with the Claude Pro plan` | Erreurs de requête |
| `thinking.type.enabled is not supported for this model` | Erreurs de requête |
| `max_tokens must be greater than thinking.budget_tokens` | Erreurs de requête |
| `API Error: 400 due to tool use concurrency issues` | Erreurs de requête |
| Refus de la politique d'utilisation | Erreurs de requête |
| Les réponses semblent de moins bonne qualité qu'habituellement | Qualité des réponses |

## Réessais automatiques

Claude Code réessaie les échecs transitoires avant de vous afficher une erreur. Les erreurs de serveur, les réponses surchargées, les délais d'expiration des requêtes, les throttles 429 temporaires et les connexions interrompues sont tous réessayés jusqu'à 10 fois avec un backoff exponentiel.

Vous pouvez ajuster le comportement avec deux variables d'environnement :

| Variable | Défaut | Effet |
| :------- | :----- | :---- |
| `CLAUDE_CODE_MAX_RETRIES` | 10 | Nombre de tentatives de réessai |
| `API_TIMEOUT_MS` | 600000 | Délai d'expiration par requête en millisecondes |

## Erreurs de serveur

Ces erreurs viennent du fournisseur d'inférence plutôt que de votre compte ou requête.

### API Error: 500 Internal server error

```text
API Error: 500 Internal server error. This is a server-side issue, usually temporary — try again in a moment. If it persists, check https://status.claude.com.
```

**Que faire :**
* Vérifiez [status.claude.com](https://status.claude.com) pour les incidents actifs
* Attendez une minute, puis envoyez à nouveau votre message
* Si l'erreur persiste sans incident publié, exécutez `/feedback`

### API Error: Repeated 529 Overloaded errors

```text
API Error: Repeated 529 Overloaded errors. The API is at capacity — this is usually temporary. Try again in a moment.
```

Un 529 n'est pas votre limite d'utilisation et ne compte pas vers votre quota.

**Que faire :**
* Vérifiez [status.claude.com](https://status.claude.com) pour les avis de capacité
* Réessayez dans quelques minutes
* Exécutez `/model` et passez à un modèle différent pour continuer à travailler

### Request timed out

```text
Request timed out
```

**Que faire :**
* Réessayez la requête
* Pour les tâches de longue durée, décomposez le travail en invites plus petites
* Si un réseau lent ou un proxy est la cause, augmentez `API_TIMEOUT_MS`

### Le mode auto ne peut pas déterminer la sécurité d'une action

Quand le modèle classificateur est surchargé :

```text
<model> is temporarily unavailable, so auto mode cannot determine the safety of <tool> right now.
```

Quand le classificateur a retourné une réponse non analysable :

```text
Auto mode could not evaluate this action and is blocking it for safety — run with --debug for details
```

Quand la conversation a dépassé la fenêtre de contexte du classificateur :

```text
Auto mode classifier transcript exceeded context window — falling back to manual approval (try /compact to reduce conversation size)
```

**Que faire :** Réessayez l'action ou exécutez `/compact` pour réduire la taille de la conversation.

## Limites d'utilisation

### You've hit your session limit

```text
You've hit your session limit · resets 3:45pm
You've hit your weekly limit · resets Mon 12:00am
You've hit your Opus limit · resets 3:45pm
```

**Que faire :**
* Attendez l'heure de réinitialisation indiquée dans l'erreur
* Exécutez `/usage` pour voir les limites de votre plan
* Exécutez `/usage-credits` pour acheter de l'utilisation supplémentaire sur Pro et Max

### Server is temporarily limiting requests

```text
API Error: Server is temporarily limiting requests (not your usage limit)
```

**Que faire :** Attendez brièvement et réessayez.

### Request rejected (429)

```text
API Error: Request rejected (429) · this may be a temporary capacity issue.
```

**Que faire :**
* Exécutez `/status` et confirmez que la clé d'identification active est celle que vous attendez
* Vérifiez votre console fournisseur pour les limites actives
* Réduisez la concurrence : abaissez `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY`

### Credit balance is too low

```text
Credit balance is too low
```

**Que faire :**
* Ajoutez des crédits sur platform.claude.com/settings/billing
* Passez à l'authentification par abonnement avec `/login`

## Erreurs d'authentification

### Not logged in

```text
Not logged in · Please run /login
```

**Que faire :**
* Exécutez `/login` pour vous authentifier
* Confirmez que `ANTHROPIC_API_KEY` est défini et exporté dans le shell

### Invalid API key

```text
Invalid API key · Fix external API key
```

**Que faire :**
* Vérifiez les fautes de frappe et confirmez que la clé n'a pas été révoquée
* Exécutez `env | grep ANTHROPIC` dans le même shell
* Exécutez `/status` pour confirmer quelle source de clé d'identification Claude Code utilise réellement

### This organization has been disabled

```text
Your ANTHROPIC_API_KEY belongs to a disabled organization · Unset the environment variable to use your other credentials
```

**Que faire :** Désactivez `ANTHROPIC_API_KEY` dans le shell actuel et supprimez-le de votre profil shell.

### Your organization has disabled Claude subscription access

```text
Your organization has disabled Claude subscription access for Claude Code · Use an Anthropic API key instead, or ask your admin to enable access
```

**Que faire :**
* Demandez à votre administrateur d'activer l'accès Claude Code pour votre organisation
* Authentifiez-vous avec une clé API Console

### Routines are disabled by your organization's policy

```text
Routines are disabled by your organization's policy.
```

**Que faire :** Demandez à votre administrateur d'activer le bouton **Routines** sur claude.ai/admin-settings/claude-code.

### OAuth token revoked or expired

```text
OAuth token revoked · Please run /login
OAuth token has expired · Please run /login
```

**Que faire :**
* Exécutez `/login` pour vous reconnecter
* Si l'erreur revient, exécutez `/logout` d'abord, puis `/login`

### OAuth scope requirement

```text
OAuth token does not meet scope requirement: user:profile
```

**Que faire :** Exécutez `/login` pour créer un nouveau jeton avec les portées actuelles.

## Erreurs réseau et de connexion

### Unable to connect to API

```text
Unable to connect to API. Check your internet connection
Unable to connect to API (ECONNREFUSED)
Unable to connect to API (ECONNRESET)
Unable to connect to API (ETIMEDOUT)
```

**Que faire :**
* Confirmez que vous pouvez atteindre l'hôte API : `curl -I https://api.anthropic.com`
* Si vous êtes derrière un proxy d'entreprise, définissez `HTTPS_PROXY` avant de lancer Claude Code
* Si vous routez via une passerelle LLM, définissez `ANTHROPIC_BASE_URL`

### SSL certificate errors

```text
Unable to connect to API: SSL certificate verification failed.
```

**Que faire :**
* Exportez le bundle CA de votre organisation et pointez Claude Code dessus avec `NODE_EXTRA_CA_CERTS=/path/to/ca-bundle.pem`
* Ne définissez pas `NODE_TLS_REJECT_UNAUTHORIZED=0`

### Host not allowed in a cloud session

```text
HTTP 403
x-deny-reason: host_not_allowed
```

**Que faire :** Ouvrez la routine pour modification, sélectionnez l'icône cloud, et changez l'accès réseau de **Trusted** à **Custom**, puis ajoutez le domaine bloqué aux **Allowed domains**.

## Erreurs de requête

### Prompt is too long

```text
Prompt is too long
```

**Que faire :**
* Exécutez `/compact` pour résumer les tours précédents
* Exécutez `/context` pour voir ce qui consomme la fenêtre
* Désactivez les serveurs MCP inutilisés avec `/mcp disable <name>`

### Error during compaction: Conversation too long

```text
Error during compaction: Conversation too long. Press esc twice to go up a few messages and try again.
```

**Que faire :**
* Appuyez deux fois sur Esc pour ouvrir la liste de messages et revenir plusieurs tours
* Si cela ne libère pas assez d'espace, exécutez `/clear`

### Request too large

```text
Request too large (max 30 MB). Double press esc to go back and remove or shrink the attached content.
```

**Que faire :**
* Appuyez deux fois sur Esc et revenez au-delà du tour qui a ajouté le contenu trop grand
* Référencez les grands fichiers par chemin au lieu de coller leur contenu

### Image was too large

```text
Image was too large. Double press esc to go back and try again with a smaller image.
```

**Que faire :**
* Appuyez deux fois sur Esc et revenez au-delà du tour où l'image a été ajoutée
* Redimensionnez l'image avant de la coller

### PDF errors

```text
PDF too large (max 100 pages, 32 MB). Try splitting it or extracting text first.
PDF is password protected. Try removing protection or extracting text first.
```

**Que faire :**
* Pour les PDFs trop grands, demandez à Claude de lire une plage de pages avec l'outil Read
* Pour les PDFs protégés, supprimez le mot de passe ou ré-exportez le fichier

### Extra inputs are not permitted

```text
API Error: 400 ... Extra inputs are not permitted ... context_management
```

Un proxy ou une passerelle LLM entre Claude Code et l'API a supprimé l'en-tête de requête `anthropic-beta`.

**Que faire :**
* Configurez votre passerelle pour transmettre l'en-tête `anthropic-beta`
* Ou définissez `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`

### There's an issue with the selected model

```text
There's an issue with the selected model (claude-...). It may not exist or you may not have access to it.
```

**Que faire :**
* Exécutez `/model` pour choisir parmi les modèles disponibles pour votre compte
* Utilisez un alias comme `sonnet` ou `opus` au lieu d'un ID complet versionné

### Claude Opus is not available with the Claude Pro plan

```text
Claude Opus is not available with the Claude Pro plan · Select a different model in /model
```

**Que faire :**
* Exécutez `/model` et sélectionnez un modèle inclus dans votre plan
* Si vous avez récemment mis à niveau, exécutez `/logout` puis `/login`

### thinking.type.enabled is not supported for this model

```text
API Error: 400 ... "thinking.type.enabled" is not supported for this model.
```

**Que faire :**
* Exécutez `claude update` et redémarrez Claude Code
* Si vous ne pouvez pas mettre à niveau, exécutez `/model` et sélectionnez Opus 4.6 ou Sonnet

### Thinking budget exceeds output limit

```text
API Error: 400 ... max_tokens must be greater than thinking.budget_tokens
```

**Que faire :**
* Abaissez `MAX_THINKING_TOKENS`, ou augmentez `CLAUDE_CODE_MAX_OUTPUT_TOKENS` au-dessus du budget de réflexion

### Tool use or thinking block mismatch

```text
API Error: 400 due to tool use concurrency issues. Run /rewind to recover the conversation.
API Error: 400 ... unexpected `tool_use_id` found in `tool_result` blocks
API Error: 400 ... thinking blocks ... cannot be modified
```

**Que faire :**
* Exécutez `claude update` en premier si vous utilisez Opus 4.7 ou Opus 4.8
* Exécutez `/rewind`, ou appuyez deux fois sur Esc, pour revenir à un point de contrôle avant le tour corrompu

### Usage Policy refusal

```text
API Error: Claude Code is unable to respond to this request, which appears to violate our Usage Policy.
```

**Que faire :**
* Appuyez deux fois sur Esc ou exécutez `/rewind` pour revenir à un point de contrôle avant le tour déclencheur
* Si vous ne pouvez pas identifier quel tour l'a causé, exécutez `/clear`

## Les réponses semblent de moins bonne qualité qu'habituellement

Vérifiez ces éléments en premier :

* **Sélection du modèle** : exécutez `/model` pour confirmer que vous êtes sur le modèle que vous attendez
* **Niveau d'effort** : exécutez `/effort` pour vérifier le niveau de raisonnement actuel
* **Pression de contexte** : exécutez `/context` pour voir à quel point la fenêtre est pleine
* **Instructions obsolètes** : les grands fichiers CLAUDE.md obsolètes et les définitions d'outils MCP consomment du contexte

Quand une réponse tourne mal, le retour en arrière fonctionne généralement mieux que de répondre avec des corrections. Appuyez deux fois sur Esc ou exécutez `/rewind` pour revenir au tour précédant la mauvaise réponse.

## Signaler une erreur

Si une erreur n'est pas listée ici ou si le correctif suggéré n'aide pas :

* Exécutez `/feedback` à l'intérieur de Claude Code pour envoyer la transcription et une description à Anthropic
* Exécutez `/doctor` pour vérifier les problèmes de configuration locale
* Vérifiez [status.claude.com](https://status.claude.com) pour les incidents actifs
* Recherchez les problèmes existants sur GitHub
