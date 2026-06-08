# Choisir un environnement sandbox

> Comparez les options de sandbox Claude Code : l'outil Bash sandboxé intégré, le runtime sandbox, les dev containers, Docker et les machines virtuelles. Choisissez l'isolation appropriée pour votre modèle de menace.

L'isolation de Claude Code limite ce qu'une session peut lire, écrire et atteindre sur le réseau. Cela importe surtout lorsque vous laissez Claude travailler avec moins d'invites de permission, l'exécutez sans surveillance ou le pointez vers du code en lequel vous n'avez pas entièrement confiance.

Claude Code peut s'exécuter dans plusieurs types d'environnements isolés, allant d'un sandbox léger par commande à une machine virtuelle entièrement séparée.

> Pour le modèle de sécurité plus large, voir [Sécurité](/fr/security). Pour les déploiements Agent SDK, voir [Déploiement sécurisé](/fr/agent-sdk/secure-deployment).

## Comparer les approches de sandboxing

Les deux premières approches s'exécutent sur le système d'exploitation hôte sans conteneurs. Les autres placent Claude Code à l'intérieur d'un conteneur ou d'une machine virtuelle.

| Approche                           | Ce qui est isolé                                                                                     | Nécessite Docker | Effort de configuration                          |
| :--------------------------------- | :--------------------------------------------------------------------------------------------------- | :--------------- | :----------------------------------------------- |
| Outil Bash sandboxé                | Commandes Bash et leurs processus enfants                                                            | Non              | Minimal sur macOS ; faible sur Linux et WSL2     |
| Runtime sandbox                    | L'ensemble du processus Claude Code, y compris les outils de fichiers, les serveurs MCP et les hooks | Non              | Faible                                           |
| Dev container                      | Environnement de développement complet                                                               | Oui              | Moyen                                            |
| Conteneur personnalisé             | Environnement de développement complet                                                               | Oui              | Moyen à élevé                                    |
| Machine virtuelle                  | Système d'exploitation complet                                                                       | Non              | Élevé                                            |
| Claude Code sur le web             | Système d'exploitation complet, hébergé par Anthropic                                                | Non              | Aucun ; nécessite un abonnement Claude et GitHub |

L'outil Bash sandboxé est intégré à Claude Code et restreint uniquement les commandes Bash. Les outils de fichiers intégrés, les serveurs MCP et les hooks s'exécutent toujours directement sur votre hôte. Chaque autre approche place l'ensemble du processus Claude Code à l'intérieur de la limite d'isolation.

> **Avertissement** : L'isolation sandbox réduit l'impact d'une violation, mais ne l'élimine pas. Toute approche qui permet l'egress réseau peut toujours fuir les données que l'agent peut lire, et toute approche qui monte votre répertoire de projet en écriture peut toujours modifier ce code.
>
> L'isolation ne change pas non plus ce qui est envoyé au modèle. Vos invites et les fichiers que Claude lit sont transmis à l'API Anthropic ou à votre fournisseur configuré avec ou sans sandbox.

## Choisir une approche

| Vous voulez                                                                                       | Commencez par                                                                     |
| :------------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------- |
| Réduire les invites de permission pendant le travail quotidien                                    | L'outil Bash sandboxé, activé avec `/sandbox`                                     |
| Laisser Claude travailler sans surveillance avec `--dangerously-skip-permissions` ou en mode auto | Le dev container préconfiguré, n'importe quel conteneur ou VM, ou le runtime sandbox |
| Isoler les serveurs MCP et les hooks ainsi que Bash, sans Docker                                  | Le runtime sandbox                                                                |
| Travailler sur un référentiel non fiable                                                          | Une VM dédiée, ou Claude Code sur le web                                          |
| Standardiser un environnement sandboxé dans une équipe                                            | Le dev container préconfiguré, copié dans votre référentiel                       |
| Utiliser Claude Code depuis un appareil sans configuration locale                                 | Claude Code sur le web                                                            |
| Exiger l'isolation pour chaque développeur de votre organisation                                  | Appliquer l'isolation dans une organisation                                       |
| Travailler sur un hôte Windows natif                                                              | Un conteneur ou une VM, ou le sandbox Bash à l'intérieur de WSL2                 |

