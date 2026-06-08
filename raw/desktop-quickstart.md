# Démarrer avec l'application de bureau

Source: https://code.claude.com/docs/fr/desktop-quickstart

Nécessite un abonnement Pro, Max, Team ou Enterprise.

## Téléchargement

- macOS : Universal (Intel + Apple Silicon)
- Windows : x64 ou ARM64

## Onglets de l'application

- **Chat** : Conversation générale sans accès aux fichiers (comme claude.ai)
- **Cowork** : Agent autonome en arrière-plan dans une VM cloud
- **Code** : Assistant de codage interactif avec accès aux fichiers locaux ← ce guide

## Installation en 2 étapes

1. Installer et se connecter (lancer Claude → onglet Code)
2. Ouvrir l'onglet Code

L'app inclut Claude Code — pas besoin d'installer Node.js ou la CLI séparément.

## Première session

1. **Choisir environnement** :
   - **Local** : votre machine, fichiers directs (recommandé pour débuter)
   - **Remote** : infrastructure cloud Anthropic (continue même si app fermée)
   - **SSH** : machine distante via SSH (installe Claude Code automatiquement)

2. **Choisir un modèle** : menu déroulant (Opus, Sonnet, Haiku)

3. **Décrire la tâche** :
   - "Trouver un commentaire TODO et le corriger"
   - "Ajouter des tests pour la fonction principale"
   - "Créer un CLAUDE.md"

4. **Examiner et accepter** : vue de différence, boutons Accepter/Rejeter

## Fonctionnalités clés

- **Interrompre et diriger** : bouton stop ou taper une correction sans attendre
- **Contexte** : `@filename` pour extraire un fichier, glisser-déposer, pièces jointes (images, PDF)
- **Skills** : `/` ou `+` → Slash commands
- **Vue de différence** : `+12 -1` pour voir les changements, commentaires en ligne
- **Modes de permission** : Demander (défaut), Auto-accepter, Plan mode
- **Plugins** : bouton `+` → Plugins
- **Aperçu de l'app** : menu déroulant Preview pour voir l'app en cours d'exécution
- **Suivi PR** : surveille CI et peut corriger/fusionner automatiquement
- **Tâches planifiées** : runs récurrents automatiques
- **Sessions parallèles** : barre latérale, chacune dans son propre git worktree
- **Volet tâches** : sous-agents et commandes en arrière-plan

## Venant de la CLI ?

Desktop utilise le même moteur. Configuration partagée : CLAUDE.md, MCP, hooks, skills, paramètres.
