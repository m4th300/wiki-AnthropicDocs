# Configurer le mode auto

> Indiquez au classificateur du mode auto quels dépôts, compartiments et domaines votre organisation approuve. Définissez le contexte d'environnement, remplacez les règles de blocage et d'autorisation par défaut, et inspectez votre configuration effective avec les sous-commandes CLI du mode auto.

Le mode auto permet à Claude Code de fonctionner sans invites de permission en acheminant chaque appel d'outil via un classificateur qui bloque tout ce qui est irréversible, destructif, ou visant l'extérieur de votre environnement. Utilisez le bloc de paramètres `autoMode` pour indiquer à ce classificateur quels dépôts, compartiments et domaines votre organisation approuve, afin qu'il cesse de bloquer les opérations internes de routine.

Le mode auto est disponible pour tous les utilisateurs sur l'API Anthropic. Sur Amazon Bedrock, Google Cloud Vertex AI et Microsoft Foundry, vous devez d'abord définir `CLAUDE_CODE_ENABLE_AUTO_MODE`.

Par défaut, le classificateur ne fait confiance qu'au répertoire de travail et aux remotes configurés du dépôt actuel. Des actions comme pousser vers l'org de contrôle de code source de votre entreprise ou écrire dans un compartiment cloud d'équipe sont bloquées jusqu'à ce que vous les ajoutiez à `autoMode.environment`.

## Où le classificateur lit la configuration

Le classificateur lit le même contenu CLAUDE.md que Claude lui-même charge, donc une instruction comme « ne jamais forcer le push » dans votre CLAUDE.md de projet oriente à la fois Claude et le classificateur en même temps. Commencez là pour les conventions de projet et les règles comportementales.

Pour les règles qui s'appliquent à tous les projets, comme l'infrastructure de confiance ou les règles de refus à l'échelle de l'organisation, utilisez le bloc de paramètres `autoMode`. Le classificateur lit `autoMode` depuis les portées suivantes :

| Portée | Fichier | Utilisation |
| :----- | :------ | :---------- |
| Un développeur | `~/.claude/settings.json` | Infrastructure de confiance personnelle |
| Un projet, un développeur | `.claude/settings.local.json` | Compartiments ou services de confiance par projet, gitignored |
| Organisation entière | Paramètres gérés | Infrastructure de confiance distribuée à tous les développeurs |
| Drapeau `--settings` ou Agent SDK | JSON inline | Remplacements par invocation pour l'automatisation |

Le classificateur ne lit pas `autoMode` depuis les paramètres de projet partagés dans `.claude/settings.json`, donc un dépôt vérifié ne peut pas injecter ses propres règles d'autorisation.

Les entrées de chaque portée sont combinées. Un développeur peut étendre `environment`, `allow`, `soft_deny`, et `hard_deny` avec des entrées personnelles mais ne peut pas supprimer les entrées que les paramètres gérés fournissent.

## Définir l'infrastructure de confiance

Pour la plupart des organisations, `autoMode.environment` est le seul champ à définir. Il indique au classificateur quels dépôts, compartiments et domaines sont de confiance.

La liste d'environnement par défaut fait confiance au dépôt de travail et à ses remotes configurés. Pour ajouter vos propres entrées à côté de cette valeur par défaut, incluez la chaîne littérale `"$defaults"` dans le tableau.

```json
{
  "autoMode": {
    "environment": [
      "$defaults",
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted cloud buckets: s3://acme-build-artifacts, gs://acme-ml-datasets",
      "Trusted internal domains: *.corp.example.com, api.internal.example.com",
      "Key internal services: Jenkins at ci.example.com, Artifactory at artifacts.example.com"
    ]
  }
}
```

Les entrées sont en prose, pas en regex ou en motifs d'outil. Le classificateur les lit comme des règles en langage naturel.

Un modèle de départ utile :

```json
{
  "autoMode": {
    "environment": [
      "$defaults",
      "Organization: {COMPANY_NAME}. Primary use: {PRIMARY_USE_CASE}",
      "Source control: {SOURCE_CONTROL}",
      "Cloud provider(s): {CLOUD_PROVIDERS}",
      "Trusted cloud buckets: {TRUSTED_BUCKETS}",
      "Trusted internal domains: {TRUSTED_DOMAINS}",
      "Key internal services: {SERVICES}",
      "Additional context: {EXTRA}"
    ]
  }
}
```

## Remplacer les règles de blocage et d'autorisation

Trois champs supplémentaires permettent de remplacer les listes de règles intégrées du classificateur :

* `autoMode.hard_deny` — limites de sécurité inconditionnelles
* `autoMode.soft_deny` — actions destructives que l'intention de l'utilisateur peut effacer
* `autoMode.allow` — exceptions aux règles soft_deny

La priorité fonctionne en quatre niveaux :

1. Les règles `hard_deny` bloquent inconditionnellement. L'intention de l'utilisateur et les exceptions `allow` ne s'appliquent pas.
2. Les règles `soft_deny` bloquent ensuite. L'intention de l'utilisateur et les exceptions `allow` peuvent les remplacer.
3. Les règles `allow` remplacent ensuite les règles `soft_deny` correspondantes comme exceptions.
4. L'intention explicite de l'utilisateur remplace les blocs soft restants.

Pour conserver les règles intégrées tout en ajoutant les vôtres, incluez `"$defaults"` dans le tableau :

```json
{
  "autoMode": {
    "environment": [
      "$defaults",
      "Source control: github.example.com/acme-corp and all repos under it"
    ],
    "allow": [
      "$defaults",
      "Deploying to the staging namespace is allowed",
      "Writing to s3://acme-scratch/ is allowed"
    ],
    "soft_deny": [
      "$defaults",
      "Never run database migrations outside the migrations CLI",
      "Never modify files under infra/terraform/prod/"
    ],
    "hard_deny": [
      "$defaults",
      "Never send repository contents to third-party code-review APIs"
    ]
  }
}
```

**DANGER** : Définir l'un de `environment`, `allow`, `soft_deny`, ou `hard_deny` sans `"$defaults"` remplace toute la liste par défaut pour cette section.

## Inspecter les valeurs par défaut et votre configuration effective

Trois sous-commandes CLI aident à inspecter et valider votre configuration.

Afficher les règles intégrées `environment`, `allow`, `soft_deny`, et `hard_deny` en JSON :

```bash
claude auto-mode defaults
```

Afficher ce que le classificateur utilise réellement en JSON, avec vos paramètres appliqués :

```bash
claude auto-mode config
```

Obtenir un retour IA sur vos règles `allow`, `soft_deny`, et `hard_deny` personnalisées :

```bash
claude auto-mode critique
```

## Examiner les refus

Quand le mode auto refuse un appel d'outil, le refus est enregistré dans `/permissions` sous l'onglet Recently denied. Appuyez sur `r` sur une action refusée pour la marquer pour réessai.

Des refus répétés pour la même destination signifient généralement que le classificateur manque de contexte. Ajoutez cette destination à `autoMode.environment`, puis exécutez `claude auto-mode config` pour confirmer que cela a pris effet.

## Voir aussi

* Modes de permission : ce qu'est le mode auto, ce qu'il bloque par défaut, et comment l'activer
* Paramètres gérés : déployer la configuration `autoMode` dans toute votre organisation
* Permissions : règles d'autorisation, de demande et de refus qui s'appliquent avant l'exécution du classificateur
* Paramètres : la référence complète des paramètres, y compris la clé `autoMode`