### Comment l'isolation se rapporte aux modes de permission

Les modes de permission décident si un appel d'outil s'exécute et si vous êtes invité en premier. L'isolation restreint ce qu'une commande peut accéder une fois qu'elle s'exécute. Les deux fonctionnent ensemble.

`--dangerously-skip-permissions` supprime entièrement l'examen par action, donc une limite d'isolation est la seule chose limitant ce que Claude peut faire. Exécutez-le toujours à l'intérieur d'un conteneur, d'une VM ou du runtime sandbox.

Le mode auto remplace l'invite par un classificateur qui examine les actions et bloque celles qui escaladent au-delà de la demande, ciblent une infrastructure non reconnue ou semblent motivées par du contenu hostile. Le classificateur est un contrôle par action, pas une limite d'isolation, donc une limite d'isolation ajoute toujours une défense en profondeur.

L'outil Bash sandboxé seul contraint uniquement Bash, donc il n'est pas suffisant pour les exécutions entièrement sans surveillance. Vous pouvez superposer les approches : exécuter l'outil Bash sandboxé à l'intérieur d'un conteneur ou d'une VM.

## Outil Bash sandboxé

> Cette option ne supporte pas Windows natif. Sur les hôtes Windows, utilisez WSL2 ou les approches de conteneur/VM.

L'outil Bash sandboxé est intégré à Claude Code. Il utilise des primitives du système d'exploitation pour restreindre l'accès au système de fichiers et au réseau de chaque commande Bash que Claude exécute : **Seatbelt** sur macOS et **bubblewrap** sur Linux et WSL2. Par défaut, il autorise les écritures dans le répertoire de travail et invite la première fois qu'une commande a besoin d'un nouveau domaine réseau.

Activez-le avec la commande `/sandbox`. Le guide [Sandboxing](/fr/sandboxing) couvre les modes d'approbation, la limite par défaut et comment l'élargir ou la réduire.

**Ce que le sandbox par commande ne couvre pas :**

* Les outils intégrés comme Read, Edit et WebFetch s'exécutent à l'intérieur du processus Claude Code sans générer de code arbitraire — les règles de permission les contrôlent.
* Les serveurs MCP et les hooks sont des processus séparés qui s'exécutent sans contrainte sur l'hôte.

Pour mettre les outils intégrés, les serveurs MCP et les hooks derrière une limite du système d'exploitation, exécutez l'ensemble du processus Claude Code à l'intérieur du runtime sandbox, d'un dev container ou d'un conteneur personnalisé.

## Runtime sandbox

