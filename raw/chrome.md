# Claude Code avec Chrome (bêta)

Source: https://code.claude.com/docs/fr/chrome

## Ce que c'est

Integration de Claude Code avec l'extension Chrome pour automatisation du navigateur.
Fonctionne avec : Google Chrome et Microsoft Edge. PAS Brave, Arc, autres Chromium. PAS WSL.

## Prérequis

- Extension [Claude in Chrome](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn) v1.0.36+
- Claude Code v2.0.73+
- Plan Anthropic direct (Pro, Max, Team, Enterprise). PAS via Bedrock, Vertex, Foundry.

## Démarrer

```bash
claude --chrome
# ou dans une session existante :
/chrome
```

Claude ouvre de nouveaux onglets et partage l'état de connexion de votre navigateur.

## Capacités

- **Débogage en direct** : lire les erreurs de console et l'état du DOM
- **Vérification de conception** : comparer l'UI avec des maquettes Figma
- **Test d'application web** : validation de formulaires, régressions visuelles, flux utilisateur
- **Applications web authentifiées** : Google Docs, Gmail, Notion, etc. sans API
- **Extraction de données** : extraire des informations structurées des pages
- **Automatisation** : remplissage de formulaires, saisie de données, flux multi-sites
- **Enregistrement de session** : sauvegarder les interactions sous forme de GIF

## Exemples

```text
Go to code.claude.com/docs, click on the search box, type "hooks", 
and tell me what results appear

Open localhost:3000, try submitting the form with invalid data,
and check if the error messages appear correctly

Go to the product listings page and extract the name, price, and
availability for each item. Save the results as a CSV file.
```

Si Claude rencontre une page de connexion ou un CAPTCHA → s'arrête et vous demande de le gérer manuellement.

## Activer par défaut

`/chrome` → "Enabled by default"

Note : augmente l'utilisation du contexte (outils du navigateur toujours chargés).

## Dépannage

**Extension non détectée** :
1. Vérifier que l'extension est installée et activée dans `chrome://extensions`
2. Redémarrer Chrome (config d'hôte native installée au premier lancement)
3. `/chrome` → "Reconnect extension"

**Le navigateur ne répond pas** :
- Vérifier s'il y a une boîte de dialogue modale bloquant la page
- Demander à Claude de créer un nouvel onglet

**Connexion interrompue lors de longues sessions** :
Service worker peut devenir inactif → `/chrome` → "Reconnect extension"

**Windows** :
- Conflits de tuyau nommé : redémarrer Claude Code
- Erreurs d'hôte native : réinstaller Claude Code
