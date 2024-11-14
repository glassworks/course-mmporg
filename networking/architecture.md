# Architecture

Vous devrez concevoir une architecture pour votre jeu en réseau.

Pour cet exercice, vous utiliserez très probablement une approche client-serveur. (Les connexions peer-to-peer peuvent également être utilisées, mais elles sont plus difficiles à gérer pour les jeux massivement en ligne).

Dans une approche client-serveur, vous développerez généralement 2 applications différentes :

## Une application serveur

Cette application stocke l'état global du jeu. Cela signifie qu'elle stocke une représentation interne de données telles que :

- La transformation (position, rotation, échelle) des objets.
- Les paramètres concernant les objets dynamiques
- Le contenu de la scène
- Le score, la vie, l'inventaire, ...
- ...

L'application commence par écouter sur un port spécifique, et accepte les connexions entrantes (si TCP) ou les datagrammes (si UDP).

Lorsqu'elle met à jour son état interne, elle doit diffuser (ou multidiffuser, ou envoyer des messages individuels) l'état mis à jour aux clients connectés.

L'application serveur peut être sans tête (c'est-à-dire sans interface graphique) ou comporter un composant de rendu pour afficher une vue centralisée du jeu.


## Une application client

Une application cliente a généralement besoin de l'adresse IP et du port du serveur pour fonctionner.

1. L'adresse IP est saisie manuellement ou découverte automatiquement via des protocoles tels que Bonjour (Apple).
2. Le client établit une connexion avec le serveur.
3. Le serveur envoie l'état actuel au client
4. Le client met à jour son propre état interne en conséquence.
5. Le client commence à rendre la scène sur l'écran de l'utilisateur, en acceptant les interactions de l'utilisateur.

En fonction du jeu, l'application cliente peut recevoir de nombreux messages de mise à jour de la part du serveur, mettant à jour l'état interne du client.

Le client peut également envoyer de nombreuses versions de mise à jour au serveur, notamment en ce qui concerne les objets qu'il contrôle directement (généralement l'avatar de l'utilisateur). Le client peut également envoyer des **commandes** au serveur (ou des appels de procédure à distance (RPC)) qui déclencheront une certaine logique de jeu directement sur le serveur.


## Paradigmes de synchronisation

Le jeu que vous allez développer est une application 3D en temps réel. Cela signifie qu'il y aura des objets 3D qui se déplacent continuellement ou qui changent de position en fonction des interactions de l'utilisateur. L'objectif est de s'assurer que tous les utilisateurs participants voient (approximativement) la même version du jeu à tout moment.

La stratégie que vous utilisez pour synchroniser vos clients et votre serveur dépend de nombreux facteurs spécifiques à votre jeu et à votre déploiement :

