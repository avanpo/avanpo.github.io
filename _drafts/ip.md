---
title: What is IP?
categories: networking, protocols
---

The Internet Protocol is basically responsible for establishing a way for data to be routed across network boundaries. How does it do this?

### Addressing

To route data from one machine to another, it's necessary to uniquely&ast; and deterministically&ast; identify the source and destination machine (&ast;*for some definitions of these words*). IP does this by defining a numerical IP address. For IPv4, this is a 32 bit number (e.g. 172.16.254.1). Since the number of possible IPv4 addresses is grossly inefficient, IPv6 defines a 128 bit number. IPv6 addresses are typically shown using eight four-digit hexadecimal numbers delimited by colons.

These addresses are managed by the IANA and five regional internet registries. Ultimately, an ISP ends up with a pool of IP addresses, and can assign them to customers as they please.

### Packets

When routing data across networks, each node must be aware of the destination address. This, combined with a need to manage variable length data, leads to the definition of a packet. A packet encapsulates a segment of data by adding an IP header.

This header contains the source and destination address, in addition to a number of other fields such as the IP protocol version, the transport layer protocol, and packet length. Notably, IPv6 greatly simplifies the header format. Neither version has a checksum for the payload, although IPv4 includes a checksum for the header.

Both versions of IP specify the length of the packet as a 16 bit number, thereby limiting the packet length to 65535 octets. IPv6 does specify a jumbo payload option, but this is not typically used as it isn't compatible with TCP or UDP (common transport layer protocols).

### Routing
