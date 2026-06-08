# Héberger le SDK Agent en production

> Déployer l'Agent SDK : architecture, persistance des sessions, mise à l'échelle, observabilité et isolation multi-locataire.

## Le modèle de sous-processus

`query()` → Lance un sous-processus CLI `claude` via stdio → Possède le shell, le répertoire de travail, les fichiers JSONL de session.

Chaque session = un sous-processus. N sessions concurrentes = N sous-processus.

Passer `cwd` explicitement pour des sessions avec des systèmes de fichiers séparés:
```python
query(prompt=prompt, options=ClaudeAgentOptions(cwd="/work/session-a"))
```

## État sur le disque local (perdu au redémarrage)

| État | Emplacement |
|------|-------------|
| Transcriptions de session | `~/.claude/projects/` ou `$CLAUDE_CONFIG_DIR/projects/` |
| Fichiers mémoire CLAUDE.md | `~/.claude/CLAUDE.md` et cwd de la session |
| Artefacts du répertoire de travail | Le répertoire de travail de la session |

Pour persister les transcriptions entre hôtes → `SessionStore`.

## Modèles de session

### Sessions éphémères
Un conteneur par tâche, détruit à la fin. Point d'entrée unique appelle le SDK et se termine.

### Sessions de longue durée
Instances persistantes servant plusieurs processus. Exposer HTTP/WebSocket, mapper sessions → sous-processus.

TypeScript: `streamInput()` + `startup()` pour préchauffer.
Python: `ClaudeSDKClient` pour maintenir une session ouverte.

### Sessions hybrides (recommandé pour la reprise)
Conteneurs éphémères avec `SessionStore`. S'hydratent au démarrage, persistent les mises à jour.

```python
async for message in query(
    prompt=user_input,
    options=ClaudeAgentOptions(
        resume=session_id,
        session_store=session_store,
    ),
):
```

### Conteneur multi-agent
Plusieurs sous-processus dans un conteneur. Donner à chaque agent son propre répertoire de travail.

## Sandboxing

Fournisseurs recommandés:
- Modal Sandbox
- Cloudflare Sandboxes
- Daytona, E2B
- Fly Machines
- Vercel Sandbox

Questions à considérer: latence de démarrage à froid, stockage persistant, tarification, réseau.

## Dépendances d'exécution

- Python 3.10+ ou Node.js 18+
- Le package SDK inclut le binaire Claude Code natif (pas d'installation séparée)
- Mise à jour: mettre à jour le package SDK suffit

## Ressources recommandées

- 1 GiB RAM, 5 GiB disque, 1 CPU par agent comme point de départ
- Calculer: `agents_par_hôte = (RAM_hôte - surcharge) / plafond_RAM_par_session`
- Mesurer le plafond par session avec une session représentative

## Réseau

- HTTPS sortant vers `api.anthropic.com` (ou endpoint régional Bedrock/Vertex)
- Proxy de sortie recommandé pour la production

## Gestion des préoccupations de production

### Persistance
- `SessionStore` pour transcriptions
- Volume monté ou sync objet-store pour CLAUDE.md et artefacts

### Observabilité (OpenTelemetry)
```bash
CLAUDE_CODE_ENABLE_TELEMETRY=1
CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1
OTEL_TRACES_EXPORTER=otlp
OTEL_EXPORTER_OTLP_ENDPOINT=http://collector.example.com:4318
```

### Authentification
- Clé API: `ANTHROPIC_API_KEY` depuis gestionnaire de secrets
- Entrant: authentification à la passerelle devant le conteneur
- Outils sortants: proxy injectant les credentials

### Isolation multi-locataire

```python
query(
    prompt=prompt,
    options=ClaudeAgentOptions(
        cwd=tenant_dir,
        setting_sources=[],  # Pas de paramètres du système de fichiers
        env={
            "CLAUDE_CONFIG_DIR": config_dir,
            "CLAUDE_CODE_DISABLE_AUTO_MEMORY": "1",
        },
    ),
)
```

Également:
- `CLAUDE_CONFIG_DIR` → répertoire par locataire
- `cwd` → répertoire de travail par locataire
- Règles de sortie par locataire sur le proxy

## Limitations connues

| Limitation | Solution |
|------------|---------|
| Pas de timeout de session | Définir `maxTurns` |
| Croissance mémoire sessions longues | Limiter durée ou recycler sous-processus |
| Déploiements parallèles de sous-agents | Diviser en petits lots |
| Pas de limite horloge par sous-agent | `maxTurns` dans `AgentDefinition` |
