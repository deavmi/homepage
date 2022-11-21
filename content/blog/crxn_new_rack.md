---
title: CRXN updates ahead, new peering and switched to Bird!
author: Tristan B. Kildaire
date: 2021-05-26
---

I've been busy in the past 2 months with exams and now that the first batch is over I have, other than a lot of class work to catch up on,
some time to work on my projects (other than the top secret one, I have taken a hiatus from that and will continue work on it shortly).

# CRXN switches to Bird

In the last month I have aimed to get CRXN switched over to Bird and to IPv6 only. That goal has been reached with only two nodes
(4 in total as the other 2 need to get reconnected) remaining running the old (but still usable with Bird (compatibility wise)) babeld.

The switch to Bird was one that actually started as me just playing around with Bird and wanting to gte acustomed with something
that enterprises used and soon I was able to get one of my nodes running it and it worked! I also had a goal of connecting CTWUG
to CRXN and since they used OSPF I wanted a daemon that could easily run OSPF - that would be Bird. (The plan to get CTWUG connected
seemed to have fallen through - their admins are inactive as hell). Regardless, the knowledge gained from wanting to learn how to
connect tha Babel-based side of routers with thhose that ran OSPF was a useful exercise as a few weeks later some two folks from
Britain had an idea to build a link within their district and we discussed inter-connecting over, you guessed it, OSPF and even with
Mikrotik devices (which I had gotten to work with Bird's OSPF)!

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_89@23-05-2021_17-15-38.jpg">
    <p>The two Mikrotik RouterBOARDs I am using for OSPFv3</p>
</center>
{{</bruh>}}

This gives us an oportunity to really grow the network using Bird as now we have access to many protocols and a really great
filtering mechanism that Bird offers.

### Looking glass support

This upgrade also meant that we could now use those cool looking glass softwares that many exchangs use for inspecting their
Bird instances.

This software let's you get a general overview of all nodes and what protocols are active.

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/Screenshot from 2021-05-26 23-09-05.png">
</center>
{{</bruh>}}

Let's you show all routes known to each node:

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/Screenshot from 2021-05-26 23-09-05.png">
</center>
{{</bruh>}}

Let's you also perform a traceroute to a given address on each of the nodes (this is super useful to see which paths are being
used for traffic going out of a certain node to a certain destination):

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/Screenshot from 2021-05-26 23-15-29.png">
</center>
{{</bruh>}}

> I am aware most nodes are not connecting, I have to check Yggdrasil to see as to why that isn't working - but atleats CRXN **itself** was still routing and forwarding - just the looking glass that was broken

---

# RouterBOARDs on a Board

Seeing now that I got Mikrotik RouterOS's OSPFv3 interoperating with Bird's OSPv3 I now needed a home for these routers and starting with this year I wanted to really neaten up my network setup in the garage to make my lab look way more neater. I decided I would start with the left most part of the lab and put these Mikrotik's on the wall on a piece of pine wood.

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_88@23-05-2021_15-58-02.jpg">
</center>
{{</bruh>}}

---

## The build process

We first had to get a few supplies

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_95@23-05-2021_17-15-38.jpg">
    <p>Some self-tapping screws (the carve a whole as you screw them in)</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_93@23-05-2021_17-15-38.jpg">
    <p>Some pine wood to screw onto, we ended up cutting it in half</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_101@23-05-2021_17-16-10.jpg">
    <p>What the dimensions looked like</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_107@23-05-2021_17-16-10.jpg">
    <p>The mounting was trivial due to the hole mounts of the RouterBOARDs</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_97@23-05-2021_17-15-38.jpg">
    <p>I will be using these slim ethernet cables, they actually turned out better than I thought, way earier than had they been cylindrical</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_112@23-05-2021_17-16-15.jpg">
    <p>Got quite a few of these UGreen cables</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_115@23-05-2021_17-16-15.jpg">
    <p>I don't know man, John China sussy</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_109@23-05-2021_17-16-12.jpg">
    <p>And this is how I <i>clamped</i> the cables down, now you see why it worked out so well</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_108@23-05-2021_17-16-12.jpg">
    <p>This is what the cable management looked like from afar, very neat I must say</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_117@23-05-2021_17-16-15.jpg">
    <img src="/img/crxn_rack/photo_120@23-05-2021_17-16-15.jpg">
    <p>I also ahd to include a switch if things were gonna reach my bigger rack when this was mounted on the wall</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_110@23-05-2021_17-16-12.jpg">
    <p>All put toghether, I few things I could to to make it neater but now everything is powered on</p>
</center>
{{</bruh>}}

---

## Testing the setup

Everything was wokring, well kinda. Broadcast mode it doesn't like, so I set to ptp on both Bird and RouterOS (else the next-hop was fucked to `::` for routes coming in or something hugely fucked - like the routes would get installed but incorrecty with respect to that attribute)

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_129@23-05-2021_17-17-15.jpg">
    <p>Here it was broken, I don't think you can see it or it was in the middle of transitioning from one being active and another to inactive</p>
</center>
{{</bruh>}}

Later, I got it working but only two ptp routers on one subnet else it bounces between the 3.

---

## Installation

Some photos of the installation in the garage next to my rack.

{{<bruh>}}
<center>
    <img src="/img/crxn_rack/photo_130@23-05-2021_17-17-40.jpg">
    <img src="/img/crxn_rack/photo_132@23-05-2021_17-17-40.jpg">
</center>
{{</bruh>}}

And lo and behold, it did actually work! The second router though was only connected to the first one (and then that to my Pi over Bird and through that black Ethernet switch at the bottom of the pine wood).