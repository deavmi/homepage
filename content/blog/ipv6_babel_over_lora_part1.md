---
title: IPv6 Babel over LoRa: Part 1
author: Tristan B. V. Kildaire
date: 2025-01-13
draft: true
---

# Why?

![](ipv6_lora_part1/gfy.jpeg)

Too many times stupid questions like these are posed by _weak betas_ - I will not entertain such silly questions.

I'm doing it because _unlike the detractors it's_:

* Fun
* You learn something
* I'm not a party pooper 💩️

# What?

Let's specify specifically what it is that we will be attempting today. In my [previous blog post]() [TODO Add link here (to previous blog post)] we got an IPv6 link working over LoRa using our RNodes and a program called `tncattach`.

## Where does babel fit in?

Once we have a working link over which IPv6 traffic can flow and of which the network interfaces on both sides have an IPv6 link-local address assigned, we can then make use of a program called `babeld`.

Say now you have a a few routes in your IPv6 routing table like:

![](ipv6_lora_part1/routes.png)

Now how would you distribute these routes to  _other_ nodes? Well, that's exactly what Babel allows us to do. It runs as a daemon on all of your nodes you wish to have _receive_ routing information and also _distribute out_ (their) routing information.

## How?

We will be setting up an IPv6-routed network between two nodes, node $A$ and node $B$. We will then be able to have routes shared between these nodes via Babel, this should allow us to ping one another once the routing information has been successfully shared.

# Setting up

## Preparing our TNC

If you followed my second part blog post on setting up your RNode to be a TNC or _terminal node controller_ with `tncattach` then you will understand what the this part is about. TODO Add link here (to previous blog post)

Firstly, ensure that your device is in the TNC mode with `rnodeconf`:

```bash
rnodeconf /dev/ttyACM1 -T --freq 868000000 --bw 250000 --txp 20 --sf 8 --cr 6
```

This is to be run on **all devices**.

We can now move onto the next section.

## Setting up our TNC

We now need to get the TNC up and running on both hosts, to do so we run the following command on host $A$ and host $B$:

```bash
sudo ./tncattach /dev/ttyACM1 115200 --ethernet --ll -vvvv
```

Note that we pass the `--ll` option. If you recall from our previous endeavour on getting IPv6 working over LoRa, this is a required option as it sets the MTU to a value of 1280 if it is not already greater than or equal to that value. This is important as on Linux such a constraint on the MTU is required if IPv6 is to be enabled on the interface.

Running `ifconfig tnc0` on host $A$ now we get:

```
tnc0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1280
        inet6 fe80::6481:dfff:fec1:ec6c  prefixlen 64  scopeid 0x20<link>
        ether 06:0d:f2:a1:72:16  txqueuelen 10  (Ethernet)
        RX packets 460  bytes 49615 (48.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 795  bytes 129308 (126.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

>Note: This link-local is for illustration purposes **only**; it changes later on as I tore down the `tnc0` interface a few times

And on host $B$:

```
tnc0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1280
        inet6 fe80::88c:65ff:feac:5a7  prefixlen 64  scopeid 0x20<link>
        ether 0e:33:83:f5:61:88  txqueuelen 10  (Ethernet)
        RX packets 749  bytes 115611 (115.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 831  bytes 299501 (299.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

>Note: This link-local is for illustration purposes **only**; it changes later on as I tore down the `tnc0` interface a few times

Therefore we can see the following IPv6 information:

1. host $A$ has an IPv6 link-local address of `fe80::6481:dfff:fec1:ec6c`
2. host $B$ has an IPv6 link-local address of `fe80::88c:65ff:feac:5a7`

