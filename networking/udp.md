# UDP

UDP, or Uniform Datagram Protocol, is a connectionless, non-reliable protocol that can be used to send high-frequency packets to a destination.

The idea that each packet (or datagram) is of a relatively small size (max. 65,535 bytes). There is very little overhead, and no additional messages like ACKS.

This means we will not overburden our network with administrative messages, and can send messages at a high frequency.

But be careful ! You may need to manually handle certain situations :

- packets arriving out of order
- dropped packets (too many packets leaving or entering our network interface)
- packets not reaching their destination

It is a compromise between speed and reliability!

## Setup

In UDP we do not have the role of server versus client. Both parties will set up a listener on a port for receiving messages. 

Both can also send messages - all that is needed is an IP Address and a Port. 

Once the message is sent, there is no follow-up or assurance that the message will arrive at its destination.


Typically, once a message is received, we will get the IP address and port of the message sender (as part of the UDP datagram). We can send a response accordingly.


## UDP in C#/dotnet

The `System.Net.Sockets` library contains tools that encapsulate the UDP protocol for us.

We can create a UDP listeners and send messages as follows :


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