# Mouvements

Chaque entité (GameObject) dans Unity possède au moins un composant Transform. 

Le composant Transform représente la position, la rotation et l'échelle d'un objet. 

Pour donner l'impression qu'un objet bouge, il suffit de mettre à jour régulièrement l'une de ces valeurs à chaque itération de la boucle d'animation. Cela donne l'illusion de l'animation, tout comme un film (qui n'est qu'une série d'images changeantes).

La différence avec un film, cependant, est qu'un utilisateur peut interagir avec le jeu, modifiant ainsi éventuellement la trajectoire de l'objet.

## Position

La superclasse `Monobehavior` révèle une référence au composant Transform via la variable `transform` comme raccourci.

Nous pouvons ainsi obtenir la position de l'objet (en tant que vecteur tridimensionnel) : 

```c#
Vector3 position = transform.position;
```

Nous pouvons mettre à jour la position, et donc faire en sorte que notre objet se téléporte à cette nouvelle position, en définissant simplement une nouvelle valeur :

```c#
transform.position = new Vector3(1,4,5);
```

Cela placera l'objet à 1 unité vers la droite (w), 4 unités vers le haut (y) et 5 unités le long de l'axe z.

Nous pouvons également *déplacer* un objet de sa position actuelle vers une nouvelle position en spécifiant uniquement la translation à appliquer :


```c#
Vector3 displacement = new Vector3(5, -2, 0);
transform.Translate(displacement)
```

Dans l'exemple ci-dessus, l'objet se déplacera de 5 unités vers la droite de sa position actuelle et de 2 unités vers le bas.

En mathématiques vectorielles, cela équivaut à obtenir la position actuelle de l'objet et à appliquer le déplacement en effectuant une addition entre les deux vecteurs. Le code suivant fait exactement la même chose :


```c#
Vector3 displacement = new Vector3(5, -2, 0);
transform.position = transform.position + displacement;
```

## Animer la position

Pour créer l'illusion d'une animation, nous devons déplacer l'objet un peu à chaque image.

Idéalement, nous devrions spécifier une vitesse et une direction pour l'objet, et déplacer l'objet de la distance relative au temps qui s'est écoulé depuis la dernière image de notre jeu.


```c#
float Speed = 5;

// Speed = distance / time
// Therefore: distance = Speed * time
float distance = Speed * Time.deltaTime;

// Get a normalized vector representing the direction we
// want to travel
Vector3 direction = (new Vector3(1, 1, 0)).normalized;

// Calculate our displacement 
Vector3 displacement = direction * distance;

// Apply the displacement
transform.Translate(displacement)
```

Ou nous pourrions faire tout cela en moins de lignes :

```c#
float Speed = 5;
Vector3 direction;

transform.Translate(direction.normalized * Speed * Time.deltaTime);
```

Il s'agit bien entendu de la même chose que :


```c#
transform.position = transform.position + direction.normalized * Speed * Time.deltaTime;
```

{% hint style="success" %}

Le code psueudo ci-dessus vous donne une idée de la façon de déplacer un objet.

Il vous appartient maintenant d'effectuer cette opération à chaque itération du jeu. Vous devez donc implémenter cette technique dans la méthode **Update** de votre script.

Essayez donc !

{% endhint %}

## Rotations

Dans Unity, les rotations sont stockées sous forme de quaternions, une structure à 4 dimensions représentant l'orientation d'un objet dans l'espace 3D.

L'éditeur Unity fournit une vue plus simple, les angles d'Euler de l'objet : 
- la rotation autour de l'axe x
- la rotation autour de l'axe y
- la rotation autour de l'axe z


Pour faire pivoter un objet, vous devez spécifier la quantité de rotation autour d'un axe spécifique :


```c#
transform.Rotate(new Vector3(0, 1, 0), 5);
```

Dans l'exemple ci-dessus, nous faisons pivoter l'objet de 5 degrés autour de l'axe vertical.

Les rotations peuvent également être animées, de la même manière que les positions : 


```c#
float RotateSpeed = 5;

transform.Rotate(new Vector3(0, 1, 0), RotateSpeed * Time.deltaTime);
```

Ceci fera pivoter l'objet de 5 degrés **par seconde** s'il est placé dans la méthode `Update` d'un script.

