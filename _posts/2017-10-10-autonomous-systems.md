---
title: "Internet 101: Autonomous systems"
categories: networking
---

<p class="preface">This article assumes basic knowledge of <a href="{{ site.url }}{% post_url 2017-10-08-ip %}">IP</a> and <a href="{{ site.url }}{% post_url 2017-10-09-cidr %}">CIDR</a>.</p>

The internet is made up of a huge number of networks. These networks are owned by many different entities in many different countries, who each control one or many IP prefixes. The IANA and the internet registries under it allocate the prefixes. But how are these networks identified, and how does traffic flow between them?

An autonomous system is collection of IP prefixes controlled by a single entity. This entity can be an ISP, large company, government or academic institution. The IANA, via the internet registries under it, allocates a unique number (32 bits, used to be 16) to every such entity. This is called an ASN, or autonomous system number.

Traffic flowing from one autonomous system to another will often need to traverse networks owned by neither party. The routers and fiber links that make up these networks aren't free. There may also be legal, operational and regulatory considerations. This results in agreements between autonomous systems, which specify how traffic is allowed to flow between them. For example, a small ISP may pay a large ISP to use its network to reach the rest of the world. Alternatively, a large company may have an agreement to freely exchange traffic with an ISP, but only if that traffic originates or terminates within its network.

The distinction between autonomous systems helps simplify routing. Routing within an AS is controlled by internal routing protocols, and need not affect the way routes are calculated outside the network. Instead, the AS advertises a single routing policy to the rest of the world. The external routing protocol that exchanges these policies between autonomous systems uses the ASN to identify networks.

The original guideline for autonomous systems in [1] explicitly states that an AS should only be used as a representation of a single routing policy. Interestingly, it also highlights the ideal situation of one prefix per AS. Looking at the internet today, some companies advertise hundreds to thousands of prefixes. IPv4 exhaustion has demolished this policy.

[1] *Guidelines for creation, selection, and registration of an Autonomous System (AS)*. RFC 1930, 1996. [https://tools.ietf.org/html/rfc1930](https://tools.ietf.org/html/rfc1930)
