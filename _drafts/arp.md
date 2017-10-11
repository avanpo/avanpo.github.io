---
title: "Networking 101: ARP"
categories: networking protocols
---

The Address Resolution Protocol is responsible for resolving network layer addresses (i.e. a protocol address such as IPv4) into data link layer addresses (i.e. a hardware address such as a MAC). It is confined to a single local network, and does not cross the router boundary.

A machine broadcasts an ARP packet when it wants to find the data link address for a corresponding network layer address. This packet includes the sender hardware and protocol addresses, along with the target protocol address. It also specifies the hardware and protocol types, along with some length information. This request is accepted by all machines on the network.

A machine that owns the target protocol address will respond with a similarly formed unicast packet, including its hardware address. The first machine may then update its cached ARP table with the address mapping. Once the first machine knows the hardware address of the target, it is capable of sending packets of the relevant protocol to the correct machine.

ARP caches are designed to be shortlived, so that machines taken off the network do not remain in the cache forever. When a machine is added to the network, a gratuitous ARP is sometimes sent. This is a broadcast packet whose target protocol address is itself.

ARP is primarily used with IPv4. It has a number of security issues, stemming from the lack of authentication on ARP responses. The protocol itself has no mitigations for these attacks.
