# Checkpointing

Source: https://code.claude.com/docs/fr/checkpointing

## Ce que c'est

Claude Code suit automatiquement les modifications de fichiers effectuées par ses outils d'édition AVANT chaque modification. Permet d'annuler et de revenir à des états antérieurs.

- Un checkpoint = créé à chaque prompt envoyé
- Persistent entre les sessions (accessibles dans les conversations reprises)
- Nettoyés automatiquement avec les sessions après 30 jours (configurable)

## Accéder au menu de rembobinage

- Appuyer sur `Esc` deux fois (quand le champ de saisie est vide)
- Ou exécuter `/rewind`

Note : Si le champ contient du texte, `Esc` double efface le texte. Le texte effacé est dans l'historique → `↑` pour rappeler.

## Options du menu de rembobinage

Sélectionner un message → choisir une action :

- **Restaurer le code et la conversation** : revenir au code ET à la conversation à ce moment
- **Restaurer la conversation** : rembobiner jusqu'à ce message + conserver le code actuel
- **Restaurer le code** : annuler les modifications de fichiers + conserver la conversation
- **Résumer à partir d'ici** : compresser la conversation depuis ce moment → libère context window
- **Résumer jusqu'à ici** : compresser la conversation avant ce moment → conserver les messages récents
- **Annuler** : revenir à la liste sans modification

Après "Restaurer la conversation" ou "Résumer à partir d'ici" : l'invite originale du message est restaurée dans le champ de saisie.

## Restaurer vs. résumer

| Option | Ce qui change |
|--------|--------------|
| Restaurer code + conversation | Annule les deux |
| Restaurer conversation | Annule l'historique de messages |
| Restaurer code | Annule les éditions de fichiers |
| Résumer à partir d'ici | Messages APRÈS le point → résumé (messages avant = intacts) |
| Résumer jusqu'à ici | Messages AVANT le point → résumé (messages après = intacts, vous restez à la fin) |

Dans les deux cas de résumé : les messages originaux restent dans la transcription, Claude peut y référencer.

Similaire à `/compact`, mais ciblé sur un côté du message sélectionné.

Note : Résumer = rester dans la même session. Pour brancher une nouvelle session avec un autre approche → utiliser `claude --continue --fork-session`.

## Cas d'usage courants

- Explorer des alternatives (essayer des implémentations différentes)
- Récupérer des erreurs (annuler des bugs introduits)
- Itérer sur des fonctionnalités (variantes en sachant qu'on peut revenir)
- Libérer de l'espace de contexte (résumer une session verbeuse à partir du milieu)

## Limitations importantes

### Les modifications de commandes Bash NE sont PAS suivies

```bash
rm file.txt
mv old.txt new.txt
cp source.txt dest.txt
```
Ces modifications ne peuvent PAS être annulées via le rembobinage.

Seuls les outils d'édition de fichiers de Claude sont suivis (Edit, Write, etc.).

### Les modifications externes ne sont pas suivies

Modifications manuelles hors Claude Code et modifications d'autres sessions concurrentes ne sont généralement pas capturées.

### Pas un remplacement du contrôle de version

- Checkpoints = récupération rapide au niveau de la session
- Git = historique permanent et collaboration
- Checkpoints complètent git, ne le remplacent pas
