# How does the Internet work?

The internet is the world’s most popular computer network. There are two main concepts that are fundamental to the way the Internet functions: **packets** and **protocols**.
#### Packets

In networking, a packet is a small segment of a larger message. Each packet contains both data and information about that data. The information about the packet's contents is known as the "header," and it goes at the front of the packet so that the receiving machine knows what to do with the packet. To understand the purpose of a packet header, think of how some consumer products come with assembly instructions.

When data gets sent over the Internet, it is first broken up into smaller packets, which are then translated into bits. The packets get routed to their destination by various networking devices such as routers and switches. When the packets arrive at their destination, the receiving device reassembles the packets in order and can then use or display the data.

#### Protocols

Connecting two computers, both of which may use different hardware and run different software, is one of the main challenges that the creators of the Internet had to solve. It requires the use of communications techniques that are understandable by all connected computers, just as two people who grew up in different parts of the world may need to speak a common language to understand each other.

This problem is solved with standardized protocols. In networking, a protocol is a standardized way of doing certain actions and formatting data so that two or more devices are able to communicate with and understand each other.

There are protocols for sending packets between devices on the same network (**Ethernet**), for sending packets from network to network (**IP**), for ensuring those packets successfully arrive in order (**TCP**), and for formatting data for websites and applications (**HTTP**). In addition to these foundational protocols, there are also protocols for routing, testing, and encryption. And there are alternatives to the protocols listed above for different types of content — for instance, streaming video often uses **UDP** instead of **TCP**. 

Because all Internet-connected computers and other devices can interpret and understand these protocols, the Internet works no matter who or what connects to it.