---
title: "Sockets 101"
categories: networking
---

A socket is an endpoint for general-purpose interprocess communication. Sockets are created with a specific type. These abstract socket types are implemented by protocols belonging to an associated address family.

// maybe a note about POSIX?

## Address families

Address families provide basic services to the protocol implementation, which may include packet fragmentation and reassembly, routing, and basic transport. An address family comprises of zero or more protocols for every abstract socket type.

The address family is described using a general structure called a sockaddr, and defines the format of the socket address. The network address is described using a structure called sockaddr (defined in posix.1-2008 sys/socket.h). The sa_family field contains the address family identifier.

// is this everything? or are there more

### Network sockets

Network sockets allow process to process communication between hosts on a network, using various available protocols. The symbolic constaint AF_INET (defined in sys/socket.h) is used to identify the IPv4 address family. The IPv4 address can be mapped to an IPv6 address by opening a socket using the identifier AF_INET6.

### UNIX domain sockets

UNIX domain sockets allow interprocess communication within a host, optionally using the same endpoint. There is only one valid protocol for AF_UNIX, namely 0 (see unix(7) manpages).

### Netlink sockets

Netlink sockets allow Linux-specific (non-POSIX) process to kernel communication within a host, allowing processes to interact with various kernel subsystems.

// maybe make that a list instead of sub headers

## Socket types

The socket type defines communication semantics and allows the selection of an appropriate communication protocol. There are four defined types, although implementations may specify additional socket types.

### SOCK_STREAM

The SOCK_STREAM socket type allows a reliable, sequenced, full-duplex octet stream between a socket and its peer. The socket must be in a connected state before any data is sent or received. Record boundaries are not maintained, so output operations of one size may be received using input operations of another. Data may be buffered, so a successful return does not imply data has be received or even transmitted.

If data is not successfully transmitted within a given time frame, the connection is considered broken, and subsequent operations will fail. Out-of-band data support is protocol-specific.

### SOCK_SEQPACKET

The SOCK_SEQPACKET socket type is similar to the SOCK_STREAM type. The only difference is that record boundaries are maintained. A single operation never tranfers parts of more than one record.

### SOCK_DGRAM

The SOCK_DGRAM socket type supports connectionless data transfer which is not necessarily acknowledged or reliable. Datagrams may be sent to the address specified (possibly broadcast or multicast) in each output operations. They may be received from multiple sources, with the source address available to the receiving socket. The application may also pre-specify a peer address, in which case only datagrams from that peer are received.

Datagrams must be sent and received in a single output or input operation. The output may be buffered, however the implementation should attempt to detect transmission errors.

### SOCK_RAW

The SOCK_RAW socket type is similar to the SOCK_DGRAM type. It is normally used with communication providers that underlie those used for other types. The creation of a SOCK_RAW socket requires appropriate privileges. The format of datagrams sent and received are protocol-specific and implementation-defined.

// wot else goes here m8

Sockets have a non blocking setting, can have an owner, could have queue limits, pending errors which are asynchronous, then receive queue which is buffered and ancillary data etc stuff, signals, connection queue and options and stuff

------lol old stuff here --------------------=====

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
