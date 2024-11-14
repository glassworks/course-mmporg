# TCP

TCP, ou **Transmission Control Protocol**, est un protocole basé sur la connexion qui garantit la délivrabilité des messages.

La base du TCP est que la « connexion » est maintenue ouverte entre deux hôtes par le biais d'un certain nombre de messages administratifs qui sont envoyés entre eux.

Par exemple, une connexion est établie après une **handshake** initiale entre les deux machines. Notez qu'elles ne sont pas physiquement connectées ! Il y a encore l'internet avec tous ses problèmes qui se trouve entre elles !

Cependant, TCP donne l'illusion d'une connexion en envoyant régulièrement des messages entre les deux hôtes pour s'assurer qu'ils sont toujours joignables. Si aucun message n'est reçu après un certain temps, TCP suppose que l'autre hôte s'est déconnecté.

Une autre caractéristique du TCP est qu'il envoie des messages de  manière **fiable** et **dans l'ordre**, en exigeant un accusé de réception (ACK) de ce message avant d'envoyer le suivant. Si un ACK n'est pas reçu dans un certain délai, TCP réessaie d'envoyer le message jusqu'à ce qu'un ACK soit reçu, ou jusqu'à ce qu'un timeout signale que l'hôte a disparu.

![](../graphics/tcp.png)


{% hint style="info" %}

Toutes les requêtes HTTP utilisent TCP comme couche de transport. Lorsque nous envoyons une requête à un serveur, une connexion TCP est établie et nous nous assurons que la requête est envoyée de manière fiable et dans l'ordre. Il en va de même pour la réponse. Une fois la requête HTTP terminée, la connexion TCP est fermée.

{% endhint %}


Pour le développement d'un jeu, TCP pourrait être considéré comme trop coûteux en termes de bande passante, et pourrait également introduire une latence supplémentaire dans notre synchronisation.

Pensez-y. Si nous envoyons 30 mises à jour par seconde, est-ce vraiment grave si nous en manquons une ou deux ? Voulons-nous vraiment attendre que le message X soit envoyé de manière fiable ou devrions-nous simplement l'ignorer et passer à autre chose ? 

Voulons-nous surcharger notre réseau avec des ACK pour chaque mise à jour envoyée ?

Probablement pas, mais existe-t-il un autre moyen d'envoyer des messages en temps réel sans surcharge ? Oui ! UDP.


## TCP en C#/dotnet

La bibliothèque `System.Net.Sockets` contient des outils qui encapsulent le protocole TCP pour nous.

Nous pouvons créer un serveur TCP qui écoute les connexions entrantes :

```c#
using System.Net;
using System.Net.Sockets;

...
public class MyServer {
    TcpListener tcp;

    public void StartServer() {
        tcp = new TcpListener(IPAddress.Any, ListenPort);
        tcp.Start();


    }

    public void Tick() {
        // Check for new connections
        while (tcp.Pending()) {
            TcpClient tcpClient = tcp.AcceptTcpClient();       
            Debug.Log("New connection received from: " + ((IPEndPoint)tcpClient.Client.RemoteEndPoint).Address);       
            
            // Do something with the new connection, like store it in a list or dictionary
            Connections.Add(tcpClient);     
        }

        // Listen for incoming messages on the connections
        foreach (TcpClient client in Connections) {           

            while (client.Available > 0)
            {   
                byte[] data = new byte[client.Available];
                client.GetStream().Read(data, 0, client.Available);

                try
                {
                   // ... do something with the data
                }
                catch (System.Exception ex)
                {
                    Debug.LogWarning("Error receiving TCP message: " + ex.Message);
                }
            }
        }

    }
    
}
```