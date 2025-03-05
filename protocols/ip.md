# IP

The IP (Internet Protocol) is responsible for delivering packages between machines. By default, it is not reliable, it simply tries to deliver the package without performing any type of verification after delivery.

 If we want reliability and greater control, we can associate the IP protocol with the TCP protocol and if we want speed, we can associate the IP protocol with the UDP protocol.
 
Let's take a look at the structure of the IP protocol header.


Let's see this using wireshark.


As we can see above, we have the ethernet frame with origin from the client host and destination from the server host. (we know this by looking at the Mac Address as seen previously).

Next we have the Internet Protocol Version 4 (IPv4 Protocol), we can see that at the source we have the IP address of the client host and at the destination we have the IP address of the server host.

