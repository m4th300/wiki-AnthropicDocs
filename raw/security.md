# Sécurité

Source: https://code.claude.com/docs/fr/security

## Architecture de sécurité

- Permissions strictes en lecture seule par défaut
- Approbation explicite requise pour : édition de fichiers, exécution de tests, commandes
- Anthropic développe selon son programme de sécurité complet
- Certifications : SOC 2 Type 2, ISO 27001 (voir Trust Center)

## Protections intégrées

**Bash en sandbox** : isolation OS du système de fichiers et réseau (activer avec `/sandbox`).

**Restriction d'écriture** : Claude ne peut écrire que dans le dossier de démarrage et ses sous-dossiers. Peut lire hors du répertoire (bibliothèques système), mais pas écrire.

**Atténuation de la fatigue des invites** : listes blanches des commandes fréquemment utilisées.

**Mode Accepter les modifications** : auto-approuve éditions de fichiers + commandes Bash fixes (mkdir, touch, rm, mv, cp, sed) dans le répertoire de travail.

## Protection contre l'injection de prompt

### Protections principales
- Système de permissions : approbation explicite pour opérations sensibles
- Analyse contextuelle : détecte les instructions potentiellement nuisibles
- Assainissement des entrées : prévient l'injection de commandes
- Liste noire de commandes : `curl` et `wget` bloqués par défaut

### Protections supplémentaires
- Approbation des demandes réseau (outils qui font des requêtes HTTP)
- Fenêtres de contexte isolées pour WebFetch
- Vérification de confiance : première exécution + nouveaux serveurs MCP
  - Vérification désactivée en non-interactif avec `-p` (sauf `--worktree`)
  - Dans le répertoire personnel : confiance en session uniquement (pas sur disque)
- Détection d'injection de commande pour les commandes bash
- Correspondance fail-closed : commandes non correspondues → approbation manuelle
- Descriptions en langage naturel pour les commandes complexes
- Stockage chiffré des identifiants

**Risque WebDAV Windows** : ne pas activer WebDAV avec Claude Code (déprécié par Microsoft, contourne les permissions réseau).

### Meilleures pratiques avec du contenu non fiable
1. Examiner les commandes suggérées avant approbation
2. Éviter de piper du contenu non fiable directement vers Claude
3. Vérifier les modifications proposées aux fichiers critiques
4. Utiliser des VMs pour les scripts et appels d'outils vers des services web externes
5. Signaler les comportements suspects avec `/feedback`

## Sécurité MCP

- Liste des serveurs MCP autorisés : dans votre code source (.mcp.json), enregistrée dans le contrôle de source
- Utiliser des serveurs MCP de fournisseurs de confiance
- Configurer les permissions Claude Code pour les serveurs MCP
- Anthropic examine les connecteurs avant de les ajouter au Répertoire Anthropic, mais n'audite pas la sécurité des serveurs MCP tiers

## Sécurité de l'exécution cloud

Pour Claude Code sur le web :
- VMs isolées par session
- Accès réseau limité par défaut (configurable)
- Authentification via proxy sécurisé (identifiant limité dans le sandbox, traduit en token GitHub réel)
- Restrictions de branche (push limité à la branche de travail)
- Journalisation d'audit de toutes les opérations
- Nettoyage automatique après la session

Pour Remote Control :
- Code s'exécute sur votre machine locale (pas de VM cloud)
- Les données transitent via l'API Anthropic via TLS
- Connexion utilise des identifiants de courte durée et à portée étroite

## Meilleures pratiques de sécurité

### Avec du code sensible
- Examiner toutes les modifications proposées
- Paramètres de permission spécifiques aux repos sensibles
- Envisager les devcontainers pour isolation supplémentaire
- Auditer régulièrement avec `/permissions`

### Sécurité d'équipe
- Utiliser les paramètres gérés pour les normes organisationnelles
- Partager les configurations approuvées via le contrôle de source
- Former l'équipe aux meilleures pratiques
- Surveiller l'utilisation via métriques OpenTelemetry
- Auditer/bloquer les modifications de paramètres avec les hooks `ConfigChange`

## Signalement des vulnérabilités

Via le programme HackerOne d'Anthropic. Ne pas divulguer publiquement. Inclure les étapes de reproduction.

## Ressources connexes

- [Plugin de conseils en sécurité] : faire examiner et corriger par Claude les vulnérabilités dans ses propres modifications
- [Sandbox environments] : comparaison des approches d'isolation
- [Sandboxing] : isolation OS pour Bash
- [Permissions] : contrôles d'accès granulaires
- [Centre de confiance Anthropic](https://trust.anthropic.com) : certifications SOC 2, ISO 27001
