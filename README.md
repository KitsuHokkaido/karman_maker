# TPs Maillage SALOME

## Présentation

Au cours de ces TPs, vous allez apprendre à utiliser l'API Python de SALOME.
L'objectif est de vous rendre indépendants sur cette API, notamment en vous apprenant
à vous repérer dans la documentation.

Ce TP se déroule en trois parties :

1. Présentation de la syntaxe de base de l'API : construction d'une géométrie simple,
   des groupes (conditions aux limites) et du maillage.
2. Construction en autonomie du maillage du cas test de Von Kármán.
3. Pilotage et optimisation automatique du maillage grâce à l'API Python.

## Intérêts

Pourquoi passer par l'API plutôt que par l'interface graphique (GUI) ? Voici les
principaux intérêts de cette méthode :

- **Reproductibilité** : le script contient toute la recette du maillage. On peut le
  versionner (git), le relire, le partager et obtenir exactement le même maillage
  des mois plus tard, ce qui est impossible avec une succession de clics.
- **Automatisation** : un changement de géométrie ou de résolution ne demande que de
  modifier quelques paramètres, et non de tout refaire à la main. On peut aussi générer
  des dizaines de maillages d'affilée (études de convergence, balayages paramétriques).
- **Optimisation du maillage** : comme le maillage est piloté par des paramètres,
  on peut le calculer dans une boucle, mesurer sa qualité ou sa taille, et faire ajuster
  les paramètres automatiquement par un algorithme (TP3).

## Démarrer

1. [TP1](tps/tp1.md) (1 heure)
2. [TP2](tps/tp2.md) (1 heure)
3. [TP3](tps/tp3.md) (30 minutes)

## Licence

Ce projet est sous licence MIT - voir le fichier [LICENSE](LICENSE) pour les détails.
