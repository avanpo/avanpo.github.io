---
title: "Exploring Facebook's network"
categories: networking
---

What does the network of a big internet company like Facebook look like? I tried
to find out.

A good start is the traceroute tool. It performs reverse DNS lookups for each hop, which often
reveals interesting information. Facebook.com is no exception.

For example, look at this traceroute from my box in Amsterdam:

```
$ traceroute facebook.com
...
4  ae37.pr04.ams2.tfbnw.net (157.240.67.244)  1.469 ms ae37.pr03.ams2.tfbnw.net (157.240.67.242)  1.516 ms  1.447 ms
5  po111.asw02.ams2.tfbnw.net (204.15.22.62)  0.374 ms po131.asw02.ams3.tfbnw.net (173.252.66.38)  0.644 ms po141.asw02.ams3.tfbnw.net (129.134.34.164)  0.566 ms
6  po223.psw02.ams2.tfbnw.net (157.240.47.35)  0.468 ms po244.psw04.ams2.tfbnw.net (129.134.33.189)  0.706 ms po242.psw03.ams2.tfbnw.net (157.240.35.191)  0.830 ms
7  173.252.67.111 (173.252.67.111)  0.858 ms 173.252.67.53 (173.252.67.53)  0.871 ms 157.240.38.129 (157.240.38.129)  0.676 ms
8  edge-star-mini-shv-01-amt2.facebook.com (31.13.64.35)  0.775 ms  0.726 ms  0.572 ms
```

The first name we get is `ae37.pr04.ams2.tfbnw.net`. The domain name stands for
"the Facebook network". The subdomain `ams2` clearly refers to the location.
Airport codes are often used to name devices, and ams happens to be the code
for Schiphol--the largest airport near Amsterdam. `pr02` probably stands for peering
router, which makes sense since that's where our traceroute entered Facebook's
network. I'm not sure what `ae37` stands for, but maybe we'll find out later.

After the peering router, we pass through two layers of devices, the first layer named `po***.asw02` and
the second `po***.psw0*`. These are likely colocated with the peering router. Judging by the
names, they are probably simple layer 3 switches. After that there is another
layer of devices without any PTR records.

Lastly, we end up on `edge-star-mini-shv-01-amt2.facebook.com`, which serves the
IP Facebook's authoritative DNS servers gave us. There's lots of interesting
bits of information to unpack here, but let's start with "edge". 

## The edge

Facebook has a number of large
[datacenters](https://www.facebook.com/PrinevilleDataCenter/), where they store huge amounts of
data and perform massive amounts of computation. Datacenters are big, require a lot of
power, and are very expensive. It's not cost effective to build these in the
middle of a big city. So you build it out in the middle of nowhere, where land
and electricity is cheap. Unfortunately, that means your servers are also far
away from your users, and your users don't like high latency.

The solution here is to have much smaller deployments as close to your users as
possible. For Facebook, this means putting racks of servers in the same building
where they peer with ISPs. In this case, that's in or at least very close to a major city and therefore very close to their users. My traceroute landed on the very edge of their network.

This [presentation](https://www.cs.unc.edu/xcms/wpfiles/50th-symp/Moorthy.pdf) (from 2015) illustrates this concept. They call these deployments edge POPs. According to the presentation, the benefits are two-fold. One, it allows them to terminate TCP and SSL connections much closer to the user, and thereby reduce initial latency. Two, they can cache static content, which probably saves an enormous amount of money in terms of bandwidth costs.

I'd like to do some more traceroutes, Facebook's DNS servers keep sending me to
`ams2` or `amt2`, which are probably two different [colocs](https://en.wikipedia.org/wiki/Colocation_centre). So I'm going to enumerate Facebook's IPv4 space and do reverse DNS lookups.


```python
import ipaddress
import os

# Get all FB IPv4 subnets.
WHOIS_CMD = "whois -h whois.radb.net -- '-i origin AS32934' | grep 'route:' | awk '{print $2}'"

# Do a reverse DNS lookup on the address.
REVERSE_DNS_FORMAT = "dig -x %s +short"

def get_slash24s():
    subnets = []
    for line in os.popen(WHOIS_CMD).read().splitlines():
        subnet = ipaddress.ip_network(line)
        subnets.extend(subnet.subnets(new_prefix=24))
    return subnets

for s in get_slash24s():
    for o in range(0, 0x100):
        ip = s.network_address + o
        name = os.popen(REVERSE_DNS_FORMAT % ip).read().strip()
        if name:
            print("%s %s" % (ip, name))
```

This gave me a nice list of names to peruse. There's a lot of really interesting
stuff in here.

## The backbone

Facebook has multiple datacenters, which are connected by a backbone. According
to an
[article](https://engineering.fb.com/data-center-engineering/building-express-backbone-facebook-s-new-long-haul-network/) from 2017, they actually have two: an express backbone designed for
inter-datacenter traffic, and an old classic backbone which is used for egress
traffic.

We can find many names of the form `ae43.bb01.ams2.tfbnw.net` or `bb01.ams2.tfbnw.net`.

The `bb` almost certainly stands for a backbone router, while the character group
after it looks like an airport code plus number that probably designates a particular [coloc](https://en.wikipedia.org/wiki/Colocation_centre). In particular, the number after the `bb` appears to range from 0 to 1 or 0 to 3 depending on the coloc, suggesting the routers are deployed in pairs.

For every name of the latter form, for most colocs there are a large number of the same but
prefixed with `ae` and a number. I couldn't see an immediate pattern to the
numbering, but I'm guessing this has to do with their express backbone and its
four parallel topologies (which they call "planes").

From the coloc names and a [list of datacenters](https://baxtel.com/data-centers/facebook#locations), we can get an idea of what their backbone topology looks
like. Not all of the coloc names map to well known airport codes, and some do
but are probably wrong---`prn` probably refers to their first
datacenter in Prineville, rather than the airport in Pristina, Kosovo. Most of
the rest are
pretty obvious---for example `eag` likely refers their datacenter in Eagle
Mountain, Utah. I wasn't able to figure out `vab`, and I probably
got a few others wrong. The rest I've
plotted on a map (red indicates a datacenter location):

[img]

It's pretty telling how little backbone presence there is outside the USA and
Europe.

Interestingly, there's three colocs (eag, gig and sac) which don't have any
names prefixed with `ae`. Presumably the express backbone does not extend to
these locations (yet). It's possible these locations are still in the process of
coming online.
