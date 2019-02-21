---
title: "Playing with ICMP and NAT"
categories: networking protocols icmp nat
---

ICMP stands for Internet Control Message Protocol. It is used by network devices to send error messages and operational information. If you've ever used `ping` to poke a machine, you've used the ICMP protocol!

The ICMP protocol is very simple. It is considered a layer 3 protocol, and messages are wrapped by IP. The header consists of several fields. The first byte is a numeric type. For example, 8 is an IPv4 echo request, while 0 is an IPv4 echo reply. Next is a code that expands on the type, then a checksum and finally 4 bytes that depend on the type/code. For ping, these four bytes consist of an identifier and a sequence number. After the header there may be a payload.

Here's an example IPv4 echo request from my machine (the MAC addresses in the wireless frame have been zeroed out):

```
23:32:45.397131 IP 192.168.0.194 > 167.99.46.15: ICMP echo request, id 17076, seq 1, length 64
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0054 d0b9 4000 4001 d312 c0a8 00c2 a763  .T..@.@........c
	0x0020:  2e0f 0800 271d 42b4 0001 1def 695c 0000  ....'.B.....i\..
	0x0030:  0000 420f 0600 0000 0000 1011 1213 1415  ..B.............
	0x0040:  1617 1819 1a1b 1c1d 1e1f 2021 2223 2425  ...........!"#$%
	0x0050:  2627 2829 2a2b 2c2d 2e2f 3031 3233 3435  &'()*+,-./012345
	0x0060:  3637
```

I captured this by running `sudo tcpdump -XX -i wlp2s0 icmp -n` in one terminal, and running `ping 167.99.46.15` in another.

We can see the IPv4 header start at address 0x000e (the second last byte of the first line), with the version (4) and minimum IHL of 5. We can also see the protocol at byte 0x17, which is 01 for ICMP. The ICMP header begins at 0x0022. We have the type (08), code (00), checksum (271d), identifier (42b4, or 17076 in decimal), and sequence number (0001). The payload varies, but in this case it looks like it begins with a timestamp (`struct timeval`). After that there are incrementing bytes starting from 0x10 to fill up the rest of the payload.

In return for this packet I received a nearly identical packet; only the source and destination addresses were switched, and the ICMP type was 1 instead of 8!

On my server, the source address on the incoming ICMP echo request was different. Instead of 192.168.0.194, which is a private IP address on my home network, it was my external IP address. Let's pretend my external IP is 8.8.8.8. Like most home networks, mine uses NAT (Network Address Translation) to hide several machines behind a single IP address. So the ICMP packet is altered on its way to the server, and again on its way back.

Here's what my server received:

```
23:32:45.409248 IP 8.8.8.8 > 167.99.46.15: ICMP echo request, id 17076, seq 1, length 64
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  .,L.....9..}..E.
	0x0010:  0054 d0b9 4000 3501 3016 0808 0808 a763  .T..@.5.0.P....c
	0x0020:  2e0f 0800 271d 42b4 0001 1def 695c 0000  ....'.B.....i\..
	0x0030:  0000 420f 0600 0000 0000 1011 1213 1415  ..B.............
	0x0040:  1617 1819 1a1b 1c1d 1e1f 2021 2223 2425  ...........!"#$%
	0x0050:  2627 2829 2a2b 2c2d 2e2f 3031 3233 3435  &'()*+,-./012345
	0x0060:  3637
```

The request makes its way to the server by being "translated" at each NAT level. The gateway router remembers the source address and port, and maps it to the egress address and port. For ICMP, the identifier is used as the port. So ignoring the two levels of NAT for now, in this example the gateway router translated 192.168.0.194:17076 to 8.8.8.8:17076. My server sent the response to 8.8.8.8:17076, and this was translated back correctly to allow the packet to end up at my laptop.

What if I want to ping my laptop from my server? I can't ping 192.168.0.194, since it's a private address. A simple `ping 8.8.8.8` appears to work, but my laptop never receives the packet. Instead, the gateway router was responding to the requests. Let's try reusing the same identifier from before, and see if this helps. `ping` doesn't support specifying the identifier, so let's use `nping`.

```
sudo nping --icmp -c 1 --icmp-type 8 --icmp-code 0 --source-ip 167.99.46.15 --dest-ip 8.8.8.8 --icmp-id 17076 --icmp-seq 1 --data-string 'server'
```

Running tcpdump on my laptop, I don't see anything. Then again, does it make sense for the gateway router to forward ICMP echo requests to inside its private network? Probably not. Let's try an ICMP echo reply.

From my client, I make an echo request:

```
sudo nping --icmp -c 1 --icmp-type 8 --icmp-code 0 --source-ip 192.168.0.194 --dest-ip 167.99.46.15 --icmp-id 48879 --icmp-seq 1 --data-string 'client'
```

Shortly after that, I send an echo reply from my server:

```
sudo nping --icmp -c 1 --icmp-type 0 --icmp-code 0 --source-ip 167.99.46.15 --dest-ip 8.8.8.8 --icmp-id 48879 --icmp-seq 1 --data-string 'server'
```

The tcpdump running on my client shows the following:

```
23:58:15.375884 IP 192.168.0.194 > 167.99.46.15: ICMP echo request, id 48879, seq 1, length 14
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0022 5988 0000 4001 8a76 c0a8 00c2 a763  ."Y...@..v.....c
	0x0020:  2e0f 0800 fdc8 beef 0001 636c 6965 6e74  ..........client
23:58:15.406577 IP 167.99.46.15 > 192.168.0.194: ICMP echo reply, id 48879, seq 1, length 14
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0022 8d5a 0000 3701 5fa4 a763 2e0f c0a8  .".Z..7._..c....
	0x0020:  00c2 0000 05c9 beef 0001 636c 6965 6e74  ..........client
	0x0030:  0000 0000 0000 0000                      ........
23:58:36.451918 IP 167.99.46.15 > 192.168.0.194: ICMP echo reply, id 48879, seq 1, length 14
	0x0000:  0000 0000 0000 0000 0000 0000 0800 4500  ..............E.
	0x0010:  0022 4324 0000 3701 a9da a763 2e0f c0a8  ."C$..7....c....
	0x0020:  00c2 0000 f5c0 beef 0001 7365 7276 6572  ..........server
	0x0030:  0000 0000 0000 0000                      ........
```

Success! I noticed this would only work for a minute or so. After that, the echo replies would no longer show up at my laptop. So my home router's NAT implementation appears to be fairly aggressive in removing mappings, at least for ICMP echo replies.

I wonder what happens when two clients in a private network using NAT send ICMP echo requests with identical identifiers to the same external host simultaneously?
