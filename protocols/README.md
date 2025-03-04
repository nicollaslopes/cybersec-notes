# Protocols

A computer network is the communication between two or more computers capable of exchanging information with each other. Each computer on the network can be called a **Host** and in a network we can have **client** and server **hosts**. The **server** is a host that provides some service to the network and the **client** is a host that consumes some service offered by a server.

It turns out that there are hundreds of different hardware and software, we can have Intel, AMD, Dell, Apple computers as well as different operating systems such as Windows, Linux, FreeBSD, macOS etc. How can these computers exchange information with each other since they are so different?

That's where **Protocols** come in. Protocols ensure that different computers using different hardware and software can communicate. Most network protocols have a structure that we can divide into two parts: **header** and **payload**.

Header - contains the protocol's traffic and control information and Each header has a specific structure.

Payload - contains the information that the protocol carries.

## Reference Models

There was a need for standardization to be used as a reference for software developers and hardware manufacturers in order to create products that were compatible with each other.

The OSI Model (Open Systems Interconnection) is a conceptual model made up of 7 layers.

The TCP/IP Model consists of 4 layers and has become a simplification of the OSI model.

<figure><img src="broken-reference" alt=""><figcaption></figcaption></figure>
