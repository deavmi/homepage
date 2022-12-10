---
title: Switch to IPv6!
author: Tristan B. Velloza Kildaire
date: 2022-12-10
draft: true
---

# Why?

Well, this blog post should give you an idea as to why IPv6 is better than IPv4 and not just because of what you might be expecting, the cliche _"IPv6 has more addresses than IPv4"_ but more than that. There are indeed several more aspects to IPv6 that it simply does better than IPv4 ever could and I hope to convince you of those features in an effort to evangelize you.

## Link-local

### Example 1: On the LAN

In the world of IPv4 you may have, if you do anything with routing at all, come across scenarios where you just want to have simple communication between device on the same Ethernet LAN but may have not setup anything further than a simple Ethernet connection between said two devices on an Ethernet switch. Now, as we know, Ethernet is automatic mainly due to the fact that every Ethernet card comes hard-coded with a MAC address meaning that two parties using an Ethernet card already have all they need in lorder to communicate using a `(souyrceMAC, destinationMAC)` format. However, we know that Ethernet does only provide a carrier for payloads and nothing more such as service de-multiplexing (which IP does with the payload type field) or guaranteed delivery which TCP does when loaded into the payload section of an IP packet. Therefore if one were to try and accomplish this you would need to assign IP addresses and routing information to both machines in a manner such as:

```bash
# On machine 1
sudo ifconfig eth0 192.168.1.1/24

# On machine 2
sudo ifconfig eth0 192.168.1.2/24
```

Provided the same LAN is  connected to by `eth0` on both machines this would then cause the Linux kernel to add a direct route for `192.168.1.0/24` on `eth0`. And voila you now have two things:

1. Routing information
    * The kernel knows that if you want to send anything that falls into the subnet (when masked with `(dest & 255.255.255.0) == 192.168.1.0`) 192.168.1.0 then ARP over `eth0` and find the L2 address of the machine holding `192.162.1.2` (in case of machine 1)
2. Acceptance information
    * The machine knows what its IP is and can respond to ARP requests for it and/or accept (not drop) packets destined to it

Now with link-local you **get this for free** without any configuration. As soon as you up your ethernet interface it will get a link-local address (therefore point 2 is sorted) and it will get a `fe80::/64` route installed via that interface - **no manual configuration required**.

#### Some caveats

The only difference is that now an interface specifier is needed on the address as you can imagine having something like this in your IPv6 routing table:

```
fe80::/64 dev tun0 proto kernel metric 256 pref medium
fe80::/64 dev enp3s0 proto kernel metric 1024 pref medium
```

THis is not actually a case of the kernel wanting to choose one rote over the other (in this case the `tun0` as it has a lower metric). Infact both of these routes are to be used for their respective ethernet segments (one on `tun0` and one on `enp3s0`). This is why we need an extra piece of routing information. So you would ping the address not like this:

```
ping fe80::1
```

But rather like this:

```
ping fe80::1%enp3s0
```

If you were trying to reach a machine with the address of `fe80::1` on the Ethernet LAN connected to `enp3s0`. This is a minor inconvenience as a lot of the communication protocols such as Syncthing will automatically do this for you, it's obviously more work for the end-user if you need to manually specify a host like using SSH but it's minor really - I'm used to it already.

### Example 2: Routing protocols

A lot of the routing protocols which run over IPv4 would require you to assign both an address to the router (such that it can communicate with another router) and a corresponding route on the link. So if you are `10.1.1.1`, then you need at least a `10.1.1.2/32 dev eth0` in your routing table to communicate to machine `10.1.1.2` from `10.1.1.1` over the LAN both of them are connected to lon the first machine's `eth0`. The same applies for the second machine, however it would be `10.1.1.1/32 dev eth0`. Now, this is obvious why it is needed - there is no doubt about that but the amount of times I have to do this is more than for the regular home network because you can imagine how many address and route assignments I need to make when doing tunneled routing - this must be done for all of those interfaces - this becomes a nightmare after sometime.

Well, with link-local the addresses and needed routes are already there - both routing protocols can see this and then make use of said addresses and routing information for the interfaces they want to use (and eventually communicate to routers on the other end with). Such information is then automatically picked up via, for example, IPv6 multicast packets sent out by routing protocols such as Babel.

No more headaches like these!

## Radv and SLAAC

