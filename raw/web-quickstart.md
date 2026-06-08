# Démarrer avec Claude Code sur le web

Source: https://code.claude.com/docs/fr/web-quickstart

Aperçu de recherche — Pro, Max, Team, Enterprise (sièges premium ou Chat + Claude Code).

## Ce que c'est

Exécute Claude Code sur infrastructure cloud Anthropic (pas votre machine). Nécessite un repo GitHub. Clone le repo dans une VM isolée, fait les modifications, pousse une branche pour révision.

## Idéal pour

- Tâches parallèles (plusieurs sessions indépendantes)
- Repos que vous n'avez pas localement
- Tâches autonomes sans direction fréquente
- Exploration de code

## Comparaison des modes d'exécution

| | Sur le web | Remote Control | Terminal CLI | Desktop |
|--|-----------|----------------|-------------|---------|
| Code s'exécute sur | VM Anthropic | Votre machine | Votre machine | Les deux |
| Configuration locale | Non | Oui | Oui | Selon session |
| Nécessite GitHub | Oui | Non | Non | Session cloud seulement |
| Continue si déconnecté | Oui | Selon terminal | Non | Selon session |

## Connexion à GitHub (une seule fois)

1. Aller sur [claude.ai/code](https://claude.ai/code)
2. Installer l'app Claude GitHub → accorder accès aux repos
3. Créer un environnement cloud :
   - **Nom** : étiquette d'affichage
   - **Accès réseau** : `Trusted` (registres npm/PyPI/etc.) ou `None`
   - **Variables d'environnement** : format `.env`, sans guillemets
   - **Script de configuration** : Bash optionnel avant lancement (mis en cache ~5 min)

### Depuis le terminal (avec CLI GitHub `gh`)

```
/web-setup    # dans Claude Code CLI
```

Synchronise votre jeton `gh` avec votre compte Claude, crée environnement par défaut.

## Démarrer une tâche

1. Sélectionner repo + branche
2. Choisir mode de permission : Auto-accepter (défaut) ou Plan mode
3. Décrire la tâche (précis : nom de fichier, erreur, comportement attendu)

Chaque tâche = sa propre session + branche. Pas besoin d'attendre entre les tâches.

## Examiner et itérer

1. Ouvrir la vue diff (indicateur `+42 -18`)
2. Laisser des commentaires en ligne (sélectionner une ligne → taper → Enter)
3. Créer PR → Ouvrir, Brouillon, ou page GitHub
4. Continuer à itérer après la PR (même session active)

## URL de pré-remplissage

```
https://claude.ai/code?prompt=Fix%20the%20login%20bug&repositories=acme/webapp
```

Paramètres : `prompt`/`q`, `prompt_url`, `repositories`/`repo`, `environment`.

## Dépannage courant

- Aucun repo → vérifier accès GitHub du compte connecté
- "Non disponible pour l'organisation" → admin doit activer pour Enterprise
- Script de configuration timeout → < 5 min, paralléliser avec `&` et `wait`
- Session continue après fermeture onglet → comportement attendu (tourne en arrière-plan)