- la largeur de bande disponible anticipée (s'agit-il d'une connexion locale via un réseau local ? s'agit-il d'une connexion via l'internet)
- le nombre de connexions simultanées
- les conditions du réseau et la latence
- le niveau de précision requis
- ...


### Le client muet

La stratégie la plus simple consiste à contrôler l'ensemble de l'état sur le serveur. Le serveur envoie des informations sur la position et la rotation de tous les objets dynamiques à tous les clients à une fréquence élevée. 

Le client n'effectue donc aucune mise à jour réelle de l'état. Il se contente de recevoir et de mettre à jour son état interne en fonction de ce qu'il reçoit du serveur.

![](../graphics/dumbclient.png)

Mais comment les utilisateurs interagissent-ils avec le jeu ? En général, les interactions de l'utilisateur sont traduites en commandes qui sont transmises au serveur. Le serveur interprète la commande et met à jour l'état. Cet état sera finalement mis à jour par tous les clients.

![](../graphics/dumbclient-command.png)

Cette approche présente un certain nombre d'avantages :

- elle est simple à mettre en œuvre
- le serveur a également le dernier mot sur la manière d'interpréter une commande et de mettre à jour l'état
- les conditions de course peuvent être gérées selon le principe du premier arrivé, premier servi.

Cependant, elle peut présenter de sérieux inconvénients :

- Le serveur doit envoyer beaucoup de messages à un grand nombre de clients (au moins 30 messages par seconde pour maintenir l'illusion de l'animation). Il aura besoin d'une très grande bande passante pour ce faire, et n'est pas bien adapté.
- Les utilisateurs ressentent souvent la latence entre leur entrée et le résultat final apparaissant à l'écran. Ils doivent attendre que la commande parvienne au serveur, qu'elle soit traitée et que la prochaine mise à jour arrive du serveur.

{% hint style="info" %}

Il est possible de réduire l'utilisation de la bande passante à l'aide d'un certain nombre d'heuristiques. Par exemple, nous n'envoyons des mises à jour que si un objet s'est déplacé ou est en train de se déplacer. Cependant, cela peut créer d'autres problèmes pour les clients qui n'ont pas reçu le dernier message de mise à jour pour un objet.

Une autre stratégie consisterait à n'envoyer des messages de mise à jour que pour les objets se trouvant à proximité (ou en vue) d'un client. Vous devrez cependant gérer la façon d'indiquer au serveur où se trouve votre caméra et ce qu'elle regarde, ainsi que la façon de mettre à jour l'état périmé qui peut revenir à la vue.

{% endhint %}


### Client avec contrôle local

Une stratégie permettant d'éviter la latence pour un utilisateur consiste à prendre le contrôle, sur le client, des objets contrôlés par l'utilisateur.

L'idée est qu'une partie de l'état est « détenue » par le client. C'est alors le client qui envoie des mises à jour régulières de ses propres objets au serveur, voire directement à d'autres clients.

![](../graphics/localcontrol.png)

Le serveur met à jour son état interne et diffuse la mise à jour aux autres clients.

Le principal avantage de cette approche est que l'utilisateur voit immédiatement les résultats de ses actions, sans attendre la réponse du serveur, ce qui permet une expérience beaucoup plus fluide.

{% hint style="info" %}

Ceci est absolument crucial pour toute expérience VR ou AR, où la latence entre une interaction et le résultat visuel peut provoquer des nausées.

{% endhint %}

Mais voyez-vous la difficulté de cette approche ?

Que se passe-t-il si deux utilisateurs différents touchent tous deux un « bonus » dans le monde sur leurs machines locales. Techniquement, ils gagnent tous les deux un point. Mais le bonus ne peut être accordé qu'à l'un des deux joueurs. Le serveur devra décider qui est le gagnant, et l'un des joueurs finira par ne pas gagner de point, même si, visuellement, il s'est vu gagner le point !


### Paramétrage

Une solution pour éviter d'envoyer de nombreux messages de mise à jour serait d'envoyer les paramètres du mouvement à tous les clients au début d'un mouvement, et de laisser chaque client calculer et mettre à jour son état interne localement.

![](../graphics/parameterisation.png)

Il s'agit d'une économie considérable en termes de bande passante, puisque nous n'avons plus qu'à envoyer un message contenant les paramètres du mouvement (vitesse, direction, accélération, etc.), et que chaque client implémente l'algorithme localement.

Cependant, c'est l'aspect le plus difficile à contrôler, notamment en raison de la latence variable entre le client qui émet la commande initiale et l'heure de démarrage sur le serveur et chaque client. En effet, même un décalage d'une demi-seconde peut faire en sorte que les objets se retrouvent à des positions très différentes sur les différents clients !

### Algorithmes de recalage (reckoning)

Pour contourner le problème de la désynchronisation des clients en combinant des mises à jour régulières (mais moins fréquentes) avec la paramétrisation.

L'idée est que les objets continuent à se déplacer localement jusqu'à ce que le client reçoive une mise à jour qui corrige son état. 

Nous comptons sur le fait que pour la majorité des objets, l'erreur sera minime, et que les corrections seront à peine visibles pour l'utilisateur.

{% hint style="info" %}

Avez-vous déjà vu des personnages se « téléporter » soudainement d'une position à une autre dans vos jeux en ligne ? C'est le résultat d'une erreur de paramétrage, enfin résolue par une mise à jour !

{% endhint %}

En fait, [Dead reckoning](https://en.wikipedia.org/wiki/Dead_reckoning) est une technique utilisée dans la navigation, la robotique et les jeux en réseau pour essayer d'anticiper le mouvement des objets sur la base de leurs paramètres actuels, et de passer en douceur à la position mise à jour (correcte) lorsqu'une mise à jour arrive.

![](../graphics/reckoning.png)


## Messagerie

Compte tenu de ces connaissances, vous pouvez commencer à modéliser l'architecture de votre jeu. Vous savez que vous aurez besoin d'au moins deux applications, un serveur et un client. Elles devront communiquer entre elles. Mais comment ?

Une requête HTTP classique ne suffira pas pour cette architecture. Comme vous pouvez le voir, le serveur envoie activement des mises à jour à chaque client !

Si vous les avez déjà utilisées, vous pensez peut-être aux sockets web... et vous êtes sur la bonne voie. L'idée d'un web-socket est de maintenir une connexion ouverte entre un client et un serveur, de sorte que le serveur puisse renvoyer des données au client en temps réel.

Mais pourquoi ajouter la surcharge des sockets « web », alors que nous pouvons simplement utiliser de simples sockets !

Pour ce projet, nous allons descendre dans la pile réseau et travailler directement avec TCP ou UDP pour implémenter notre synchronisation !
