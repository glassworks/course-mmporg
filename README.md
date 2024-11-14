---
cover: .gitbook/assets/Capture d’écran 2024-11-14 à 13.07.27.png
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# MMP(O)(R)G

Vous êtes-vous déjà demandé comment fonctionnent les jeux multi-joueurs en ligne ? Vous ne vous êtes probablement jamais posé cette question, mais il y a en effet de nombreux défis à relever pour résoudre ce problème :

* Comment puis-je être sûr que je vois la même chose que les autres joueurs ?
* Qu'en est-il de la latence du réseau ?
* Si deux personnes récupèrent en même temps un objet bonus dans le jeu, qui gagne ?
* Comment maintenir l'état du monde si un joueur quitte le jeu et y revient ?
* Quel est le meilleur moyen de communiquer entre les différentes instances du jeu ?
* ...

## Le problème

Si vous êtes arrivé à ce niveau de vos études, cela signifie que vous maîtrisez les architectures client-serveur typiques qui envoient des requêtes HTTP à un backend (éventuellement une API). La requête est traitée par le serveur et une réponse est envoyée au client. Vous avez peut-être même joué avec d'autres types de paradigmes de connexion tels que _socket.io_.

Cependant, l'architecture généralement employée est _client-serveur_. Un seul « serveur » tout puissant qui est la source finale de vérité pour votre plateforme.

Si ce serveur disparaît, le reste de l'architecture ne dispose généralement pas de suffisamment de connaissances pour continuer à fonctionner. De nos jours, nous investissons beaucoup de temps et d'argent pour nous assurer que ce serveur reste en vie, qu'il est joignable et qu'il répond dans un délai raisonnable. Les solutions incluent des déploiements redondants basés sur le cloud, l'équilibrage de charge, les basculements, kubernetes, ...

Parfois, cependant, le modèle _requête-traitement-réponse_ n'est pas très pertinent, et peut même être trop lourd pour l'application dans de multiples dimensions :

