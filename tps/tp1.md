# TP1 - Bases de l'API Python de SALOME

## Objectifs

Dans cette partie, nous allons mailler une géométrie très simple :

<img src="figs/cas_simple.png" width="50%">

Le maillage doit être très raffiné au centre, puis devenir de plus en plus grossier
lorsqu'on s'éloigne selon +x, -x, +y et -y (voir les flèches sur l'image), avec une
progression géométrique de la taille des mailles.

Ce cas permet de voir les fondamentaux de l'API de SALOME et de vous rendre
indépendants avec elle. À la fin de ce TP, vous devez être capables de refaire
le même type de travail sur une autre géométrie : c'est exactement l'objectif du TP2.

> [!TIP]
> Vous n'avez pas besoin de tout retenir. L'important est de savoir **où chercher**
> dans la documentation. Gardez-la ouverte en permanence.

## Environnement de travail

Avant de rentrer dans le vif du sujet, vérifions que tout fonctionne.
Il faut d'abord importer les modules nécessaires dans votre fichier Python :

```python
import salome

import GEOM
from salome.geom import geomBuilder

import SMESH
from salome.smesh import smeshBuilder
```

Nous importons depuis les modules `geom` et `smesh` les classes `geomBuilder` et
`smeshBuilder` qui permettent respectivement de construire la géométrie et le maillage.

Il faut ensuite créer les deux objets en question :

```python
geompy = geomBuilder.New()
smesh  = smeshBuilder.New()
```

Pour vérifier que tout fonctionne, construisons un cube et exportons cette
géométrie au format `.vtk` :

```python
pt1 = geompy.MakeVertex(0, 0, 0)
pt2 = geompy.MakeVertex(1, 1, 1)

box = geompy.MakeBoxTwoPnt(pt1, pt2)

geompy.ExportVTK(box, "geometry.vtk")
```

Pour exécuter votre script, depuis un terminal :

```bash
salome -t mon_script.py
```

Si le fichier `geometry.vtk` apparaît (vous pouvez l'ouvrir avec ParaView), c'est bon.

## Se repérer dans la documentation

Dans toute la suite, nous nous appuierons sur deux pages de documentation :

- [La géométrie (GEOM)](https://docs.salome-platform.org/9/gui/GEOM/geompy_doc/group__l1__geomBuilder__purpose.html) :
  toutes les méthodes pour construire et manipuler des géométries, rangées par catégorie.
- [Le maillage (SMESH)](https://docs.salome-platform.org/latest/gui/SMESH/modules.html) :
  toutes les méthodes pour mailler, créer des groupes, etc.

Dans la suite, `HGEOM` et `HMESH` désignent ces deux pages d'accueil (le *home*).
Par exemple, l'adresse suivante vous donne accès aux méthodes pour construire
des primitives dans `geompy` :

```bash
HGEOM/Creating geometrical objects/Creating 3D Primitives/
```

Les adresses sont volontairement données sous cette forme, et non comme des liens directs,
pour que vous appreniez à naviguer dans l'arborescence de la documentation.
Revenez toujours à ce point de départ quand vous cherchez quelque chose.

## Voir ce que l'on construit dans la GUI

Pour vérifier visuellement vos objets, vous pouvez les enregistrer dans l'arbre de
l'étude pour les voir apparaître dans l'interface graphique de SALOME :

```python
geompy.addToStudy(box, "box")
```

et pour le maillage, `smesh.SetName(mesh.GetMesh(), "mon_maillage")`.
Pensez à rafraîchir l'arbre (`salome.sg.updateObjBrowser()`) si besoin.
C'est très utile pour déboguer : si une face ou un groupe n'est pas celui que vous
attendez, vous le verrez tout de suite.

## Géométrie

### Rectangle

Il y a plusieurs façons de construire une géométrie sous SALOME. Il existe des formes
toutes prêtes comme des cubes, des cylindres, etc. :

```bash
HGEOM/Creating geometrical objects/Creating 3D Primitives/
```

Ici, nous allons plutôt construire nous-mêmes le rectangle de l'image ci-dessus.
L'intérêt est que vous verrez comment construire n'importe quelle forme.

Pour cela, rendez-vous ici :

```bash
HGEOM/Creating geometrical objects/Creating basic Geometric Objects/
```

Vous pouvez y tracer des points, des lignes, les relier ensemble et créer une surface.
Les méthodes à utiliser sont les suivantes :

```python
pt1       = geompy.MakeVertex(x, y, z)           # un point
line      = geompy.MakeLineTwoPnt(pt1, pt2)      # un segment entre deux points
wire      = geompy.MakeWire([line1, line2, ...]) # un contour formé de segments
rectangle = geompy.MakeFace(wire, True)          # une surface plane limitée par le contour
```

Attention : les segments d'un `wire` doivent former un contour **connecté et fermé**
(le point d'arrivée d'une ligne est le point de départ de la suivante), sinon la
création de la face échouera.

> [!NOTE]
> **À vous de jouer.** Construisez le rectangle (peu importent les dimensions, par exemple
> 10 x 10). Vous pouvez en profiter pour écrire une petite fonction qui prend en paramètres
> la position et la taille du rectangle : vous la réutiliserez au TP2.

### Partitions

Sur l'image du début, le rectangle est découpé en 9 sous-rectangles. L'intérêt est de
pouvoir, lors du maillage, mailler plus finement certaines zones (ici, le rectangle central).

Pour conserver la forme du domaine tout en créant des sous-formes, on utilise une
**partition**. Cela se trouve dans :

```bash
HGEOM/Transforming geometrical objects/Basic Operations/
```

Pour l'utiliser :

```python
geometry = geompy.MakePartition([rectangle], [line1, line2, ...])
```

Le premier argument est la liste des formes à découper, le second la liste des
objets qui servent à découper (ici, des lignes).

Il est conseillé d'enchaîner ensuite avec :

```python
geometry = geompy.MakeGlueEdges(geometry, 1e-7)
```

qui fusionne les arêtes superposées (à la tolérance près). Cela garantit que deux
sous-rectangles voisins partagent bien la même arête, ce qui est indispensable pour que
le maillage soit conforme.

> [!NOTE]
> **À vous de jouer.** Réalisez le découpage de l'image (4 lignes qui traversent
> entièrement le rectangle). Les dimensions importent peu, c'est pour apprendre.

### Extrusion

Comme avec un logiciel de CAO, on peut extruder une surface pour obtenir un volume.
Nous avons besoin d'un volume car la suite (maillage en hexaèdres) travaille en 3D :
on extrude notre domaine 2D sur une seule épaisseur de maille.

Les fonctions se trouvent dans :

```bash
HGEOM/Creating geometrical objects/Creating Complex Objects/
```

Par exemple :

```python
direction = geompy.MakeVectorDXDYDZ(0, 0, 1)
geometry  = geompy.MakePrismVecH(geometry, direction, 1)
```

Cette fonction attend comme arguments :

- la forme à extruder (ici notre face partitionnée),
- le vecteur de direction de l'extrusion,
- la hauteur de l'extrusion.

## Groupes

Les groupes servent à identifier les frontières de votre domaine (entrée, sortie, parois...).
Ils seront utilisés par la suite pour définir vos conditions aux limites.

On procède en deux temps : on crée d'abord les groupes sur la **géométrie**, puis on les
transmet au **maillage**.

### Côté géométrie

Un groupe sur la géométrie rassemble des formes d'un même type : un groupe d'arêtes
(`EDGE`), de faces (`FACE`), etc. Pour créer un groupe :

```python
inlet = geompy.CreateGroup(geometry, geompy.ShapeType['FACE'])
```

Cette fonction se trouve ici :

```bash
HGEOM/Creating geometrical objects/Working with groups/
```

Avec `geompy.ShapeType['FACE']`, vous précisez que le groupe contiendra des faces.
Pour l'instant il est vide : il faut maintenant y ajouter les faces voulues avec :

```python
geompy.UnionList(inlet, [face1, face2, ...])
```

qui prend une liste de faces. Reste à savoir comment récupérer ces faces. Il y a plusieurs
façons de faire, regardez dans ces pages de la documentation :

```bash
HGEOM/Creating geometrical objects/Creating Advanced Geometrical Objects/Access to sub-shapes by criteria/
HGEOM/Transforming geometrical objects/Blocks Operations/
```

**Méthode 1 : par plan.** `GetShapesOnPlaneWithLocation(...)` récupère toutes les faces
situées dans un plan défini par un point et une normale :

```python
pt     = geompy.MakeVertex(0, 5, 0.5)
normal = geompy.MakeVectorDXDYDZ(-1, 0, 0)
faces  = geompy.GetShapesOnPlaneWithLocation(
    geometry,
    geompy.ShapeType['FACE'],
    normal,
    pt,
    GEOM.ST_ON
)
```

Ici, on récupère toutes les faces qui se trouvent dans le plan passant par `pt` et de
normale `normal`. Le paramètre `GEOM.ST_ON` précise que l'on veut ce qui est *dans* le plan.
Si vous voulez ce qui est en dehors du plan, du côté de la normale, utilisez `GEOM.ST_OUT`
(voir la documentation pour les autres valeurs).

Comme le domaine a été partitionné, la face d'entrée est découpée en plusieurs
sous-faces : cette méthode a l'avantage de toutes les récupérer d'un coup.

**Méthode 2 : par proximité.** On peut aussi récupérer la face la plus proche d'un point,
un peu comme avec un curseur :

```python
pt   = geompy.MakeVertex(0, 5, 0.5)
face = geompy.GetFaceNearPoint(geometry, pt)
```

Ici, comme il y a plusieurs sous-faces sur l'entrée, il faudrait autant de points que de
sous-faces (on peut automatiser cela avec une boucle, mais on retombe sur l'intérêt de la
méthode précédente).

> [!NOTE]
> **À vous de jouer.** Créez quatre groupes de faces :
> `inlet` (entrée, côté -x), `outlet` (sortie, côté +x), `border` (les deux parois
> en ±y) et `fake_bd` (les deux faces avant/arrière issues de l'extrusion, en z = 0 et z = 1).

### Côté maillage

Une fois les groupes créés sur la géométrie, il faut les transférer au maillage. Le
maillage lui-même est vu juste après ; pour les groupes, il suffit d'utiliser :

```python
mesh.GroupOnGeom(inlet, "INLET", SMESH.FACE)
```

Les arguments sont le groupe géométrique, le nom du groupe dans le maillage, et le type
d'élément (ici `SMESH.FACE`). Vous retrouverez cette fonction, et d'autres liées aux
groupes de maillage, ici :

```bash
HMESH/Structured documentation/Grouping elements/Creating groups/
```

## Maillage

Intéressons-nous maintenant au maillage. Comme pour la géométrie, on commence par créer
l'objet maillage, associé à notre géométrie :

```python
mesh = smesh.Mesh(geometry)
```

Voir :

```bash
HMESH/Structured documentation/Creating meshes/Constructing meshes/
```

Nous nous intéressons ici au maillage **structuré**. Le principe est de choisir un
algorithme de maillage et de l'appliquer à une forme (une arête, une face ou un volume),
puis de lui dire comment découper :

```bash
HMESH/Structured documentation/Creating meshes/Defining Algorithms/
```

```python
algo = mesh.Segment(edge)     # découper une arête en segments
algo = mesh.Quadrangle(face)  # mailler une face en quadrangles
algo = mesh.Hexahedron(solid) # mailler un volume en hexaèdres
```

Il faut donc récupérer la forme sur laquelle on veut appliquer l'algorithme. Nous avons
déjà vu comment faire. Pour une arête, par exemple, la méthode `GetEdgeNearPoint` :

```python
pt   = geompy.MakeVertex(1, 1, 1)
edge = geompy.GetEdgeNearPoint(geometry, pt)
```

### Découper une arête

Une fois l'algorithme défini sur une arête, il faut dire **comment** la découper.
Deux méthodes nous intéressent :

```python
algo.NumberOfSegments(100)
```

découpe l'arête en 100 segments de même taille. Simple, uniforme.

```python
algo.GeometricProgression(0.1, 1.1)
```

découpe l'arête avec une **progression géométrique** : la première maille a une taille
de `0.1`, et chaque maille suivante est `1.1` fois plus grande que la précédente
(taille 0.1, 0.11, 0.121, ...). Le nombre de mailles est déduit automatiquement de la
longueur de l'arête. C'est ce qui permet de passer d'un maillage fin à un maillage
grossier.

Attention au **sens** : la progression commence à l'extrémité de départ de l'arête. Si
l'arête est orientée dans le "mauvais" sens (la maille fine se retrouve du côté où vous la
voulez grossière), il faut inverser la progression en passant l'arête dans la liste des
arêtes à inverser :

```python
algo.GeometricProgression(0.1, 1.1, [edge])
```

Vous verrez vite dans le résultat si le sens est le bon.

### Propager

Sur une géométrie structurée, les arêtes opposées d'un même bloc doivent avoir le même
nombre de segments. Au lieu de le répéter à la main sur chaque arête, on utilise :

```python
algo.Propagation()
```

qui propage la découpe de l'arête aux arêtes "opposées" des faces voisines, de proche
en proche. Une seule arête bien choisie suffit donc à régler toute une rangée de mailles.

### Découpe par défaut

Les arêtes non traitées (par exemple celles dans l'épaisseur z de l'extrusion) ont besoin
d'une découpe. On peut en définir une pour toute la forme avec :

```python
mesh.Segment().NumberOfSegments(1)
```

Ici on ne met qu'une seule maille dans l'épaisseur.

> [!NOTE]
> **À vous de jouer.** Appliquez une découpe à chaque zone de votre domaine : 100 segments
> (uniformes) dans le rectangle central, et une progression géométrique de part et d'autre
> pour grossir le maillage vers l'extérieur. Pensez à `Propagation()`.

### Algorithmes 2D et 3D

Une fois les arêtes traitées, on indique les algorithmes pour les faces et le volume :

```python
mesh.Quadrangle()
mesh.Hexahedron()
```

### Calcul et export

Quand vous avez fini de décrire comment mailler chaque zone, vous pouvez calculer
le maillage :

```python
if not mesh.Compute():
    raise Exception("Impossible meshing")

print("Nombre d'hexaedres :", mesh.NbHexas())

mesh.ExportCGNS("mesh.cgns")
```

Et voilà, vous avez toutes les informations pour obtenir ce magnifique maillage :

![](figs/mesh.png)

## Checklist

Avant de passer à la suite, vérifiez que :

- le maillage est bien raffiné au centre et grossit vers les bords,
- `mesh.Compute()` ne renvoie pas d'erreur,
- vos groupes ne sont pas vides (dans la GUI, ou en affichant `group.Size()` pour chaque groupe de `mesh.GetGroups()`),
- vous savez retrouver une méthode dans la documentation sans aide.

## Et après ?

Vous pouvez passer au [TP2](tp2.md).
