# TP2 - Maillage du cas test de Von Kármán

## Objectif

L'objectif est de réaliser **en autonomie** le maillage du cas test de Von Kármán :
un écoulement autour d'un cylindre (ici un disque en 2D, extrudé sur une maille
d'épaisseur). Quelques indices vous sont fournis dans la suite, mais tout ce dont vous
avez besoin a été vu au TP1.

Un conseil avant de démarrer : essayez d'encapsuler votre code dans une classe, ou
au minimum de ne laisser aucune valeur numérique "en dur" dans le code. C'est tout
l'intérêt de passer par Python : vous pouvez piloter la géométrie et le maillage avec
quelques paramètres caractéristiques. Par exemple, vous pourriez tout piloter en ne
spécifiant que :

- les dimensions du domaine,
- la taille du disque,
- la distance du disque à l'entrée (inlet),
- une taille de maille caractéristique (par exemple la plus petite, au niveau du disque).

Et c'est tout !

> [!TIP]
> Ce découpage en paramètres sera réutilisé au TP3 pour automatiser et optimiser le maillage.

## Géométrie

Voici une décomposition possible du domaine. Vous êtes libres de ne pas la suivre.

![](figs/karman.png)

Vous aurez besoin de construire un disque. Cherchez dans la documentation le nom de
la méthode, du côté de :

```bash
HGEOM/Creating geometrical objects/Creating 3D Primitives/
```

Vous devrez ensuite faire tourner ce disque, pour des raisons topologiques : le
contour d'un disque a un point de départ imposé, et il faut que les sommets du cercle
coïncident avec les lignes de découpe du domaine (ici, les diagonales) pour que chaque
bloc soit bien un quadrangle à quatre arêtes. Regardez du côté de :

```bash
HGEOM/Transforming geometrical objects/Transformation Operations/
```

Enfin, vous aurez besoin d'une opération booléenne pour retirer le disque du domaine
(le disque est un trou dans l'écoulement). Regardez du côté de :

```bash
HGEOM/Transforming geometrical objects/Boolean Operations/
```

Une fois la géométrie 2D partitionnée, n'oubliez pas l'extrusion, comme au TP1.

## Groupes

Pour les groupes, vous avez déjà vu l'essentiel dans le TP précédent. Il vous faudra
en plus un groupe pour la paroi du disque. Ce sont des faces courbes, cherchez du côté de :

```bash
HGEOM/Creating geometrical objects/Creating Advanced Geometrical Objects/Access to sub-shapes by criteria/
```

(il existe des fonctions pour récupérer les formes situées sur un cylindre).

## Maillage

Pour le maillage, vous avez également vu toutes les méthodes nécessaires dans le TP1.
Pour grossir le maillage en s'éloignant du disque, n'oubliez pas de faire attention au
**sens** des progressions géométriques.

## À vous de jouer

Pour vérifier votre travail :

- `mesh.Compute()` réussit,
- tous les groupes (`INLET`, `OUTLET`, parois, disque, faces avant/arrière) sont non vides,
- le maillage est fin autour du disque et grossit progressivement vers l'extérieur,
- aucune valeur numérique n'est codée en dur au milieu du script.

> [!NOTE]
> N'hésitez pas à demander de l'aide.

## Et après ?

Vous pouvez passer au [TP3](tp3.md).