* Le serveur (ou l'architecture du serveur) ne peut pas gérer la quantité de demandes.
* La bande passante disponible n'est tout simplement pas suffisante pour acheminer tout le trafic nécessaire.

Un exemple typique d'un tel scénario est celui d'un jeu en réseau ou en ligne. Dans ce scénario, nous sommes confrontés à un problème de temps réel, où l'état doit être synchronisé plusieurs fois par seconde. Le modèle traditionnel est-il toujours applicable dans ce scénario ?

Ajoutez à ce scénario le problème des interactions en temps réel par rapport à la latence du réseau. Deux joueurs situés à l'autre bout du monde participent au même jeu en ligne. Tous deux tirent une flèche sur une cible exactement au même moment. Cependant, en raison de problèmes de latence du réseau, l'un des joueurs gagne, tandis que l'autre perd. Qui décide qui est le gagnant ?

Et si nous voulions animer la trajectoire de la flèche ? Il serait beaucoup trop coûteux et lent d'envoyer chaque étape du mouvement de la flèche à chaque joueur du jeu. Typiquement, la trajectoire de la flèche serait calculée sur l'ordinateur de chaque joueur. Cependant, en raison de problèmes de latence, ma flèche peut atteindre la cible en premier sur mon ordinateur, mais en réalité, c'est la flèche d'un autre joueur qui l'a atteinte en premier. Nous avons créé un paradoxe. Comment en sortir ?

Ces problèmes ne sont pas faciles à résoudre, et nous devons souvent adapter notre solution au type de jeu que nous développons, en fonction de ce que chaque joueur peut voir et de la façon dont les objets se déplacent dans le jeu.

## Objectifs

En tant qu'ingénieur logiciel, en particulier spécialisé dans le web, il est important d'être exposé à des problèmes d'ingénierie qui dépassent le paradigme client-serveur web-requête typique. Au cours de votre carrière, on vous demandera de modéliser et de développer des solutions qui sortent de cette zone familière, en utilisant éventuellement des paradigmes, des langages, des plates-formes ou des architectures qui ne vous sont pas familiers.

Ce cours a donc un certain nombre d'objectifs, visant à renforcer votre boîte à outils avec les éléments suivants :

* Comprendre et apprécier les topologies de réseau, et les alternatives (client-serveur, pair-à-pair, ...)
* Se familiariser avec les deux principaux protocoles de la couche transport, TCP et UDP, et comprendre leurs avantages et inconvénients.
* Apprendre un nouveau paradigme de programmation à travers l'utilisation d'un moteur de jeu standard, Unity 3D, et comprendre la notion de boucle d'animation, et comment animer des expériences interactives en temps réel.
* Apprendre un nouveau langage de programmation, C#, et se familiariser avec la programmation orientée objet
* Concevoir et mettre en œuvre un protocole personnalisé pertinent pour votre jeu
* Être confronté à des problèmes d'implémentation et d'optimisation liés au réseau.

Il s'agit d'un projet de groupe, et par conséquent, en tant que futur Tech-Lead/CTO, vous devrez également pratiquer et affiner vos compétences non techniques, mais néanmoins essentielles :

* travailler et coordonner avec d'autres développeurs (planification, délégation, synchronisation, résolution de conflits, ...). Vous êtes libre d'utiliser la philosophie de gestion de projet que vous souhaitez (Agile, Scrum, Kanban, ...).
* travailler avec un dépôt GIT centralisé
* révision du code entre collègues

## Le projet : un jeu massivement multijoueur

Votre mission pour cette semaine est de créer un jeu en temps réel **massivement multi-joueurs** en utilisant [Unity3D](https://unity.com/fr), un moteur de jeu populaire pour créer des jeux 2D et 3D.

Qu'est-ce que j'entends par **massivement**? Je veux dire que votre jeu doit être capable de gérer **plus de quatre joueurs simultanés** !

Je vous ai fourni un [projet de base Unity avec des demos ici](https://github.com/glassworks/course-mmporg-sample).

Vous avez trois options pour votre jeu :

* **MMPong**: l'exemple de projet Unity que j'ai fourni inclut une démo de jeu de Pong à 2 joueurs (dans `Assets/Demos/Pong/Pong.unity`). Actuellement, nous contrôlons les deux pagaies en utilisant Z et S (joueur gauche) et les flèches haut et bas (joueur droit). Votre mission est de transformer ce jeu en une version massivement multijoueur. Une idée pourrait être la suivante : les joueurs de la même promotion se connectent et précisent s'ils se trouvent à gauche ou à droite de la salle de classe. Plus il y a de joueurs qui participent simultanément, plus le jeu réagit en conséquence. Par exemple, plus il y a de joueurs à gauche qui appuient simultanément sur la flèche vers le haut, plus la palette de gauche monte rapidement dans le jeu.
* MetaVerse\*\*: l'exemple de projet Unity que j'ai fourni inclut une simulation **sims** de démonstration (dans `Assets/Demos/MetaVerse/MetaVerse.unity`). Actuellement, nous contrôlons les deux personnages en utilisant les touches Z/S/Q/D (joueur gauche) et les touches fléchées (joueur droit). Votre mission est de transformer cette scène en une expérience multijoueur où les joueurs peuvent se rejoindre et se quitter, interagir avec les objets (collecter des bonus ou des malus, déplacer des objets, etc.) Chaque utilisateur du jeu doit pouvoir avoir une vue précise de la scène et de ce que font les autres. N'oubliez pas non plus de synchroniser l'état du monde entre les joueurs !
* **Votre choix**: Vous êtes également libre de créer votre propre jeu multijoueur. Cependant, votre jeu doit prévoir >4 joueurs simultanés sur un réseau. Si vous choisissez cette option, vous devrez d'abord valider votre projet avec moi.

Il y a une contrainte majeure pour ce projet : vous devez construire votre propre protocole natif sur TCP ou UDP. **Vous ne pouvez pas utiliser les outils de réseau intégrés à Unity, les plugins ou les packages, ou toute autre extension provenant de l'Asset Store ou d'ailleurs**.

Vous vous sentez un peu perdu, ou vous ne savez pas par où commencer ? Heureusement, je vous propose quelques points de départ !

* La mise en réseau
  * [Comment concevoir une architecture de jeu](networking/architecture.md)
  * [Introduction à la mise en réseau](networking/introduction.md)
  * [Exemples TCP](networking/tcp.md)
  * [Exemples UDP](networking/udp.md)
* [Une introduction à Unity](unity3d/basics.md)
* [Un projet de démonstration](https://github.com/glassworks/course-mmporg-sample), contenant :
  * Un exemple de jeu Pong (`Assets/Demos/Pong/Pong.unity`)
  * Un exemple de jeu MetaVerse (`Assets/Demos/MetaVerse/MetaVerse.unity`)
  * Un exemple de TCP avec C# (`Assets/Demos/TCP/TCP.unity`)
  * Un exemple d'UDP en C# (`Assets/Demos/UDP/UDP.unity`)

## Travail d'équipe

Vous devez travailler en groupe de maximum 4 personnes. Merci de renseigner la constitution de vos groupes ici : [https://docs.google.com/spreadsheets/d/1L4JQTXWfbR5OX36EWY5PU2A0mlyUb5ecKjiCXGic-Qk/edit?usp=sharing](https://docs.google.com/spreadsheets/d/1L4JQTXWfbR5OX36EWY5PU2A0mlyUb5ecKjiCXGic-Qk/edit?usp=sharing)

## Notation

Nous vous avons demandé de créer un jeu multijoueur en réseau en utilisant Unity3D et le langage de programmation C#.

{% hint style="warning" %}
Le but de ce projet est de développer vos compétences d'ingénieur, votre capacité à concevoir et à exécuter un développement logiciel complexe. Les éléments suivants sont donc considérés comme contraires à l'esprit de l'exercice, et ne seront pas autorisés ou acceptés :

* La copie ou l'adaptation, de quelque manière que ce soit, des dépôts de jeux multi-joueur sur Git-Hub ou ailleurs.
* L'utilisation de Chat-GPT, ou d'une autre intelligence artificielle, pour écrire votre code.
* L'utilisation des outils de réseau intégrés à Unity, les plugins ou les packages, ou toute autre extension provenant de l'Asset Store ou d'ailleurs

Je vous demanderai fréquemment d'expliquer votre code, et vous serez pénalisé si vous ne pouvez pas expliquer suffisamment votre structure de données ou vos algorithmes.
{% endhint %}

**Livraison et livrables**

Vous devrez présenter votre logiciel le vendredi 22 novembre 2024. Vous devrez présenter les éléments suivants :

* vous devez démontrer que votre jeu fonctionne avec 4 joueurs ou plus.
* vous devrez présenter et expliquer une partie de votre code
* vous devez fournir un lien vers le projet GitHub

**Notation**

La notation est réalisée _à la carte_. Un produit de base fonctionnel (MVP) vous vaudra une note qui passe. Ensuite, vous êtes libre de mettre en œuvre toutes les techniques que vous souhaitez pour améliorer votre note, jusqu'à un maximum de 20 points.

La grille de notation suivante sera utilisée pour évaluer le projet :

<table><thead><tr><th width="612">Aspect</th><th>Note</th></tr></thead><tbody><tr><td><strong>Produit de base fonctionnel</strong></td><td></td></tr><tr><td>Un exécutable fonctionnel</td><td>1</td></tr><tr><td>Interaction utilisateur minimal qui fait bouger un objet dans la scene</td><td>1</td></tr><tr><td>Envoie de paquet simple vers une autre machine (TCP ou UDP)</td><td>1</td></tr><tr><td>Au moins un synchronisation de position d'un en force-brute entre 2 machines</td><td>3</td></tr><tr><td>Synchronisation des objets modifiés par des gestes utilisateurs</td><td>2</td></tr><tr><td><strong>Architecture C# et qualité du code</strong></td><td></td></tr><tr><td>Structures de données</td><td>1</td></tr><tr><td>Clean code</td><td>1</td></tr><tr><td>Algorithmes utilisés et correctement expliqués</td><td>1</td></tr><tr><td><strong>Points supplémentaires</strong></td><td></td></tr><tr><td>Protocol personnalisé de communication</td><td>2</td></tr><tr><td>Gestion de la phase de connexion initiale (syncronisation du state initial)</td><td>2</td></tr><tr><td>Techniques de réduction de latence/band-passante</td><td>jusqu'à 8 points</td></tr><tr><td>Gestion des objets partagés dans la scène (bonus à collectionner)</td><td>2</td></tr><tr><td>Instantiation d'autres personnages et leur synchronisation</td><td>4</td></tr><tr><td>Gestion d'une situation de course (2 joueurs prennent en même temps le même objet)</td><td>3</td></tr><tr><td>Phase initial du jeu bien géré (interfaces pour préciser les coordonnées du serveur, connexion, gestion d'erreurs)</td><td>2</td></tr><tr><td>Serveur deployé et jouable sur Internet</td><td>3</td></tr><tr><td>Graphisme supplémentaire (modèles, textures, UI)</td><td>Max. 2 points</td></tr><tr><td>Toute autre caractéristique suffisamment expliquée et mise en œuvre</td><td>3</td></tr></tbody></table>

Vous pouvez commencer par implémenter une version réseau à 2 joueurs du jeu Pong que j'ai fourni. Cela suffira pour obtenir les 11 premiers points si votre architecture est propre et bien implémentée.
