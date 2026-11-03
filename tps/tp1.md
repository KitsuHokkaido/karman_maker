# TP1 - Basique de l'API Salome
---

## Objectifs
---

L'objectif dans cette partie sera de mailler cette forme géométrique très simple : 

![](figs/cas_simple.png)

Cela permet de voir les fondamentaux de l'API de Salome et de vous rendre indépendant 
sur cet API.

## Environnement de travail
---

Avant de rentrer dans le vif du sujet, vérifions que tout fonctionne de votre côté. 
Tout d'abord il faut inclure les modules nécessaires dans votre fichier python 

```python
import salome

import GEOM 
from salome.geom import geomBuilder

import SMESH 
from salome.smesh import smeshBuilder
```

Dans ces quelques lignes nous importons depuis les modules `geom` et `smesh` deux 
classes `geomBuilder` et `smeshBuilder` qui comme leurs noms l'indiquent permettent 
respectivelent de contruire la géométrie et le maillage.

Ensuite, il faut créer les deux objets en question : 

```python 
geompy = geomBuilder.New()
smesh  = smeshBuilder.New()

```

Enfin, pour vérifier que tout fonctionne bien, contruisons un cube et exportons 
cette géometrie au format stl par exemple : 

```python 
box = geompy.MakeBox()
```

Dans la suite, nous nous référons constamment à la documentation, il y a 
deux pages sur lesquelles vous devez vous appuyer : 

- [La géometrie](https://docs.salome-platform.org/latest/gui/SMESH/modules.html) : vous 
avez ici la documentation qui vous liste par sous catégorie toutes les méthodes pour mailler.
- [Le maillage](https://docs.salome-platform.org/9/gui/GEOM/geompy_doc/group__l1__geomBuilder__purpose.html) : 
vous avez ici la documentation pour réaliser des géomtries complexes. Toujours revenir à ce point de départ 
lorsque vous cherchez qqch. 

Dans la suite pour la navigation au sein de la documentation lorsque vous verrais : 
`HGEOM` et HMESH` cela fera référence aux adresses cités précédement, le `home`. Par exemple, 
l'adresse suivante donne accès aux méthodes pour construire des primitives dans geompy : 

```bash 
HGEOM/Creating geometrical objects/Creating basic Geometric Objects/
```

## Géométrie 
---

## Groupes
---

## Maillage
---
