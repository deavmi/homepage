---
title: "Ramblings 2: Secure address assiginment done right? But not backwards compatible!"
author: Tristan B. Kildaire
date: 2021-06-03
---

So this actually sucks but I guess I2P and other cool networks that are not restrained by their interface, CJDNS's tun interface or Yggdrasil's tun interface. Now what I am going to later describe could be done using CJDNS's underlying network transport layer however I guess I'd rather simply reinvent that wheel _and then_ build ontop of that or use it as the actual network layer too - you can already see the direction this is heading in - I want to create a new network layer protocol.

So what _is_ it then?

# Problems

## IPv6 address length and fingerprints

Just a litle definition, a _fingerprint_ is another word for the hash of a public key.

I basically want to solve what CJDNS tried to solve, and one of those is the automatic IP allocation. Now from the feeling of it, and I am no crypto expert, the way they generate addresses is by generating your RSA keys and then hash the public key twice with `sha512`. So the result is actually fine to be honest. That isn't the problem. If we used that as the address I wouldn't be worrying about security.

Where the problem lies is that the hash is 512 bits but CJDNS only uses the first 16-bytes (128-bits so that they can make it fit into an IPv6 address). Now there's already a problem. The low probability of lack of proof of a clash for sha512 in it's 512-bit-space is a good thing, but if you splice that hash then the likelihood of a hash I feel would have to go up because the hash was designed to work well for that length, not anything less - who knows maybe I am wrong but that was the feeling I got when discussing this with caskd (although we didn't know they hashed it we assumed the keys were used which is actually very shit) but I stand with my take on the matter that using an upper portion like that can increase the likelihood of a clash because we have the _avalanche effect_ (changing on bit makes the hash change completely) but that doesn't imply that if you take a sub-seciton anywhere that it will be as avalanche'd as the full hash. Hopefully now you can see why I think the idea of trying to fit this into IPv6 isn't going to work and no solution using the current technology will work.

> Just to be clear that doesn't mean CJDNS's underlying CryptoAuth transort is bad, from my understanding that uses full keys so that is fine.

Now the next thing that CJDNS doesn't support but Yggdrasil does, is advertising a cyrptographic **subnet** rather than just a **Host network** address. This however, as you can see means that some bits may be used now, a large portion, for anything but not the fingerprint - the less fingerprint in the address the wore the security. Now I haven't read all too much about it but I don't see how one can recall mitigate it that much. However, Yggdrasil is aimed at solving routing in a more effective manner - they only need these keys to hopefully not clash just so they have addresses in the DHT which they use - the security aspect isn't all that important - it's more of a concern on how to quickly get _NodeIDs_ assigned so DHT lookups can be made and the _routing_ (the crux of the Yggdrasil project) can be tested/occur.

### The want for subnets

SO I guess this leads me to my next point, I want to be able to have the full fingerprint **and** extra bits for a subnet and STILL remain secure with a solid full lenght hash of the public key (a solid full fingerprint).

> One note, I know they take a double hash (of the public key and of the resulting hash) but that still doesn't sit all well with me

I expand on this point later.

## Hierachy is fine (for your network)

Now why have subnets at all. Why not take a 512-bit address and stop there because then it will be cryptographically secure. Well I want to be able to mess around with my network in a deterministic way, to know that I can have an address of a form such as:

```
[512-bit fingerprint][512-bit host bits]
```

So everyone can be assured anything past the first 512-bits is my network route and mine only **but** I can also start assigning addresses to the machines within the network address without them having to run this new routing software too **and** I can choose what their address wil be, in terms of the remaining 512-bits.

As you can see it will simulate the network structure of IP but will include key exchange (between routers) basically - much like Yggdrasil and CJDNS but , and _unlike_ CJDNS I can securely announce my host (the first 512-bits in the `[512-bit fingerprint][512-bit host bits]`) and in terms of subnetting support _like_ Yggdrasil but however securely, I can change the remaining 512-bits (the host-bits in `[512-bit fingerprint][512-bit host bits]`) to what ever I want.

> Your home network is assumed to be safe, in terms of assinging IPs to hosts - so those that get to choose their `host-bits` a.k.a. non routers - can either be chosen manually or we can copy SLAAC from IPv6 and use that. We don't need any crypto past the router and to the hosts - the home is our axiom of safety.

---

# That's all for now

I will ponder on this a bit and discuss it with caskd soon too. If we come to a conclusion I will probably make some notes in a next joint-blog post of ideas. Then we can get into how interfacing will work but that is relatively mundane userspace stuff and library wrappers can help such that one can maintain the POSIC standards (not at the syscall level but at the glibc level with the `recv()`, `write()` etc.).

## Second note

Somehow I wrote about this in [Ramblings 1: Address-implicit security protocols with subnet support and ACTUALLY secure](../ramblings1)