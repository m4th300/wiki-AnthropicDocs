# Analytique Claude Code

Source: https://code.claude.com/docs/fr/analytics

## Accès aux tableaux de bord

| Plan | URL | Inclut |
|------|-----|--------|
| Claude for Teams/Enterprise | [claude.ai/analytics/claude-code](https://claude.ai/analytics/claude-code) | Métriques utilisation, métriques contribution GitHub, classement, export CSV |
| API (Claude Console) | [platform.claude.com/claude-code](https://platform.claude.com/claude-code) | Métriques utilisation, dépenses, insights d'équipe |

## Tableau de bord Teams/Enterprise

### Métriques récapitulatives

- **PRs avec CC** : PRs contenant au moins une ligne écrite avec Claude Code
- **Lignes de code avec CC** : lignes effectives (>3 chars, pas vides, pas seulement crochets) assistées
- **PRs avec CC (%)** : pourcentage de toutes les PRs fusionnées avec code assisté
- **Taux d'acceptation des suggestions** : % fois où les utilisateurs acceptent les suggestions Edit/Write/NotebookEdit
- **Lignes de code acceptées** : total des lignes Claude Code acceptées en session

### Activer les métriques de contribution

Nécessite : admin GitHub installe l'app Claude GitHub + propriétaire Claude active dans les paramètres d'administration.

⚠️ Non disponible avec Zero Data Retention activé.

**Étapes** :
1. Admin GitHub installe l'app Claude sur [github.com/apps/claude](https://github.com/apps/claude)
2. Propriétaire Claude → [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code) → Activer "Analytique Claude Code"
3. Activer "Analytique GitHub"
4. Authentifier et sélectionner les organisations GitHub

Données disponibles dans les 24h, mises à jour quotidiennes.

### Attribution des PRs

- Les PRs sont balisées `claude-code-assisted` dans GitHub
- Fenêtre de temps : 21 jours avant à 2 jours après la date de fusion
- Lignes normalisées avant comparaison (espaces, guillemets, casse)
- Exclus automatiquement : lockfiles, code généré, répertoires dist/build/node_modules
- Code réécrit à >20% par le développeur = non attribué

### Graphiques disponibles

- **Adoption** : utilisateurs actifs quotidiens + sessions
- **PRs par utilisateur** : productivité individuelle dans le temps
- **Demandes de fusion** : répartition quotidienne avec/sans CC
- **Classement** : 10 meilleurs contributeurs par volume

### Export

**Exporter tous les utilisateurs** → CSV complet.

## Tableau de bord Console (API)

- **Lignes de code acceptées**
- **Taux d'acceptation des suggestions**
- **Activité** : utilisateurs actifs quotidiens et sessions
- **Dépenses** : coûts API quotidiens

**Insights d'équipe** : par utilisateur → dépenses et lignes ce mois.

Note : les chiffres de dépenses dans le tableau de bord = estimations. Pour les coûts réels → page de facturation.

## Utiliser les métriques

- **ROI** : comparer PRs/lignes livrées avec et sans Claude Code
- **Adoption** : identifier les utilisateurs avancés pouvant aider les autres
- **DORA** : utiliser aux côtés des métriques de vélocité et de fréquence de déploiement
