# TP3 - Optimisation de la qualité du maillage

## Objectifs

L'objectif de ce TP est d'améliorer le maillage que vous avez obtenu au TP précédent.
En l'état, il contient des mailles avec un très mauvais aspect ratio, pour ne citer que
ce critère de qualité. Nous allons nous concentrer ici sur celui-ci.

Comme pour le TP précédent, quelques pistes vous sont fournies, mais vous êtes en
autonomie.

Deux améliorations sont présentées :

1. une sur la **géométrie**, qui a un impact direct sur le maillage : courber les côtés
   du carré central ;
2. une sur la **construction du maillage** lui-même : calibrer les progressions
   géométriques pour obtenir des mailles les plus carrées possible.

Vous terminerez en faisant varier automatiquement la courbure pour trouver la meilleure
valeur, ce qui est l'intérêt de passer par l'API Python.

## Mesurer avant d'optimiser

Avant d'améliorer quoi que ce soit, il faut savoir mesurer. L'aspect ratio (AR) d'une maille
est grossièrement le rapport entre sa plus grande et sa plus petite dimension : il vaut 1
pour un carré parfait et grandit quand la maille s'étire. La définition exacte utilisée par
SALOME pour chaque type de maille est dans la documentation, ainsi que beaucoup d'autres
critères de qualité :

```bash
HMESH/Structured documentation/Quality controls and Filtering/
```

Une fonction vous sera utile :

```python
ar = mesh.GetAspectRatio(element_id)
```

qui renvoie l'aspect ratio d'un élément à partir de son identifiant.

> [!WARNING]
> Ne mesurez pas l'AR sur les hexaèdres : la dimension dans l'épaisseur de l'extrusion
> compte aussi, et une maille fine de 0.15 sur une épaisseur de 1 aurait un mauvais AR
> sans que le maillage soit mauvais dans le plan. Mesurez-le plutôt sur les quadrangles du
> groupe `FAKE_BD` (les faces en z = 0 et z = épaisseur). Vous pouvez récupérer les groupes
> d'un maillage avec `mesh.GetGroups()`, puis `group.GetName()` et `group.GetIDs()`.

> [!NOTE]
> **À vous de jouer.** Écrivez une fonction qui renvoie l'AR maximal et l'AR moyen de votre
> maillage du TP2. Notez ces valeurs : ce sont elles que vous allez chercher à diminuer.
> Pour savoir **où** sont les pires mailles, affichez aussi les coordonnées (`mesh.BaryCenter(id)`)
> de celles dont l'AR dépasse 5, par exemple.

## Géométrie

Voici la nouvelle décomposition du domaine visée dans ce TP, qui permet d'améliorer
fortement la qualité du maillage.

![](figs/karman_opti.png)

La différence est que le carré central a des côtés **courbés**.

### Les arcs

Pour réaliser des arcs, regardez du côté de :

```bash
HGEOM/Creating geometrical objects/Creating Basic Geometric Objects/Creating Curves
```

Je vous propose une méthode, libre à vous de faire autrement. Pour tracer un arc, j'utilise :

```python
arc = geompy.MakeArc(pt1, pt2, pt3)
```

qui trace un arc **passant par 3 points** : un point de départ, un point au milieu de l'arc
et un point d'arrivée.

### Contrôler la courbure

On contrôle la courbure des côtés avec un paramètre $\alpha \in [0\,;1]$. C'est un
pourcentage de la distance marquée en rouge sur l'image : la distance $d$ entre le milieu du
côté droit du carré et le milieu de l'arc. Ainsi, en variant $\alpha$, le côté passe d'une
droite ($\alpha = 0$) à un arc du cercle qui passe par les 4 coins du carré ($\alpha = 1$).

En appliquant Pythagore, vous pouvez montrer que pour un carré de demi-côté $l$ :

$$ d = \alpha \, l \, (\sqrt{2} - 1) $$

