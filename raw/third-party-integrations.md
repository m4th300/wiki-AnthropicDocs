# Aperçu du déploiement en entreprise

Source: https://code.claude.com/docs/fr/third-party-integrations

## Comparaison des options de déploiement

| Fournisseur | Idéal pour | Facturation | Inclut Claude.ai |
|------------|-----------|------------|-----------------|
| **Claude for Teams** | La plupart des organisations (recommandé) | 150$/siège Premium + PAYG | Oui |
| **Claude for Enterprise** | Grandes orgas (SSO, SCIM) | Contacter les ventes | Oui |
| **Console** | Développeurs individuels | PAYG | Non |
| **Amazon Bedrock** | Déploiements natifs AWS | PAYG via AWS | Non |
| **Claude Platform on AWS** | AWS Marketplace + fonctionnalités API Claude | PAYG via AWS Marketplace | Non |
| **Google Vertex AI** | Déploiements natifs GCP | PAYG via GCP | Non |
| **Microsoft Foundry** | Déploiements natifs Azure | PAYG via Azure | Non |

**Important** : Certaines fonctionnalités nécessitent un compte Claude.ai :
- Claude Code sur le web
- Routines
- Révision de code
- Contrôle à distance
- Extension Chrome

Si déploiement via Bedrock/Vertex/Foundry, planifier si les développeurs ont aussi besoin de sièges Claude for Teams/Enterprise.

## Configurer les proxies et passerelles

Deux configurations complémentaires :

**Proxy d'entreprise** : tout le trafic via un proxy HTTP/HTTPS pour surveillance/conformité.
```bash
export HTTPS_PROXY='https://proxy.example.com:8080'
```

**Passerelle LLM** : couche proxy devant le fournisseur de cloud (suivi centralisé, auth, rate limiting).
```bash
export ANTHROPIC_BASE_URL='https://your-llm-gateway.com'
```

### Exemples par fournisseur

Bedrock + passerelle :
```bash
export CLAUDE_CODE_USE_BEDROCK=1
export ANTHROPIC_BEDROCK_BASE_URL='https://your-llm-gateway.com/bedrock'
export CLAUDE_CODE_SKIP_BEDROCK_AUTH=1  # Si la passerelle gère l'auth AWS
```

Vertex + passerelle :
```bash
export CLAUDE_CODE_USE_VERTEX=1
export ANTHROPIC_VERTEX_BASE_URL='https://your-llm-gateway.com/vertex'
export CLAUDE_CODE_SKIP_VERTEX_AUTH=1  # Si la passerelle gère l'auth GCP
```

## Meilleures pratiques pour les organisations

1. **CLAUDE.md à plusieurs niveaux** : à l'échelle de l'organisation (chemin système) + niveau référentiel (dans git)
2. **Installation en un clic** : créer un moyen simple d'installer Claude Code pour augmenter l'adoption
3. **Commencer par l'utilisation guidée** : questions, corrections de bugs, puis progresser vers l'agentique
4. **Épingler les versions de modèle** (Bedrock/Vertex/Foundry) : `ANTHROPIC_DEFAULT_OPUS_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`, `ANTHROPIC_DEFAULT_HAIKU_MODEL`
5. **Configurer les politiques de sécurité** : paramètres gérés pour permissions, sandboxing, MCP
6. **MCP pour les intégrations** : `.mcp.json` dans la base de code pour que tous les utilisateurs en bénéficient
7. **Surveiller avec OpenTelemetry** : `CLAUDE_CODE_ENABLE_TELEMETRY=1`

## Vérifier la configuration

`/status` dans Claude Code → ligne "Enterprise managed settings" avec la source entre parenthèses.
