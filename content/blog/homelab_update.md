---
title: Homelab update
author: Tristan B. Kildaire
date: 2022-05-12
---

It's about time I give you guys an update on my homelab seeing as a significant amount of it has changed.

# Begone shelves!

![](/img/homelab_update/DSC00001.JPG)

One of the things that had irked me for a while was the use of shelving as a legitimate
means of holding all of my equipment toghether. A lot of the components simply wasted a
lot of space on the y-axis (for lack of a better description) - perhaps _"depth wise"_
is a better term. A lot of this space could be used for other things (as you will see).

The second point was that the shelving mechanism really made it hard to access all sides
of a lot of the equipment I was using. I couldn't, for example, easily reconnect ethernet
of some components, say now a Pi to the ethernet switch, without having to really work my
fingers around a delicate mess of wires wound on my shelf. To sum up it was not a setup that
was friendly to future expansion and changes.

With this new setup I hope that a lot of those pains will be reduced, as you will see there
are still somethings which can be improved but it is significantly better than what I had
previously.
 
# The new setup

In this section I'll go over all the components used within the new lab setup.

## The big beasts

![](/img/homelab_update/DSC00002.JPG)

So I have obviously decided to still keep the big ATX towers, however now that thjey are kept
on the floor it is a lot easier for me to actually maneuver them and do maintenance on them.

![](/img/homelab_update/DSC00010.JPG)

This completed with the fact that I now have a KVM switch I can easily reinstall these machines
if needs be. I have this connected to two of the three ATX machines. The KVM switch connects
a keyboard via USB and external monitor via VGA. The USB output (emulating a keyboard) is coupled
with a VGA connector which then plugs into each respective ATX machine.

I actually found this KVM switch whilst dumpster diving at my local ISP and I must say, this
works extremely well - not too sure why they threw it out but it doesn't seem broken to me.

### Bernd

![](/img/homelab_update/DSC00003.JPG)

My machine `Bernd` is still running my I2P floodfill router (however still only over IPv6 as I
have no public IPv4). I would like to in the future switch this over to a Pi which can more
than easily perform the same job. The reasoning is twofold. Power consumption would be reduced
significantly and I would also have better stabilility. _Stability_ you ask? Yes, _stability_,
this current machine either is broken or has a dead CMOS battery because it can never power on
after lost AC - and instead of fixing that I have had to have a relative continuously come
and power on the machine whenever load shedding ended - this is not a good way to ensure high
uptime at all hence I may as well move away from this machine or repurpose it for something less
critical than what it is currently doing - being a core Yggdrasil router of which a lot of
services I run rely on.

### Jaco and Bester

![](/img/homelab_update/DSC00004.JPG)

Jaco is my one tunneling CRXN router that is peered on the `community.deavmi.crxn` network.
It's been doing its job well now, I don't expect to remove it any time soon as it is working
perfectly. Next to it is my IRC host for the `lockdown.bnet` BonoboNET node which also hosts
my mumble server as well.

## Power management

![](/img/homelab_update/DSC00005.JPG)
![](/img/homelab_update/DSC00021.JPG)

The power management could be better, I have tried removing all cables from the main _"face"_
of the lab but the cables had to go somewhere and for now that means under the shelf/table.

![](/img/homelab_update/DSC00020.JPG)

I have kept the UPS (Uninterrupted Power Supply) which provides power for the ARM boards
and the Mikrotik (the board mounted vertically on the wooden panel).

## Raspberry Pis

![](/img/homelab_update/DSC00014.JPG)
![](/img/homelab_update/DSC00015.JPG)

I haven't really updated much to do with my two CRXN routers for the `home.deavmi.crxn` and
`community.deavmi.crxn` . They're operating as usual with no major problems to report. I musat
say I am impressed by how these things have held up as routers, they can most definately do
the job very well and allow for easy expansion and customizability, a lot thanks to the BIRD
Internet Routing Daemon and the general fact that these are running Linux which lets you mash
up a lot of different protocols and daemons to get just the right setup that you want.

![](/img/homelab_update/DSC00013.JPG)

I have put toghether a DNS server using Unbound which can resolve some `.crxn` domains I have
setup which point to important hosts for me. I have a script that updates the zone files based
off of a file hosted on a remote git repository. I however, tried setting up unbound with
name server records but that did not work for some odd reason - so for now this will do.

![](/img/homelab_update/DSC00016.JPG)

I also setup another IRC server on another Pi, which hosts an instance of _The Lounge_ which
is a web IRC frontend for users who wish to join via that. This node, like `lockdown.bnet`,
also runs Lokinet such that Lokinet users may access the IRC services on this node. It peers
as well with some servers too as to provide higher redundancy. This node is `worcester.bnet`.

I also added a nice little fan to it via the GPIO pins and a heatsink ontop of the CPU which
is quite neat.

![](/img/homelab_update/DSC00012.JPG)

Mounting the Pis was a walk in the park. All the cases had mounting points (besides one which
a while back we drilled holes into) so mounting this was as easy as using self-tapping screws
as they allow one to use a screw driver to screw them into soft wood such as the one depicted.

## Networking gear

![](/img/homelab_update/DSC00011.JPG)

I have gotten this new Mikrotik RouterBOARD which I aim to use as a replacement for the current
router that routes `community.deavmi.crxn`, the main reason is that it has MANY ethernet
ports which will be useful for future expansion and consolidation. It also can run OpenWRT
and coupling that with the aforementioned multiple ethernet interfaces (onboard as well
as compared to the USB ethernet being used on my Pi) this is a welcomed upgrade.

I would like to, in the future, move away from using Raspberry Pis and x86 machines (ATX towers)
as routers for CRXN and just use Mikrotik hardware but with OpenWRT running on it. I also have
a WiFi radio on this device which I will be using for my Raspberry Pi 2 Zeroes such that they
can gain CRXN connectivity too (more on that in a future post)!

![](/img/homelab_update/DSC00008.JPG)
![](/img/homelab_update/DSC00009.JPG)

As for the rest of the networking gear, not much has changed. I have changed my big 1U switch
to a smaller one but with the same 16 ports.

![](/img/homelab_update/DSC00017.JPG)

The other switches have remained for now as they are compact enough and do their job well. No
need for more ports for these switches seeing what is connected to them currently.

![](/img/homelab_update/DSC00007.JPG)

One thing I did add though was this new Mikrotik managed switch which I would like to setup
bonding with for a failover ethernet connection in the case that does occur. Seeing as there
is only a single ethernet cable travelling from here to my bedroom where the bridgeIPv6 router
is located.

![](/img/homelab_update/DSC00019.JPG)
![](/img/homelab_update/DSC00018.JPG)

Nothing has changed for this Mikrotik RouterBOARD which provides CRXN On-the-go access (meaning
that it basically provides either Yggdrasil or IPv6 clearnet access to CRXN via a Wireguard
tunnel).

---

# A video of it all

This homelab update would not be completed without a terribly exposed video shot on a Sony Cyber-shot!

{{<bruh>}}
<center>
	<video src="/img/homelab_update/lab.webm" controls=true></video>
</center>
{{</bruh>}}