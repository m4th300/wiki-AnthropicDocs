# Glossaire Claude Code

Source: https://code.claude.com/docs/fr/glossary

## A

**Agent teams** — Plusieurs sessions Claude Code coordonnées par un chef d'équipe, avec liste de tâches partagée et messagerie pair-à-pair. Expérimental, activer avec `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`.

**Agentic coding** — Flux de travail où l'IA peut lire des fichiers, exécuter des commandes et faire des modifications de manière autonome (vs chat qui répond uniquement avec du texte).

**Agentic harness** — Outils, gestion du contexte et environnement d'exécution qui transforment un LLM en agent de codage. Claude Code est le harness, Claude est le modèle.

**Agentic loop** — Cycle : rassembler contexte → agir → vérifier résultats → répéter. Les hooks, skills et MCP se connectent à des phases spécifiques.

**Auto memory** — Notes que Claude écrit pour lui-même, stockées sous `~/.claude/projects/`. Les 200 premières lignes ou 25 KB de MEMORY.md se chargent au démarrage.

**Auto mode** — Mode de permission avec classificateur séparé examinant chaque action en arrière-plan. Bloque escalade de portée et prompt injection. Aperçu de recherche.

## B

**Bare mode** — Drapeau `--bare` qui ignore hooks, skills, plugins, MCP, auto memory et CLAUDE.md. Pour CI et scripts nécessitant comportement identique.

**Bundled skills** — Skills inclus avec Claude Code : `/batch`, `/code-review`, `/debug`, `/loop`.

## C

**Channel** — Serveur MCP qui pousse des événements dans la session en cours (Telegram, Discord, iMessage). Bidirectionnel.

**Checkpoint** — Point de restauration avant chaque édition. `Esc` deux fois ou `/rewind` pour revenir. Séparé de git. Ne couvre pas les commandes Bash.

**`.claude` directory** — Répertoire de configuration au niveau projet. `~/.claude/` pour paramètres utilisateur.

**CLAUDE.md** — Fichier d'instructions persistantes chargé en début de session. Survit à la compaction. Cherché dans : `./CLAUDE.md`, `./.claude/CLAUDE.md`, `~/.claude/CLAUDE.md`, managed policy.

**Command** — Instruction réutilisable invoquée avec `/nom`. Intégrées (`/clear`, `/model`, `/compact`) + personnalisées dans `.claude/commands/`. Skills sont la méthode recommandée.

**Compaction** — Résumé automatique quand la context window approche sa limite. `/compact` pour déclencher manuellement, avec focus possible.

**Context window** — Mémoire de travail d'une session. `/context` pour voir l'utilisation.

## D

**Dispatch** — Routeur initié par téléphone qui crée une session Claude Code dans l'app Desktop depuis l'app mobile Claude. Plans Pro et Max.

## E

**Effort level** — Quantité du budget de réflexion adaptative. Pris en charge sur Opus 4.6+ et Sonnet 4.6. Plus élevé = raisonnement plus profond, plus lent.

**Extended thinking** — Raisonnement étape par étape visible. Plafonner avec `MAX_THINKING_TOKENS`. Apparaît en texte gris italique.

## H

**Hook** — Gestionnaire défini par l'utilisateur qui s'exécute à un point spécifique du cycle de vie (avant/après outil, après édition, au démarrage). Shell, HTTP, MCP, LLM ou subagent.

## M

**Managed settings** — Paramètres appliqués à l'échelle organisation, placés hors de `~/.claude`. Non remplaçables par l'utilisateur.

**MCP (Model Context Protocol)** — Standard ouvert pour connecter les outils d'IA aux services externes. Via `/mcp` ou `.mcp.json`.

**MCP Tool Search** — Mécanisme économisant le contexte : seuls les noms d'outils se chargent au démarrage, schémas complets chargés à la demande.

## N

**Non-interactive mode** — Mode `-p` ou `--print` : exécute une invite et ferme. Pour CI, scripts, piping. Anciennement "headless mode".

## O

**Output style** — Configure le comportement, ton ou format. Désactive certaines parties de l'invite système. Styles : Default, Proactive, Explanatory, Learning.

## P

**Permission mode** — Comportement d'approbation de base. Modes : `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`. Shift+Tab pour changer.

**Permission rule** — Entrée autorisant/demandant/refusant une invocation d'outil. Ordre : deny→ask→allow, premier match gagne.

**Plan mode** — Mode lecture seule : recherche et propose des modifications sans les appliquer. `/plan` ou Shift+Tab.

**Plugin** — Ensemble de skills, hooks, subagents et serveurs MCP packagés. Espacés de noms `plugin-name:skill-name`.

**Project trust** — Dialogue d'acceptation d'un répertoire avant chargement de la configuration.

**Prompt injection** — Instructions hostiles dans des fichiers/pages web tentant de rediriger Claude. Défenses : système de permissions, listes de blocage, auto mode.

## R

**Remote Control** — Continuer une session locale depuis téléphone/navigateur via claude.ai. Le code reste sur votre machine.

**Rules** — Fichiers dans `.claude/rules/` chargés aux côtés de CLAUDE.md. Peuvent être délimités par chemin avec `paths:` en frontmatter YAML.

## S

**Sandboxing** — Isolation OS du système de fichiers et réseau pour l'outil Bash.

**Session** — Conversation liée à votre répertoire, avec sa propre context window. Transcription dans `~/.claude/projects/`.

**Settings layers** — Hiérarchie : managed policy > args CLI > `.claude/settings.local.json` > `.claude/settings.json` > `~/.claude/settings.json`.

**Skill** — Fichier `SKILL.md` avec instructions, connaissances ou workflow. Chargé automatiquement ou invoqué avec `/skill-name`. Successeur des custom commands.

**Subagent** — Assistant spécialisé dans sa propre context window. Travaille sur une tâche déléguée et retourne un résumé. Subagents intégrés : Explore, Plan, usage général.

**Surface** — Tout endroit d'accès à Claude Code : CLI, VS Code, JetBrains, Desktop, claude.ai. Même moteur partout.

## T

**Teleport** — `/teleport` tire une session cloud dans le terminal local. Direction inverse : `--remote`.

**Tool** — Action que Claude peut prendre : lire, modifier, exécuter, rechercher, générer un subagent.

**Turn** — Réponse complète de Claude dans une session (de votre message à sa réponse finale). Stop hooks se déclenchent à la fin.

## V

**Verification loop** — Condition préalable pour `/goal`, exécutions sans surveillance et dynamic workflows. Donne à Claude une vérification exécutable pour savoir que le travail est terminé.

## W

**Worktree isolation** — Mode `-w` ou `isolation: worktree` : exécute Claude dans un git worktree séparé sous `.claude/worktrees/`.

## Termes dépréciés

| Ancien | Actuel |
|--------|--------|
| Headless mode | Non-interactive mode |
| Custom commands | Skills |
| Slash commands | Commands |
