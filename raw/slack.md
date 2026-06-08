# Claude Code dans Slack

Source: https://code.claude.com/docs/fr/slack

## Ce que c'est

Mention `@Claude` dans un canal Slack → Claude détecte l'intention de codage → crée une session Claude Code sur le web automatiquement.

Construit sur l'application Claude existante pour Slack.

## Prérequis

| Condition | Détails |
|-----------|---------|
| Plan Claude | Pro, Max, Team ou Enterprise avec accès Claude Code |
| Claude Code sur le web | Doit être activé |
| GitHub | Compte connecté + au moins un repo authentifié |
| Slack | Compte lié au compte Claude |

## Configuration

1. **Admin installe l'app Claude** depuis la Slack Marketplace
2. **Connecter votre compte Claude** : Claude (app Slack) → onglet Accueil → Connecter → authentification navigateur
3. **Configurer Claude Code sur le web** : vérifier [claude.ai/code](https://claude.ai/code), connecter GitHub
4. **Choisir le mode de routage** :
   - **Code uniquement** : toutes les @mentions → sessions Claude Code
   - **Code + Chat** : détection intelligente entre Claude Code et Claude Chat
5. **Inviter Claude aux canaux** : `/invite @Claude` dans chaque canal

## Fonctionnement

### Détection automatique

Claude analyse chaque message pour détecter l'intention de codage. Si détecté → session Claude Code sur le web.
Sinon → peut basculer vers Chat.

**Note** : fonctionne uniquement dans les canaux (publics ou privés), PAS dans les DMs.

### Contexte collecté

- À partir d'un fil : tous les messages du fil
- À partir d'un canal : messages récents du canal

### Flux de session

1. @mentionner Claude avec une demande de codage
2. Claude détecte l'intention → crée une session Claude Code
3. Mises à jour de statut dans le fil Slack
4. Achèvement → @mention avec résumé et boutons d'action

### Boutons d'action

- **Afficher la session** : ouvre la session complète sur claude.ai/code
- **Créer une PR** : crée une PR directement
- **Réessayer en tant que Code** : forcer une session Code si détection incorrecte
- **Changer de référentiel** : sélectionner un autre repo

## Accès et permissions

Chaque utilisateur exécute les sessions sous son propre compte Claude :
- Sessions comptent dans les limites du plan individuel
- Accès aux repos = seulement ceux personnellement connectés
- Historique visible dans votre Claude Code sur le web

### Contrôle d'accès

- Chaque utilisateur doit connecter son propre compte Claude
- Claude n'est présent que dans les canaux où il a été invité (`/invite @Claude`)
- Les admins contrôlent quels canaux peuvent utiliser Claude

## Meilleures pratiques

- Être précis (noms de fichiers, noms de fonctions, messages d'erreur)
- Mentionner le repo si pas clair du contexte
- Définir ce que signifie "terminé" (tests, docs, PR ?)
- Utiliser les fils pour que Claude collecte le contexte complet

## Limitations

- GitHub uniquement (pas GitLab, Bitbucket)
- Une PR par session
- Limites de débit du plan individuel s'appliquent
- Accès Claude Code sur le web requis
