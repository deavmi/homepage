---
title: "IPv6 Babel over LoRa: Part 2"
author: Tristan B. V. Kildaire
date: 2025-02-08
draft: true
---

# Why?

Because in the [last post](../ipv6_over_babel_part1/) we did it and now I want to continue working on it.

# What?

In part 1 we got setup with two RNode's running in their TNC (terminal node controller) mode. We were able to turn these into virtual networking interfaces that would appear on both $node_A$ and $node_b$ as `tnc0`.

## A new topology

What we now want to do is to introduce a third node, $node_c$, which will be placed into the following topology:

![](diagram.png)

What we want to setup this time is the following:

1. Every node from $node_a \space ...\space node_c$ should be running an identical Babe configuration as we showed last time
2. $node_a$ must be in radio range of $node_c$
3. $node_b$ must be in radio range of $node_c$
4. There should be **no** radio overlap between $node_a$ and $node_b$. We don't want overlap as we want to force traffic from $node_a$ to travel _via_ $node_c$ in order to reach $node_b$ **instead** of going directly from $node_a$ to $node_b$.

## Hardware

For this we will be making use of the [LillyGo T3S3](https://www.robotics.org.za/development-boards/esp32-lilygo-boards/H596) for each of our three nodes:

![2025-01-12-16-41-17.jpeg](../assets/2025-01-12-16-41-17.jpeg)

In terms of antennas I will be making use of [these](https://www.robotics.org.za/communication-wireless-Industrial/antenna-866mhz/YN-868MHZ-5DBI) antennas. They seem to perform rather well and are easy to mount on any surface that is magnetic - which will probably make testing easy for accomplishing the topology shown prior.

![2025-01-12-16-41-40.jpeg](../assets/2025-01-12-16-41-40.jpeg)

# Setup

## TNC

Firstly, plugin each RNode and then run the following command. After each successive command unplug the current RNode and plug in the next RNode:

```bash
rnodeconf /dev/ttyACM1 -T --freq 868000000 --bw 250000 --txp 20 --sf 8 --cr 6
```

After this each RNode will be placed in TNC mode.

Next, let us start up our TNCs. Run this on each node:

```bash
sudo ./tncattach /dev/ttyACM1 115200 --ethernet --ll -vvvv
```

## Version of babel

For these tests we were using the `1.13.x` series. Anything where the the $x$ in `1.x.y`  is different is incompatible.

You can build `babeld` with the following:

```bash
sudo apt install build-essential -y

git clone https://github.com/jech/babeld
cd babeld
git submodule init
git submodule update
make
sudo make install
```

## Babel

Firstly, let's revise what the Babel filter should be per each node. This should be saved to a file named `filter.txt`:

```
redistribute local ip 0.0.0.0/0 ge 0 deny
redistribute local ip fd00::/8 ge 64 allow
redistribute local ip ::/0 ge 0 deny
```

Now let us start Babel on $node_a$ and $node_c$ like this:

```bash
sudo babeld -d 5 -H 5 -c filter.txt tnc0
```

We start Babel on $node_b$ like this so that we can expose the control port which `babelweb` will use in order to get routing information from:

```bash
sudo babeld -d 5 -H 5 -c filter.txt -g 3001 tnc0
```

Let us also start `babelweb` on $node_b$ with the following:

```bash
./babelweb2 -static bweb/static/ -node [::1]:3001
```

We can confirm it is running by visiting `http://localhost:8080/` in your web browser.

# Testing

Let's begin testing.

## Assigning some addresses and _making some routes_

Let's assign the following addresses to the `tnc0` interface on each of our respective nodes:

1. $node_a$
    a. `fd20::1`
2. $node_b$
    a. `fd20::2`
3. $node_c$
    a. `fd20::3`
    b. Note: We don't need to do this for forwarding to work, the forwarding address used is the link-local one and that is used for link-layer (MAC) address resolution when constructing the Ethernet frame for forwarding

We can do this easily on each node as such:

```bash
sudo ip addr add <ip> dev lo
```

Yes, I am adding it to `lo` - it doesn't matter. The address just needs to be assigned to _some_ interface in order for it to be assigned to the host (and of course picked up by babel for advertising). **Also**, it is a full `/128` that will be generated. We are not doing any sub-netting at each router where hosts behind it are non-babel nodes - therefore this is perfectly fine for our use case. Our assumption is that each host _is_ also a router.

## Tests

Here we do some tests.

### Test 1

The antennas used were:

* $node_b$ long-pole
* $node_c$ long-pole
* $node_a$ magnetic-base

In this test I, $node_b$ (my laptop), was sitting next to $node_c$ and as we can see from the tests, it appeared that I had direct reachability to both $node_b$ and $node_a$:

![image.png](../assets/image_1736510131867_0.png){:height 379, :width 659}

### Test 2

The antennas used were:

* $node_b$ long-pole
* $node_c$ long-pole
* $node_a$ magnetic-base

Here I got up with my laptop ($node_b$) and then moved to the middle of the house, effectively between both antennas.

We can see a few things start to happen here. The neighbour relationship between my laptop ($node_b$) and $node_c$ stopped, the hellos being sent by either router were not being received. It is via this mechanism that reachability will drop every 5 seconds (when an advertisement is not received within the `hello_interval` then the reachability starts to drop at each time window):

![image.png](../assets/image_1736510330941_0.png){:height 362, :width 659}

We can also see from above (and below) that the new routes for $node_b$ got installed. Meaning that in order for $node_b$ to reach $node_c$ it must first go through $node_a$. In fact we can see this here in the ttl values (purple ones):

![image.png](../assets/image_1736510463846_0.png)

---

It seems the tests were a success! 🎊️

