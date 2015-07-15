---
title: "Sockets 101"
categories: networking
---

A socket is basically an endpoint for communication. Has a specific type, and associated protocol. Accessed via file descriptor obtained when created.

    // I think we need to be a bit more general here, and get rid of vague-ish terms liked 'obtained', instead saying 'returned'.  I would define sockets along the lines of 'A socket is an endpoint for general-purpose interprocess communication.'  Then introduce the categories, based on the address families:

    //  Network sockets (AF_INET and AF_INET6): process--process communication between hosts on a network, using various available protocols

    //  UNIX domain sockets (AF_UNIX): process--process communcation within a host, optionally using the same endpoint, and generally protocol-less (not sure if you can specify anything other than the default protocol number 0 for AF_UNIX)

    //  Netlink sockets (AF_NETLINK): Linux-specific (non-POSIX) process--kernel communication within a host, allowing processes to interact with various kernel subsystems

network protocols assoc -> address family. address family provides basic services to protocol impl. packet fragmentation, routing, addressing, basic transport. address family is number of protocols, one per socket type.

address family defines format of socket address. network address conforms to gen structure called sockaddr, defined in posix.1-2008 sys/socket.h. sa_family field in sockaddr contains address family identifier, specifying format of sa_data area.

protocol characterized by socket type

sockets provide packet routing facility. routing information database is maintained, used for selecting appropriate network interface for packet transmission

network interface corresponds to path that can send and receive messages. usually a hardware device is associated here, not always.

socket types:
	* SOCK_STREAM reliable sequenced full-duplex octet stream between socket and peer. must be in connected state before any data can be sent or received. record boundaries not maintained -- output operation of one size may be received with input ops of other size without data loss. data may be buffered. if data not transmitted within given time, connection considered broken. SIGPIPE signal: thread attempting to send data on broken stream.
	* SOCK_SEQPACKET similar to stream, except record boundaries maintained. single op never transfers parts of more than one record.
	* SOCK_DGRAM connectionless data transfer, not necessarily acknowledged or reliable. datagram may be sent to address specified (multicast or broadcast) in each output op, incoming may be received from multiple sources. source address available.

    // Also list the optional SOCK_RAW
