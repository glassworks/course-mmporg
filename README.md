# Jeu MMP(O)RG

Have your ever wondered how online multi-player games work ? You have probably never asked yourself that question, but indeed there are many challenges to this problem :

- How can I be sure that I am seeing the same thing as the other players ?
- What about network latency ?
- If two people both recover a bonus object in the game at the same time, who wins ?
- How do we persist the state of the world if a player leaves and comes back ?
- What is the best and most efficient way to communicate between different instances of the game ?
- ...

## The problem

If you have arrived at this level in your studies, it means you have mastered typical client-server architectures that send HTTP requests to a backend (possibly an API). The request is handled by the server, and a response is sent to the client. You may have even played with other types of connection paradigms such as *socket.io*.

However, the architecture typically employed is *client-server*. A single all powerful "server" that is the final source of truth for your platform. 

If this server disappears, then typically the rest of your architecture does not have enough knowledge to continue to function. These days, we invest a lot of time and money in ensuring that this server remains alive, reachable and that it responds within a reasonable delay. Solutions include cloud based redundant deployments, load-balancing, failovers, kubernetes, ...

Sometimes, however, the model of *request-processing-response* is not very pertinent, and can even be overly burdensome for the application in multiple dimensions :

- The server (or server architecture) cannot handle the quantity of requests 
- The available bandwidth is just not enough to carry all required traffic.

A typical example of such a scenario is a networked or online game. In this scenario, we are faced with a *real-time* problem, where state needs to be synchronised multiple times per second. Is the traditional model still applicable in the scenario ?

Add to this scenario the problem of real-time interactions vs. network latency. Two players on opposite sides of the world are participating in the same online game. Both shoot an arrow at a target at exactly the same time. However, due to network latency problems, one player will win, will the other loses. Who decides who is the winner ? 

What if we want to animate the arrow's trajectory ? It would be way too costly and slow to send each step of the arrow's movement to every player in the game. Typically, the arrow's trajectory would be calculated on each player's computer. However, due to latency problems, my arrow may reach the target first on my machine, but in reality another player's arrow reached it first. We have created a paradox. How do we get out of it ?

These problems are not easy to solve, and often we have to tailor our solution to the type of game we are developing, according to what each player can see, and how objects move in the game.

## Objectives

As a software engineer, especially one specializing in the web, it is important to be exposed to engineering problems beyong the typical client-server web-request paradigm. During your career, you will be asked to model and develop solutions that is outside this familiar zone, possibly using paradigms, languages, platforms or architectures to which you are unfamiliar. 

This course therefore has a number of objectives, aimed at reinforcing your toolset with the following elements :

- Understand and appreciate network topologies, and alternatives (client-server, peer-to-peer, ...)
- Become familiar with the two principle transport layer protocols, TCP and UDP, and understand their advantages and disadvantages
- Learn a new programming paradigm through the use of an industry standard game-engine, Unity 3D, and understand the notion of the animation loop, and how to animate real-time interactive experiences
- Learn a new programming language, C#, and become familiar with object oriented programming
- Design and implement a custom protocol that is pertinent to your game
- Be confronted by network related implementation and optimisation problems

Il s'agit d'un projet de groupe, et par conséquent, en tant que futur Tech-Lead/CTO, vous devrez également pratiquer et affiner vos compétences non techniques, mais néanmoins essentielles :

- travailler et coordonner avec d'autres développeurs (planification, délégation, synchronisation, résolution de conflits, ...). Vous êtes libre d'utiliser la philosophie de gestion de projet que vous souhaitez (Agile, Scrum, Kanban, ...).
- travailler avec un dépôt GIT centralisé
- révision du code entre collègues



## The project : a massively multiplayer game

Your mission for this week is to create a massively multi-player real-time game using [Unity3D](https://unity.com/fr), a popular game-engine for creating 2D and 3D games.

What do I mean by **massively** ? I mean, your game should be able to handle more than four concurrent players at any one time!

You have three options for your game :

- **MMPong** : the Unity project sample I have provided includes a demo 2 player Pong game (in `Assets/Demos/Pong/Pong.unity`). Currently we control the two paddles using Z and S (left player) and the up and down arrow (right player). Your mission is to transform this into a massively multiplayer version. One idea could be the following : players from the same promo connect and specify if they are on the left or right hand side of the classroom. The more players that participate simultaneously, the more the game responds accordingly. For example, the more players on the left there are that push the up arrow simultaneously, the quicker the left paddle ascends in the game.

- **MetaVerse** : the Unity project sample I have provided includes a demo **sims** simulation (in `Assets/Demos/MetaVerse/MetaVerse.unity`). Currently we control the two characters using Z/S/Q/D keys (left player) and the arrow-keys (right player). Your mission is to transform this scene into a multiplayer experience where players can join and leave, interact with objects (collect bonuses or maluses, move objects, etc.). Each user of the game should be able to see an accurate view of the scene and what the others are doing. Don't forget to synchronise the state of the world between players too!

- **Your choice** : You are also free to create your own multiplayer game. However, your game must provide for >4 simultaneous players over a network. If you choose this option, you will need to validate your project with me first.

There is one major constraint for this project : you must build your own native protocol over TCP or UDP. **You may not use built in Unity networking tools, plugins or packages, or any other extension from the Asset Store or elsewhere**

## Travail d'équipe

Vous devez travailler en groupe de maximum 4 personnes. Merci de renseigner la constitution de vos groupes ici : 

## Notation

We have asked to create a networked multiplayer game using Unity3D and the C# programming language.

Le but de ce projet est de développer vos compétences d'ingénieur, votre capacité à concevoir et à exécuter un développement logiciel complexe. Les éléments suivants sont donc considérés comme contraires à l'esprit de l'exercice, et ne seront pas autorisés ou acceptés :

La copie ou l'adaptation, de quelque manière que ce soit, des dépôts de traceurs de rayons existants sur Git-Hub ou ailleurs.

L'utilisation de Chat-GPT, ou d'une autre intelligence artificielle, pour écrire votre code.

Je vous demanderai fréquemment d'expliquer votre code, et vous serez pénalisé si vous ne pouvez pas expliquer suffisamment votre structure de données ou vos algorithmes.

**Livraison et livrables**

Vous devrez présenter votre logiciel le vendredi 22 novembre 2024. Vous devrez présenter les éléments suivants :

- vous devez démontrer que votre jeu fonctionne avec 4 joueurs ou plus.
- vous devrez présenter et expliquer une partie de votre code
- vous devez fournir un lien vers le projet GitHub

**Notation**

