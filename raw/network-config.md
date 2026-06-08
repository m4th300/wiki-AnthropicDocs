# Configuration réseau d'entreprise

> Configurez Claude Code pour les environnements d'entreprise avec des serveurs proxy, des autorités de certification (CA) personnalisées et l'authentification mutuelle Transport Layer Security (mTLS).

Claude Code prend en charge diverses configurations réseau et de sécurité d'entreprise via des variables d'environnement. Cela inclut le routage du trafic via des serveurs proxy d'entreprise, la confiance envers des autorités de certification (CA) personnalisées et l'authentification avec des certificats mTLS (Transport Layer Security mutuel) pour une sécurité renforcée.

> Toutes les variables d'environnement affichées sur cette page peuvent également être configurées dans `settings.json`.

## Configuration du proxy

### Variables d'environnement

Claude Code respecte les variables d'environnement proxy standard :

```bash
# Proxy HTTPS (recommandé)
export HTTPS_PROXY=https://proxy.example.com:8080

# Proxy HTTP (si HTTPS non disponible)
export HTTP_PROXY=http://proxy.example.com:8080

# Contourner le proxy pour des requêtes spécifiques - format séparé par des espaces
export NO_PROXY="localhost 192.168.1.1 example.com .example.com"
# Contourner le proxy pour des requêtes spécifiques - format séparé par des virgules
export NO_PROXY="localhost,192.168.1.1,example.com,.example.com"
# Contourner le proxy pour toutes les requêtes
export NO_PROXY="*"
```

> Claude Code ne prend pas en charge les proxies SOCKS.

### Authentification basique

Si votre proxy nécessite une authentification basique, incluez les identifiants dans l'URL du proxy :

```bash
export HTTPS_PROXY=http://username:password@proxy.example.com:8080
```

> Évitez de coder en dur les mots de passe dans les scripts. Utilisez plutôt des variables d'environnement ou un stockage sécurisé des identifiants.

> Pour les proxies nécessitant une authentification avancée (NTLM, Kerberos, etc.), envisagez d'utiliser un service LLM Gateway qui prend en charge votre méthode d'authentification.

## Magasin de certificats CA

Par défaut, Claude Code fait confiance à la fois aux certificats CA Mozilla fournis avec le produit et au magasin de certificats de votre système d'exploitation. Les proxies d'inspection TLS d'entreprise tels que CrowdStrike Falcon et Zscaler fonctionnent sans configuration supplémentaire lorsque leur certificat racine est installé dans le magasin de confiance du système d'exploitation.

`CLAUDE_CODE_CERT_STORE` accepte une liste séparée par des virgules de sources. Les valeurs reconnues sont `bundled` pour l'ensemble de certificats CA Mozilla fourni avec Claude Code et `system` pour le magasin de confiance du système d'exploitation. La valeur par défaut est `bundled,system`.

Pour approuver uniquement l'ensemble de certificats CA Mozilla fourni :

```bash
export CLAUDE_CODE_CERT_STORE=bundled
```

Pour approuver uniquement le magasin de certificats du système d'exploitation :

```bash
export CLAUDE_CODE_CERT_STORE=system
```

> `CLAUDE_CODE_CERT_STORE` n'a pas de clé de schéma dédiée dans `settings.json`. Définissez-la via le bloc `env` dans `~/.claude/settings.json` ou directement dans l'environnement du processus.

## Certificats CA personnalisés

Si votre environnement d'entreprise utilise une CA personnalisée, configurez Claude Code pour la faire confiance directement :

```bash
export NODE_EXTRA_CA_CERTS=/path/to/ca-cert.pem
```

## Authentification mTLS

Pour les environnements d'entreprise nécessitant une authentification par certificat client :

```bash
# Certificat client pour l'authentification
export CLAUDE_CODE_CLIENT_CERT=/path/to/client-cert.pem

# Clé privée du client
export CLAUDE_CODE_CLIENT_KEY=/path/to/client-key.pem

# Optionnel : phrase de passe pour la clé privée chiffrée
export CLAUDE_CODE_CLIENT_KEY_PASSPHRASE="your-passphrase"
```

## Exigences d'accès réseau

Claude Code nécessite l'accès aux URL suivantes. Autorisez ces URL dans votre configuration proxy et vos règles de pare-feu, en particulier dans les environnements réseau conteneurisés ou restreints.

| URL                            | Requis pour                                                                                                                                                         |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `api.anthropic.com`            | Requêtes de l'API Claude                                                                                                                                            |
| `claude.ai`                    | Authentification du compte claude.ai                                                                                                                                |
| `platform.claude.com`          | Authentification du compte Anthropic Console                                                                                                                        |
| `downloads.claude.ai`          | Téléchargements d'exécutables de plugins ; installateur natif et mise à jour automatique native                                                                     |
| `storage.googleapis.com`       | Installateur natif et mise à jour automatique native sur les versions antérieures à 2.1.116                                                                         |
| `bridge.claudeusercontent.com` | Pont WebSocket de l'extension Claude in Chrome                                                                                                                      |
| `raw.githubusercontent.com`    | Flux de changelog pour `/release-notes` et les notes de version affichées après la mise à jour ; compteurs d'installation du marketplace de plugins                 |

Si vous installez Claude Code via npm ou gérez votre propre distribution binaire, les utilisateurs finaux n'auront peut-être pas besoin d'accès à `downloads.claude.ai` ou `storage.googleapis.com`.

Claude Code envoie également une télémétrie opérationnelle facultative par défaut, que vous pouvez désactiver avec des variables d'environnement. Consultez Services de télémétrie pour savoir comment la désactiver avant de finaliser votre liste d'autorisation.

Lors de l'utilisation d'Amazon Bedrock, de Google Vertex AI, ou de Microsoft Foundry, le trafic du modèle et l'authentification vont vers votre fournisseur au lieu de `api.anthropic.com`, `claude.ai`, ou `platform.claude.com`. L'outil WebFetch appelle toujours `api.anthropic.com` pour sa vérification de sécurité du domaine sauf si vous définissez `skipWebFetchPreflight: true` dans les paramètres.

Claude Code sur le web et Code Review se connectent à vos référentiels à partir de l'infrastructure gérée par Anthropic. Si votre organisation GitHub Enterprise Cloud restreint l'accès par adresse IP, activez l'héritage de la liste d'autorisation IP pour les applications GitHub installées. L'application GitHub Claude enregistre ses plages d'adresses IP, donc l'activation de ce paramètre permet l'accès sans configuration manuelle.

Pour les instances GitHub Enterprise Server auto-hébergées derrière un pare-feu, autorisez les mêmes adresses IP de l'API Anthropic afin que l'infrastructure Anthropic puisse accéder à votre hôte GHES pour cloner les référentiels et publier les commentaires d'examen.

## Ressources supplémentaires

* Paramètres Claude Code : `/fr/settings`
* Référence des variables d'environnement : `/fr/env-vars`
* Guide de dépannage : `/fr/troubleshooting`
