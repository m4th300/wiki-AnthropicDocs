# Claude Code avec GitHub Enterprise Server

> Connectez Claude Code à votre instance GitHub Enterprise Server auto-hébergée pour les sessions web, la révision de code et les marketplaces de plugins.

Le support GitHub Enterprise Server (GHES) est disponible pour les plans Team et Enterprise.

Le support GitHub Enterprise Server (GHES) permet à votre organisation d'utiliser Claude Code avec des dépôts hébergés sur votre instance GitHub auto-gérée au lieu de github.com. Une fois qu'un administrateur connecte votre instance GHES, les développeurs peuvent exécuter des sessions web, obtenir des révisions de code automatisées et installer des plugins depuis des marketplaces internes sans aucune configuration par dépôt.

## Ce qui fonctionne avec GitHub Enterprise Server

| Fonctionnalité | Support GHES | Notes |
| :------------- | :----------- | :---- |
| Claude Code sur le web | ✅ Supporté | L'administrateur connecte l'instance GHES une fois ; les développeurs utilisent `claude --remote` ou claude.ai/code comme d'habitude |
| Révision de code | ✅ Supporté | Mêmes révisions PR automatisées que github.com |
| Claude Security | ✅ Supporté | Disponible en version bêta publique pour les plans Enterprise |
| Sessions Teleport | ✅ Supporté | Déplacer les sessions entre web et terminal avec `--teleport` |
| Marketplaces de plugins | ✅ Supporté | Utiliser les URLs git complètes au lieu du raccourci `owner/repo` |
| Métriques de contribution | ✅ Supporté | Livrées via webhooks vers le tableau de bord d'analyse |
| GitHub Actions | ✅ Supporté | Nécessite une configuration manuelle du workflow ; `/install-github-app` est uniquement pour github.com |
| Serveur GitHub MCP | ❌ Non supporté | Le serveur GitHub MCP ne fonctionne pas avec les instances GHES |

## Configuration administrateur

Un administrateur connecte votre instance GHES à Claude Code une fois. Après cela, les développeurs de votre organisation peuvent utiliser les dépôts GHES sans configuration supplémentaire.

La configuration guidée génère un manifeste d'application GitHub et vous redirige vers votre instance GHES pour créer l'application en un clic. Si votre environnement bloque le flux de redirection, une configuration manuelle alternative est disponible.

### Étapes de configuration

1. Allez sur [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code) et trouvez la section GitHub Enterprise Server.

2. Cliquez sur **Connect**. Entrez un nom d'affichage pour la connexion et votre nom d'hôte GHES, par exemple `github.example.com`. Si votre instance GHES utilise un certificat auto-signé ou une autorité de certification privée, collez le certificat CA dans le champ optionnel.

3. Cliquez sur **Continue to GitHub Enterprise**. Votre navigateur redirige vers votre instance GHES avec un manifeste d'application pré-rempli. Examinez la configuration et cliquez sur **Create GitHub App**. GHES vous redirige vers Claude avec les identifiants d'application stockés automatiquement.

4. Depuis la page GitHub App sur votre instance GHES, installez l'application sur les dépôts ou organisations auxquels vous voulez que Claude accède.

5. Retournez sur [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code) et activez la révision de code, Claude Security et les métriques de contribution pour vos dépôts GHES.

### Permissions de l'application GitHub

| Permission | Accès | Utilisé pour |
| :--------- | :---- | :----------- |
| Contents | Lecture et écriture | Cloner les dépôts et pousser des branches |
| Pull requests | Lecture et écriture | Créer des PR et poster des commentaires de révision |
| Issues | Lecture et écriture | Répondre aux mentions d'issues |
| Checks | Lecture et écriture | Poster les vérifications de révision de code |
| Actions | Lecture | Lire le statut CI pour l'auto-fix |
| Repository hooks | Lecture et écriture | Recevoir des webhooks pour les métriques de contribution |
| Metadata | Lecture | Requis par GitHub pour toutes les applications |

L'application s'abonne aux événements : `pull_request`, `issue_comment`, `pull_request_review_comment`, `pull_request_review`, et `check_run`.

### Configuration manuelle

