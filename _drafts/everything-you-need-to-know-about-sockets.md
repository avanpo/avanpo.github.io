---
title: "Everything you need to know about sockets"
categories: networking
---

Before we get into the details of network sockets, it's helpful to first have a basic understanding of a computer network. More specifically, the computer networking model and communication protocols that we use every day on the internet. This is referred to as the Internet protocol suite, or more commonly TCP/IP.

### TCP/IP

The Internet protocol suite has several layers of abstraction. This abstraction helps ensure that the bottom layer, which are logically closer to the physical transmission of data, are isolated from having to know the details of every application that uses it and the protocols involved. Instead, the bottom layer deals only with the transmission of bits and the potential problems thereof. Similarly, the upper layer can focus on the different applications and associated protocols. This idea of abstraction is seen over and over in computing, and helps manage the complexity of the system.

The IETF loosely defines four layers. Note that these layers do not dictate a strict hierarchical encapsulation sequence -- there are scenarios where this hierarchy breaks down.

* The Link layer is the lowest component layer of the Internet protocols. It is used to move packets over a physical medium between the Internet layer interfaces of two different hosts on the same link. The link layer can be the driver for a network card, or firmware.

* The Internet layer is responsible for routing packets across potentially multiple networks. Using a hierarchical IP addressing system, a host can be addressed and identified. The internet layer identifies the next network closer to the final destination, and sends data packets to this network.

* The Transport layer is responsible for establishing end-to-end message transfer independent of the underlying network. It is also responsible for error control and application addressing (port numbers). This can be either connection-oriented (TCP) or connectionless (UDP).

* The Application layer includes the protocols used by most applications. Examples include HTTP, FTP, and SMTP.

Data encoded according to the application layer protocols are encapsulated into transport layer protocols, which then use the internet layer and link layers to effect actual data transfer.

### Internet sockets

A socket is characterized by several things. The first is a transport layer protocol (e.g. TCP, UDP, or others). The socket also has a local IP address and a port number. Once it has been connected to another socket (e.g. during the establishment of a TCP connection) it will also be characterized by its remote socket address.

The type of the socket is determined by its transport layer protocol. For example, a stream socket is connection-oriented and will use TCP or SCTP. A datagram socket is connectionless and uses UDP. Raw sockets are typically available in network equipment, and bypass the transport layer entirely -- allowing packet headers to be accessible to the application.

On startup, a server will create sockets that are in listening state. This means they wait for a client to connect.

For TCP a socket number is assigned to each unique socket pair consisting of a source and destination IP address, and two port numbers. Since UDP is connectionless, a local IP and port is unique and therefore assigned a socket number, since a UDP socket handles incoming data packets from all remote clients sequentially through the same socket.

The socket is primarily used in the transport layer bla bla networking equipment typically does not require transport layer implementations as they operate on link layer (switches) or internet layer (routers). However stateful firewalls and proxy servers do keep track of socket pairs etc hoi