Le package [`@anthropic-ai/sandbox-runtime`](https://github.com/anthropic-experimental/sandbox-runtime) enveloppe un processus entier dans la même isolation Seatbelt ou bubblewrap que le sandbox Bash intégré utilise. Exécuter Claude Code à travers lui contraint chaque outil, hook et serveur MCP dans la session, pas seulement Bash.

Le runtime est un aperçu de recherche bêta, et son format de configuration peut changer.

**Configuration** dans `~/.srt-settings.json` ou via `--settings` :

```json
// Exemple : autoriser l'accès en écriture et les domaines réseau requis
{
  "writePaths": ["~/my-project", "~/.claude", "~/.claude.json"],
  "allowedDomains": ["api.anthropic.com"]
}
```

**Lancement** :

```bash
npx @anthropic-ai/sandbox-runtime claude
```

Le runtime refuse tout accès en écriture et réseau par défaut. Autorisez l'accès en écriture à au moins votre répertoire de projet et aux chemins de configuration de Claude Code `~/.claude` et `~/.claude.json`. Autorisez les domaines réseau dont votre session a besoin.

## Dev containers

Un dev container exécute Claude Code à l'intérieur d'un conteneur Docker que VS Code ou un éditeur compatible gère, avec votre projet monté dedans.

Le référentiel claude-code publie un exemple de dev container avec un pare-feu iptables par défaut-refus. Ce pare-feu bloque l'egress non approuvé, ce qui supporte l'exécution de Claude Code avec `--dangerously-skip-permissions` pour le travail sans surveillance.

Copiez le dev container dans votre référentiel et ajustez :
- La liste blanche du pare-feu
- L'image de base
- La version épinglée de Claude Code

## Conteneur personnalisé

Vous pouvez exécuter Claude Code dans n'importe quelle image de conteneur Docker ou OCI avec vos propres politiques réseau, volumes montés et profils seccomp. C'est le chemin le plus courant pour les organisations avec une infrastructure de conteneur existante ou des exécuteurs CI.

Liste de contrôle pour tout conteneur que vous exploitez :
- Examinez ce qui est monté en écriture
- Vérifiez quelles credentials et tokens sont accessibles à l'intérieur
- Définissez ce que la politique d'egress réseau permet

Vous pouvez superposer le sandbox Bash intégré à l'intérieur du conteneur pour les restrictions par commande. Les conteneurs non privilégiés ont besoin du paramètre nested-sandbox décrit dans [Dépannage du sandboxing](/fr/sandboxing#troubleshooting).

## Machine virtuelle

Une machine virtuelle dédiée fournit la séparation la plus forte, avec son propre noyau et, dans les déploiements cloud ou microVM, son propre matériel virtualisé. Options :
- Instances cloud
- Hyperviseurs locaux
- MicroVMs tels que Firecracker
- Sandboxes Docker Desktop (microVM avec daemon Docker et synchronisation d'espace de travail)

Utilisez cette approche quand :
- Vous évaluez du code non fiable
- Votre politique de sécurité nécessite une séparation au niveau du noyau entre l'agent et l'hôte
- Aucune approche au niveau de l'hôte ne répond à vos exigences de conformité

## Claude Code sur le web

Claude Code sur le web exécute chaque session dans une machine virtuelle isolée gérée par Anthropic. Un proxy réseau applique une liste blanche par défaut, et un proxy séparé détient votre token GitHub en dehors du sandbox tout en émettant des credentials limités pour l'accès au référentiel à l'intérieur.

Utilisez cette approche quand :
- Vous voulez une isolation VM complète sans provisionner l'infrastructure vous-même
- Vous déléguez des tâches à partir d'un appareil sans environnement de développement local

Cela nécessite un abonnement Claude et un compte GitHub connecté, et les sessions clonent votre référentiel depuis GitHub.

## Appliquer l'isolation dans une organisation

Les développeurs individuels peuvent opter pour n'importe quelle approche ci-dessus. Ce qu'une organisation peut appliquer dépend de l'approche :

* **Sandbox Bash intégré** : la seule approche que Claude Code applique lui-même. Livrez les clés de paramètres `sandbox` via les paramètres gérés, soit comme un fichier géré par votre MDM, soit via les paramètres gérés par serveur sur Claude.ai.
* **Dev containers** : validez l'exemple de dev container dans vos référentiels pour standardiser l'environnement dans une équipe. C'est une convention plutôt qu'une limite d'application, car Claude Code ne nécessite pas de conteneur. Si les développeurs ne doivent pas pouvoir exécuter Claude Code en dehors, appliquez cela avec les outils de gestion d'appareils.
* **Conteneurs personnalisés et VMs** : distribuez Claude Code via l'image approuvée et utilisez les outils de gestion d'appareils ou de liste blanche de logiciels de votre organisation.

## Voir aussi

* [Sandboxing](/fr/sandboxing) : configurez l'outil Bash sandboxé intégré
* [Dev container](/fr/devcontainer) : le conteneur de développement Docker préconfiguré
* [Sécurité](/fr/security) : le modèle de sécurité complet de Claude Code
* [Déploiement sécurisé](/fr/agent-sdk/secure-deployment) : conseils d'isolation pour les applications Agent SDK
* [Paramètres](/fr/settings#sandbox-settings) : toutes les clés de configuration sandbox
