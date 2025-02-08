---
title: IPv6 Babel over LoRa: Part 1
author: Tristan B. V. Kildaire
date: 2025-02-08
draft: false
---

# Why?

![2024-12-25-20-35-07.jpeg](2024-12-25-20-35-07.jpeg)

Too many times stupid questions like these are posed by _weak betas_ - I will not entertain such silly questions.

I'm doing it because _unlike the detractors it's_:

* Fun
* You learn something
* I'm not a party pooper 💩️

# What?

Let's specify specifically what it is that we will be attempting today. In my previous blog post we got an IPv6 link working over LoRa using our RNodes and a program called `tncattach`.

## Where does babel fit in?

Once we have a working link over which IPv6 traffic can flow and of which the network interfaces on both sides have an IPv6 link-local address assigned, we can then make use of a program called `babeld`.

Say now you have a a few routes in your IPv6 routing table like:

![image.png](image_1735539166009_0.png)

Now how would you distribute these routes to  _other_ nodes? Well, that's exactly what Babel allows us to do. It runs as a daemon on all of your nodes you wish to have _receive_ routing information and also _distribute out_ (their) routing information.

## How?

We will be setting up an IPv6-routed network between two nodes, node $A$ and node $B$. We will then be able to have routes shared between these nodes via Babel, this should allow us to ping one another once the routing information has been successfully shared.

# Setting up

## Preparing our TNC

If you followed my second part blog post on setting up your RNode to be a TNC or _terminal node controller_ with `tncattach` then you will understand what the this part is about.

Firstly, ensure that your device is in the TNC mode with `rnodeconf`:

```bash
rnodeconf /dev/ttyACM1 -T --freq 868000000 --bw 250000 --txp 20 --sf 8 --cr 6
```

>This is to be run on **all devices**. 

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

This means that we have the necessary configuration setup to continue on setting up Babeld. This is because of two things:

1. We need IPv6 working on the interface as that is what we plan to _use_ over the network
2. Secondly, Babel requires IPv6 link-local addresses (and IPv6 in general) in order to send out link-local multicast traffic for the purpose of beaconing and finding other routers on the same network segment. Along with this it also will install routes with the next-hops being that of the corresponding IPv6 link-local address of the neighbouring router.

## Setting up babeld

`babeld` is the routing daemon that implements the [_Babel routing protocol_](http://www.irif.fr/~jch/software/babel/), it's available on almost all Linux distributions these days (including OpenWRT) and is relatively easy to setup.

Recalling that babel is here to help us redistribute routes. Well, we have some questions then:

1. Which routes (from our kernel table) in particular should we redistribute?
2. Should we generate `/128` (or `/32` for IPv4) routes based off of addresses assigned to interfaces?

All of the above (and more) can be controlled via the Babel configuration file.

Firstly, Babel will _by default_ scrape off **all** interface addresses and export them as `/32` routes (for IPv4) and `/128` routes (for IPv6). There isn't a problem really with the latter as that is what we want. However, I know for sure that on my host $B$ it has quite a few IPv4 addresses ob various Docker container `veth`(s); therefore on host $B$ let me add this line:

```
redistribute local ip 0.0.0.0/0 ge 0 deny
```

The `local` keyword refers to the _local address_ redistribution technique described a moment ago.

We then say, any route (the candidates in this case would only be `/32` and `/128`s because of the `local`) which when the mask of `0.0.0.0` (hence the `/0`) is applied to it _equals_ `0.0.0.0` (the network address) will be allowed **if further** it is a route of prefix length 0 or greater. This effectively means all IPv4 `/32` routes will not be redistributed and only those derived from interface addresses.

>Note: There are other more general way to do this for kernel routes and device/local-address routes using the `out` filter but this is easiest for now.

Note, we are about to add more filters. The way it works is top to bottom. If a route is matched then we stop there, apply the action (`allow` or `deny`) and we do not run the filters that appear after/below it.

Now on both hosts I will be assigning addresses to them which we will use for the tests, these addresses will be in the `fd00::/8` range. Meaning that the first 8 bits of any route must equal `fd`. Since I have _other_ IPv6 addresses on host $A$ and $B$ which I don't want to be part of the test we should filter those too:

```
redistribute local ip fd00::/8 ge 64 allow
```

But remember, all that does is match `fd00::/8` routes, any _other_ IPv6 route would not match that clause and by default would have the action of `allow` applied to it. Therefore we must add a rule that will definitely catch remaining IPv6 routes and apply an action of `deny`:

```
redistribute local ip ::/0 ge 0 deny
```

Save the above into a file called `filter.txt` so we can access it later.

Now that we have that setup we can run the `babeld` command:

```bash
sudo babeld -H 5 -d 5 -c filter.txt tnc0
```

The parameters are as follows:

1. `-H <int>`
    a. The hello interval. This is how frequently the daemon should send out announcements.
    b. It must be **the same** across all nodes.
2. `-d <int>`
    a. Sets the verbosity of the debugging output
    b. I set it relatively high so that I can see what is going on at every step of the way
3. `-C <config file path>`
    a. This contains the path to the file containing our Babel configuration.
    b. Hence the `-C filter.txt`
4. `[interface list...]`
    a. The list of interfaces to run the Babel routing protocol on.
    b. Here I have just selected `tnc0` as I only want to run it over one interface.

On host $A$ I am adding the extra flags _before_ the interface list:

```bash
-g 3001
```

This runs a debugging port in _read only_ mode (use a `-G` for write access as well) on `3001`. This will aid us in running a diagnostics tool called `babelweb2` which will let us analyse the state of our Babel network from the point of view of node $A$.

## First test

On host $B$ let's add an address of `fddd::2` to _any_ interface, let's say `lo`:

```bash
sudo ip addr add fddd::2 dev lo
```

Now on host $A$ if we look at _BabelWeb_ we should see it:

![image.png](image_1735477960879_0.png)

We can also see the route we learnt of `fddd::2/128` via our `tnc0` interface:

![image.png](image_1735478038424_0.png)

Show also that the _via_ (next-hop) is a link-local IPv6 address. This is used to make an ICMPv6 neighbour request to ask for the MAC address at that host to then put the IPv6 packet destined to it inside of an Ethernet frame destined to said MAC address.

On host $A$ lets now assign an address of `fddd::3` to any interface, let's say `lo`:

```bash
sudo ip addr add fddd::3 dev lo
```

Likewise on host $B$ we would see a route to `fddd::3/128` via our `tnc0` interface:

![image.png](image_1735478117664_0.png){:height 464, :width 689}

We can even open up Wireshark on host $B$ and attach it to network interface `tnc0` and see we are receiving Babel announcements over IPv6 link-local about routes. Here we can see host $A$ advertising us its route to `fddd::3/128`:

![image.png](image_1735478325974_0.png)

Now if we are on host $B$ we should be able to ping `fddd::3` as it will have a route installed to it:

![image.png](image_1735478483971_0.png)

That is rather reasonable latency too!