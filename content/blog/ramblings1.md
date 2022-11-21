---
title: "Ramblings 1: Address-implicit security protocols with subnet support and ACTUALLY secure"
author: Tristan B. Kildaire
date: 2021-06-02
---

Just some ramblings of what I was thinking about today. No project ideas yet but just putting ideas down.


# The problem

Been thinking a lot about things regarding routing and route distribution. I like the ideas of CJDNS in the sense you prove you own your host network route. For Yggdrasil a similiar thing but with subnet support which is awesome! The problem though here is that the hash of the public key is a good start but then use the whole hash, using only first 16 bytes (or 128-bits) such that you can have it fit in the IPv6 address is going to make a collision on the sub-set bytes of the fingerprint (the hash of the public key) a lot easier/more probable.

I want to have something secure and advertise a subnet too. I want to get rid of the need of centralised RPKI. Maybe then redNET actually HAS a future and a purpose.

# Solutions? (1/2)

### Upsides

CJDNS doesn't do this however it does provide an underlying network that is secure that we of course could reuse along with our new, let's call it, IPv7. And then make an address size whereby we use the full hash. So CJDNS does the routing and we then have an upper layer on it that uses a longer form hash (or the whole hash).

### Downsides

I don't like that we have to reinvent the wheel. IPv6 is so nice. I guess nobody really thought of this whilst working on it.

# Solutions? (2/2)

Maybe CJDNS idea for routing and key exchange is good as I think they're using full hashes there atleast or actually whole key (hash was only for IPv6 address). But then we do something where you say who you trust or whatever. I guess that is the manual PKI but it isn't centralised but of course I hate this idea. I want what CJDNS has but more secure than it is nowand with subnet ability and STILL be secure