Les coins du carré ne bougent pas : seul le milieu de chaque côté est repoussé vers l'extérieur
de la distance $d$.

L'intérêt de formuler le problème ainsi est que vous allez pouvoir faire varier ce paramètre
jusqu'à obtenir le maillage dont la **pire maille** a le meilleur aspect ratio (voir la partie
optimisation à la fin).

### Construire le domaine

Quelques points à surveiller, qui sont les pièges classiques :

- Les lignes de découpe qui traversaient tout le domaine au TP2 ne peuvent plus traverser le
  carré courbé. Elles doivent maintenant **partir des coins du carré** : une diagonale vers le
  disque et deux prolongements droits vers les bords du domaine, pour chaque coin.
- Il faut un carré courbé en tant que **face**, que vous utiliserez comme outil dans la
  partition (comme les lignes). Pour cela, un `MakeWire` des 4 arcs puis un `MakeFace`.
- Pensez au cas $\alpha = 0$ : un arc passant par 3 points alignés n'existe pas. Dans ce cas,
  construisez simplement un carré à côtés droits.
- Le carré courbé doit rester entièrement dans le domaine : vérifiez-le dans votre code.

> [!NOTE]
> **À vous de jouer.** Construisez la nouvelle géométrie à partir de votre code du TP2, avec
> $\alpha$ comme paramètre supplémentaire.

## Groupes

Pour les groupes, en principe rien ne change.

## Maillage

Vous l'avez peut-être remarqué : en l'état, le maillage n'est pas de bonne qualité dans la zone
qui nous intéresse. Il y a deux causes principales :

1. **Autour du disque**, les mailles sont étirées, car la progression géométrique ne tient pas
   compte de la taille des mailles sur les côtés.