Si le flux de redirection guidée est bloqué par votre configuration réseau, cliquez sur **Add manually** au lieu de Connect. Créez une application GitHub sur votre instance GHES avec les permissions et événements ci-dessus, puis entrez les identifiants de l'application dans le formulaire.

### Exigences réseau

Votre instance GHES doit être accessible depuis l'infrastructure Anthropic afin que Claude puisse cloner les dépôts et poster des commentaires de révision. Si votre instance GHES est derrière un pare-feu, mettez sur liste blanche les adresses IP de l'API Anthropic.

## Workflow développeur

Une fois que votre administrateur a connecté l'instance GHES, aucune configuration côté développeur n'est nécessaire. Claude Code détecte votre nom d'hôte GHES automatiquement depuis le remote git dans votre répertoire de travail.

Clonez un dépôt depuis votre instance GHES :

```bash
git clone git@github.example.com:platform/api-service.git
cd api-service
```

Puis démarrez une session web. Claude détecte l'hôte GHES depuis votre remote git :

```bash
claude --remote "Add retry logic to the payment webhook handler"
```

### Téléporter des sessions vers votre terminal

Tirez une session web dans votre terminal local avec `claude --teleport`. Teleport vérifie que vous êtes dans un checkout du même dépôt GHES avant de récupérer la branche et de charger l'historique de session.

## Marketplaces de plugins sur GHES

Hébergez des marketplaces de plugins sur votre instance GHES pour distribuer des outils internes dans toute votre organisation. La structure du marketplace est identique aux marketplaces hébergées sur github.com ; la seule différence est la façon dont vous les référencez.

### Ajouter un marketplace GHES

Le raccourci `owner/repo` se résout toujours vers github.com. Pour les marketplaces hébergées sur GHES, utilisez l'URL git complète :

```bash
/plugin marketplace add git@github.example.com:platform/claude-plugins.git
```

Les URL HTTPS fonctionnent également :

```bash
/plugin marketplace add https://github.example.com/platform/claude-plugins.git
```

### Mettre sur liste blanche les marketplaces GHES dans les paramètres gérés

Si votre organisation utilise des paramètres gérés pour restreindre les marketplaces que les développeurs peuvent ajouter, utilisez le type de source `hostPattern` pour autoriser tous les marketplaces de votre instance GHES :

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "hostPattern",
      "hostPattern": "^github\\.example\\.com$"
    }
  ]
}
```

Pré-enregistrement de marketplaces pour les développeurs :

```json
{
  "extraKnownMarketplaces": {
    "internal-tools": {
      "source": {
        "source": "git",
        "url": "git@github.example.com:platform/claude-plugins.git"
      }
    }
  }
}
```

## Limitations

* **Commande `/install-github-app`** : suivez le flux de configuration administrateur sur claude.ai à la place
* **Serveur GitHub MCP** : utilisez le CLI `gh` configuré pour votre hôte GHES à la place. Exécutez `gh auth login --hostname github.example.com` pour vous authentifier

## Résolution des problèmes

### La session web échoue à cloner le dépôt

Vérifiez que votre administrateur a terminé la configuration pour votre instance GHES et que l'application GitHub est installée sur le dépôt.

### L'ajout du marketplace échoue avec une erreur de politique

Votre organisation a restreint les sources de marketplace. Demandez à votre administrateur d'ajouter une entrée `hostPattern` pour votre nom d'hôte GHES dans les paramètres gérés.

### L'instance GHES n'est pas accessible

Si les révisions ou les sessions web expirent, votre instance GHES n'est peut-être pas accessible depuis l'infrastructure Anthropic. Confirmez que votre pare-feu autorise les connexions entrantes depuis les adresses IP de l'API Anthropic.

## Ressources connexes

* Claude Code sur le web : exécuter des sessions Claude Code sur l'infrastructure cloud
* Révision de code : révisions PR automatisées
* Marketplaces de plugins : créer et distribuer des catalogues de plugins
* Analytique : suivre l'utilisation et les métriques de contribution
* Paramètres gérés : configuration de politique à l'échelle de l'organisation
* Configuration réseau : exigences de pare-feu et de liste blanche d'IP
