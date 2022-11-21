---
title: "An odd piece of hardware"
author: Tristan B. Kildaire
date: 2020-12-03
---

**TODO**: Fix date

# Introduction

{{<bruh>}}
<img src="/img/an_odd_piece_of_hardware/first_pci.jpg" width=320 height=426.667 style="float:right;gap">
{{</bruh>}}

If you have seen any of my previous blog posts about the networking equipment I have acquired and later used in my subsequent blogs for different parts of my networking projects or _home labbing experiments_ then you would know that my local ISP here in Worcester is very keen on giving away free hardware that is no longer in use. There's actually an incentive to do so than, contrary to what you'd think. The reason for this is that doing so means that they need not pay for e-waste companies to come and collect what is essentially throwaway hardware in a trash pile.

On that idea ofd _"trash"_, most of the hardware is by no means broken **at all**! A lot of it works perfectly. I spoke with the owner of the business and he told me, for example, that the switches were thrown away as they were too slow. This is referring to a coupled of 16-port Ethernet switches that ran at 10/100 megabit. Obviously implying they switched to something higher for better capacity, such as gigabit or 1/10 gibabit perhaps. Knowing this made me feel good to know that this hardware wasn't at all faulty but at the same time it is sad it would be so easily thrown away, a switch perfect for homelabbing - atleast with the stuff I do - I don't need anymore speed for the mean time, like 100 megabit switch will give me about 12.5 megabytes per second if one converts from bits per second to 8 bit groupings (bytes) per second. The reason I will never really come to that cap is more to do so with my home internet being slow, so I doubt I'd ever push that upstream - locally perhaps for a file transfer it would be nice to have an upgrade. Thinking of this, I wish it were advertised in bytes per second, cause 1 gigabit sounds alluring but is **NOT** a gigabyte per second - rather 125 megabytes per second - that is pretty nice. In any case, that was a bit of an aside - the point is the hardware is good!

# A weird piece of tech

Let's now talk about _what the heck this thing **even is**_.

## What I thought it was

{{<bruh>}}
<img src="/img/an_odd_piece_of_hardware/pci_card_3_ethernet_ports.jpg" width=320 height=568.883 style="float:right;gap">
{{</bruh>}}

This now brings me to the subject of today's blog. *This weird PCI card*.

Now let me start of buy saying that when I saw this thing I thought I scored a jackpot of sorts. In the sense that I had obviously found Ethernet PCI cards in the trash pile at this ISPs backrooms many times, whether they worked may be a different story (also I had a mix from another business where I **payed** to take back old hardware). So when I saw this I thought, wow, three Ethernet ports on this bad boy? Surely it must be some sort of server-grade (enterprise) card. I mean it wasn't the _most amazing_ find ever but it still was something different that I would have loved to take a hold of all its potential - knowing that Ethernet interfaces are few and far between in terms of working and atleast being 100mbit-capable (you must know I have even scraped a frikken coaxial-based Ethernet card before, just for some context).

{{<bruh>}}
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

{{</bruh>}}

## What it _was_

Now this is quite the fun story I tell my techy friends, simply because it's such a coincidence how this all went about. Long story short - I thought that this card was just a broken Ethernet card for servers. I left it plugged into my server at home as I was _then_ (after installing it) too lazy to remove it from the server. Reason being, I am lazy when it comes to that sort of maintenance as it would mean powering down the node, lugging a heavy case down the rack, removing it and then putting it all back and up on the rack again - nevermind the downtime that would have been incurred by such a task - something I wish to keep to a minimum (load shedding doesn't help.)