2. **Dans le champ lointain**, la maille transversale est imposée par le découpage du carré
   (elle est propagée jusqu'aux bords), alors que la progression géométrique fait grossir les
   mailles dans la direction de l'écoulement : sur une arête longue, elles deviennent
   très allongées.

Dans les deux cas, l'idée est d'obtenir des mailles **les plus carrées possible**. Cela
signifie qu'il faut calibrer chaque progression géométrique en fonction de la taille de maille
caractéristique choisie.

> [!TIP]
> La taille de maille caractéristique fixe le nombre de segments $N$ sur les côtés du carré.
> Calculez-le sur le côté **droit** du carré, pour qu'il ne dépende pas de $\alpha$ : vous
> comparerez ainsi des maillages qui ont le même nombre de mailles.

### Autour du disque

Pour rappel, `GeometricProgression(start, ratio)` découpe une arête avec une première maille
de taille `start`, puis chaque maille est `ratio` fois plus grande que la précédente.

Plaçons-nous dans un repère cylindrique centré sur le disque. Un quart de cercle de rayon $r$
est découpé en $N$ segments : chaque segment sous-tend un angle $d\theta = \frac{\pi}{2N}$ et
a donc une longueur $r \, d\theta$ à la distance $r$ du centre.

Pour que les mailles soient carrées à **toutes** les distances, la taille radiale doit être
égale à la taille tangentielle :

$$ dr = r \, d\theta $$

Appelons $r_i$ le rayon du $i$-ème noeud le long d'une diagonale, avec $r_0$ le rayon du
disque. La condition précédente s'écrit $r_{i+1} - r_i = r_i \, d\theta$.

> [!NOTE]
> **À vous de jouer.**
> 1. Montrez que les $r_i$ forment une suite géométrique et déterminez sa raison $q$ en
>    fonction de $N$.
> 2. Déduisez-en la taille de la première maille `start` à donner à `GeometricProgression`
>    (la maille doit être carrée dès la paroi du disque).
> 3. Appliquez-la aux diagonales du domaine, et regardez ce que devient l'AR.

<details>
<summary>Vérifier votre résultat</summary>

Vous devez trouver $q = 1 + \frac{\pi}{2N}$ et une première maille égale à la maille
tangentielle sur le disque, $\frac{\pi r_0}{2N}$.

</details>

### Dans le champ lointain

Sur les arêtes longues (vers la sortie, l'entrée et les bords), la maille transversale est
celle des côtés du carré, de taille $a$ (qui n'est plus un paramètre libre). Si la
progression fait grossir la maille jusqu'à une taille $e$ à l'arrivée, l'aspect ratio de ces
mailles vaut environ $e/a$ : il faut donc **borner** $e$, par exemple
$e = AR_{max} \times a$ avec $AR_{max}$ un paramètre de votre choix (de l'ordre de 3 à 5).

Pour une arête de longueur $L$, on cherche donc la raison $q$ et le nombre de mailles $n$ tels que :

$$ a \, \frac{q^n - 1}{q - 1} = L \qquad \text{et} \qquad a \, q^{n-1} = e $$

> [!NOTE]
> **À vous de jouer.** Résolvez ce système pour $q$ (deux équations, deux inconnues : $q^n$
> s'élimine facilement) et appliquez-le à chaque arête. Que faire si l'arête est plus courte
> que $e$ ?

> [!TIP]
> Le sens des progressions : si la maille fine se retrouve du côté où vous la vouliez
> grossière, il faut inverser la progression (voir le TP1). Le sens dépend de l'orientation de
> l'arête, et il est parfois difficile à prévoir : vous pouvez le vérifier sur le maillage calculé,
> en comparant la taille des deux premières mailles et des deux dernières d'une arête.

Pour retrouver vos arêtes après le changement de géométrie : le milieu d'un côté du carré n'est
plus en $(l, 0)$ mais en $(l + d, 0)$.

## Optimisation de la courbure du carré central

Vous avez maintenant un maillage entièrement piloté par quelques paramètres. Cherchons la
courbure $\alpha$ qui donne la meilleure qualité, c'est-à-dire qui minimise l'AR de la pire
maille.

Commencez par écrire une fonction qui, pour une valeur de $\alpha$ donnée, construit la
géométrie, la maille, et renvoie l'AR maximal. Si une construction échoue, elle doit renvoyer
une valeur très grande (vous pourrez ainsi l'utiliser dans une boucle sans qu'une erreur ne
l'arrête).

Vous pouvez ensuite procéder de deux façons :

**1. Balayage.** Calculez cette fonction pour une grille de valeurs de $\alpha$ (par exemple
de 0 à 0.8 par pas de 0.1) et affichez un tableau. Vous devriez voir une courbe en forme de
**U** : un trop grand arrondi dégrade lui-même le maillage. C'est la méthode la plus simple,
et celle qui donne le plus d'intuition.

**2. Algorithme d'optimisation.** Utilisez la fonction de `scipy` :

```python
from scipy.optimize import minimize_scalar

result = minimize_scalar(f, bounds=(0, 0.8), method="bounded")
```

où `f` est votre fonction. Cette méthode ne demande pas de gradient. Un algorithme à gradient
comme L-BFGS-B se comporte mal ici : le nombre de segments est un entier, donc l'AR est
une fonction irrégulière de $\alpha$ et une petite variation du paramètre ne change parfois
rien du tout.

> [!NOTE]
> **À vous de jouer.** Réalisez les deux et vérifiez qu'ils trouvent la même valeur de
> $\alpha$. Quel est le gain sur l'AR par rapport à $\alpha = 0$ ?

## Et après ?

Vous avez obtenu un maillage de bien meilleure qualité ! Mais vous pouvez faire mieux. Vous
pouvez regarder du côté de :

- d'autres critères de qualité (angle minimal, skewness...), listés dans la documentation des
  contrôles de qualité ;
- l'optimisation de plusieurs paramètres à la fois (courbure, taille du carré central,
  $AR_{max}$...), avec un algorithme sans gradient comme Nelder-Mead
  (`minimize(..., method="Nelder-Mead")`) ;
- l'influence de la taille de maille caractéristique sur le résultat.
