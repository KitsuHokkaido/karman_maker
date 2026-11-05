# Optimisation topologique

## Objectifs

L'objectif de ce tp est d'améliorer le maillage que vous avez précédemment obtenu. 
En l'état le maillage actuel comprend des mailles avec uin très mauvais aspect ratio 
par exemple pour ne citer que ce critére de qualité. Nous allons notamment nous 
concentrer dans cette partie à améliorer ce critère. 

De manière identique au TP précédent quelques pistes vous serons fourni, masi vous êtes en 
autonomie sur ce TP.

Deux optimisations vous sont présentés ici l'une porte sur la géométrie qui a un impact
direct sur le maillage et l'autre sur la construction du maillage en lui même.

## Géométrie

Voici la nouvelle décomposition du domaine visé dans ce TP qui permet d'améliorer 
drastiquement la qualité du maillage.

![](figs/karman_opti.png)

Comme vous pouvez le constater la différence réside dans le fait que le carré central 
comporte des côtés avec une courbure. 

Pour réaliser des arcs je vous invite à regarder du coté de

```bash
HGEOM/Creating geometrical objects/Creating Basic Geometric Objects/Creating Curves
```

Je vous propose une méthode dans la suite, libre à vous de faire selon votre préférence. 
Pour tracer les arcs, j'utilise la méthode : 

```python 
arc = geompy.MakeArc(pt1, pt2, pt3)
```
qui consiste donc à tracer un arc passant par 3 points. 

Je vous propose donc de contrôler la courbure de vos côtés en utilisant en utilisant un
paramètre $c \in [0;1]$ qui est un ratio qui corresponds à un pourcentage de la distance marquée 
en rouge sur l'image précedente, qui est donc la projection du point du milieu de l'arc sur 
le côté d'un carré. L'idée ici est que vous souhaitez faire varier votre côté 
entre une droite et la courbure d'un cercle. Vous pouvez donc exprimer (pythagore) 
cette distance notée d pour un carré de demi-coté l

$$ d = r l (\sqrt{2} - 1) $$

## Groupe 

Pour les groupes en principe rien ne change.

## Maillage 
