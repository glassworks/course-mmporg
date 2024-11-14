# UDP

UDP, ou **Uniform Datagram Protocol**, est un protocole sans connexion et non fiable qui peut être utilisé pour envoyer des paquets à haute fréquence vers une destination.

L'idée est que chaque paquet (ou datagramme) est d'une taille relativement petite (max. 65 535 octets). Il y a très peu de frais généraux et pas de messages supplémentaires comme ACKS.

Cela signifie que nous ne surchargerons pas notre réseau avec des messages administratifs et que nous pouvons envoyer des messages à une fréquence élevée.

Mais attention ! Il se peut que vous deviez gérer manuellement certaines situations :

- paquets arrivant dans le désordre
- paquets abandonnés (trop de paquets quittant ou entrant dans notre interface réseau)
- les paquets qui n'atteignent pas leur destination

C'est un compromis entre vitesse et fiabilité !

## Configuration

En UDP, il n'y a pas de rôle de serveur par rapport à celui de client. Les deux parties mettront en place un auditeur sur un port pour recevoir des messages. 

Les deux parties peuvent également envoyer des messages - tout ce dont elles ont besoin, c'est d'une adresse IP et d'un port. 

Une fois le message envoyé, il n'y a pas de suivi ou d'assurance que le message arrivera à destination.

En général, lorsqu'un message est reçu, nous obtenons l'adresse IP et le port de l'expéditeur du message (dans le cadre du datagramme UDP). Nous pouvons envoyer une réponse en conséquence.


## UDP en C#/dotnet

La bibliothèque `System.Net.Sockets` contient des outils qui encapsulent le protocole UDP pour nous.

Nous pouvons créer un auditeur UDP et envoyer des messages comme suit :


```c#
using System.Net;
using System.Net.Sockets;

...
public class MyNetwork {
    UdpClient udp;

    // Start listening on a port
    public void Listen(int listenPort) {        
        udp = new UdpClient();        
        udp.Client.SetSocketOption(SocketOptionLevel.Socket, SocketOptionName.ReuseAddress, true);
        udp.ExclusiveAddressUse = false;

        IPEndPoint localEP = new IPEndPoint(IPAddress.Any, listenPort);
        udp.Client.Bind(localEP);
    }

    // Receive UDP data if present
    public void Tick() {        
        IPEndPoint sourceEP = new IPEndPoint(IPAddress.Any, 0);
        while (udp.Available > 0)
		{
            
			byte[] data = udp.Receive(ref sourceEP);

			try
			{
				// Do something with the data
			}
			catch (System.Exception ex)
			{
				Debug.LogWarning("Error receiving UDP message: " + ex.Message);
			}
		}
    }

    // Send data to a destination
    public void Send(string destinationIP, int port, byte[] bytes) {
        IPEndPoint destEP = new IPEndPoint(IPAddress.Parse(destinationIP), port);
        udp.Send(bytes, bytes.Length, destEP);
    }

}
```