# TP1 - Basique de l'API Salome

## Objectifs

L'objectif dans cette partie sera de mailler cette forme géométrique très simple : 

<img src="figs/cas_simple.png" width="50%">

Cela permet de voir les fondamentaux de l'API de Salome et de vous rendre indépendant 
sur cet API.

## Environnement de travail

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
cette géometrie au format .vtk par exemple : 

```python 
pt1 = geompy.MakeVertex(0, 0, 0)
pt2 = geompy.MakeVertex(1, 1, 1)

box = geompy.MakeBoxTwoPnt(pt1, pt2)

geompy.ExportVTK(box, "geometry.vtk")
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

### Rectangle

Vous avez plusieurs façons de construire une géométrie sous SALOME, 
vous avez des formes déjà toutes prêtes commes des cubes, des cylindres, ...
vous pouvez trouver cela dans ici : 

```bash 
HGEOM/Creating geometrical objects/Creating 3D Primitives/
```

Ce que je vous propose plutôt ça va être de constuire nous même le rectangle 
sur l'image plus haut. L'intête c'est ce que vous allez voir comment faire 
n'importe quelle forme.

Pour cela rendez-vous ici : 

```bash 
HGEOM/Creating geometrical objects/Creating basic Geometric Objects/
```
Vous pouvez tracer des points, des lignes, les reliez ensemble et créer une surface.
Les méthodes sont les suivantes : 

```python 
    pt1       = geompy.MakeVertex(x, y, z) 
    line      = geompy.MakeLineTwoPnt(pt1, pt2)
    wire      = geompy.MakeWire([line1, line2, ...])
    rectangle = geompy.MakeFace(wire, True)
```

Pour tracer le rectangle vous devez faire attention aux lignes, en effet 
elles ont un sens vous ne pouvez pas relier deux lignes entre elle si elles 
ne forment pas un contour orienté fermé.

Bien évidemment vous pouvez aussi tracer d'autres éléments, je vous laisse 
regarder dans la documentation. 

### Partitions

Précédement, vous avez vu qu'on a découpé notre rectangle en plusieurs sous formes, 
notamment en 9 sous rectangles l'intérêt ici est surtourt pour permetter lors du 
maillage de mailler plus précisment certains endroits comme pare exemple, on veut plus
maille dans le rectangle du milieu.

Pour conserver la forme de votre domaine tout en créeant des sous formes vous 
pouvez faire des partitions. Cela se trouve dans : 

```bash 
HGEOM/Transforming geometrical object/Basic Operations
```

Pour l'utiliser : 

```python 
geometry = geompy.MakePartition([rectangle], [line1, line2, ...])
```

Je vous laisse ici réaliser le découpage effectué sur l'image du haut.

## Groupes

Passons maintenant sur la construction des groupes. Ici nous allons créer des 
groupes sur la géométrie cela signifie que l'on va rassmebler ensmeble des formes 
d'un même type comme par exmple un groupe d'edges ou alors de faces etc. Ces groupes 
seront utiles par la suite pour créer des groupes sur le maillage qui seront utilisés 
poru deéfinir vos conditions aux limites. 

Pour créer un groupe vous pouvez faire appel à la fonction suivante : 

```python 
geompy.MakeGroup()
```

## Maillage
