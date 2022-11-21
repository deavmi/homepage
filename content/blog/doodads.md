---
title: "Some technical doodads"
author: Tristan B. Kildaire
date: 2021-08-01
---

Just some photos of some things that I have connected to my part of the [CRXN](/projects/crxn) inter-network.

# In my room

These are some of the things that I have running in my room.

## My desk

I have one desk where I sit down and study and write and also where I use my Dell XPS.

{{<bruh>}}
<center>
    <img src="/vlogs/1st%20of%20August%202021/DSC00008.JPG"></img>
    <p>Switch for my laptop's desk (also power my antenna, a Mikrotik Groove)</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st%20of%20August%202021/DSC00009.JPG"></img>
    <p>The aforementioned a Mikrotik Groove with an omni-directional antenna attached to it</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st%20of%20August%202021/DSC00011.JPG"></img>
    <p>My desk's setup in its entirety</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <video src="/vlogs/1st%20of%20August%202021/MOV00039.WEBM" controls></video>
</center>
{{</bruh>}}

## My computer desk

I have another desk for my desktop where I have a few doodads too.

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00012.JPG"></img>
    <p>Desktop ethernet switch that connects to the one in the garage so I can directly access stuff on that LAN from here if I want</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00013.JPG"></img>
    <img src="/vlogs/1st of August 2021/DSC00014.JPG"></img>
    <p>Currently a link from my home network switch connects here to both my IPv6 Mikrotik Router and also my Raspberry PI (for my personal web server)</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00015.JPG"></img>
    <img src="/vlogs/1st of August 2021/DSC00017.JPG"></img>
    <p>My IPv6 Mikrotik router</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00018.JPG"></img>
    <p>My personal Pi web server and quasselcore</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00019.JPG"></img>
    <p>The main switch that enters into my room, it's a Mikrotik Switch that can <b>only</b> run SwitchOS</p>
</center>
{{</bruh>}}

# In the network lab

We now enter my network lab where I give you a full tour of everything I have running there.

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00023.JPG"></img>
    <img src="/vlogs/1st of August 2021/DSC00034.JPG"></img>
    <img src="/vlogs/1st of August 2021/DSC00035.JPG"></img>
    <p>Two of my Mikrotik RouterBOARDs that I currently run OSPF on for CRXN, in the future I aim to run BGP on these for CRXN as well for those who want to connect using their Mikrotik's or protocols such as GRE and others</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00023.JPG"></img>
    <img src="/vlogs/1st of August 2021/DSC00026.JPG"></img>
    <p>Here is my full rack. It contains quite a nnumber of components, from Raspberry Pis, to x86 AT towers, Mikrotik gear and also networking Ethernet switches and battery backup too (there's also a screen there)</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00023.JPG"></img>
    <p>These 3 towers are where I run many services on my network. The rightmost machine runs my <code>lockdown.bnet</code> IRC node for <a href="/projects/bonobonet">BonoboNET</a>. It also runs my Mumble server.
    
    The middlemost machine runs the BIRD Internet Routing Daemon and connects my CRXN network here to routers (and therefore other CRXN networks) in the UK (2 of them) and one in the US.

    The leftmost machine runs a public CJDNS peer, Yggdrasil peer and an IP2P Floodfill router.
    </p>
</center>
{{</bruh>}}



{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00036.JPG"></img>
    <p>I run a small Pi here that runs Munin and logs information on all the nodes on CRXN (well many of them, not all, mostly mine). It even contacts a node in Lebanon!
    </p>
</center>
{{</bruh>}}



{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00029.JPG"></img>
    <img src="/vlogs/1st of August 2021/DSC00037.JPG"></img>
    <p>The main ethernet switch (which connects to the one in my room (same size)) that connects all the machines here to the Internet
    </p>

    <img src="/vlogs/1st of August 2021/DSC00038.JPG"></img>
    <p>
    There's also power supply here
    </p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00030.JPG"></img>
    <p>These are some of the main CRXN routers on my network with many peerings on them. The one in the middle has the most of CRXN peerings and is connected to my home network. The one at the back just connects to seperate CRXN networks (the ones for the x86 machines and another Pi) to the home network )by routing between them).

    The Pi right at the front is my I2PD services node which makes the BonoboNET server available over I2P.
    </p>

    <img src="/vlogs/1st of August 2021/DSC00031.JPG"></img>
    <p>These are the two switches for the two seperate CRXN networks (the one on the left is for an "in-between" link for the two CRXN routers, the other one is for the x86 machines).
    </p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <img src="/vlogs/1st of August 2021/DSC00033.JPG"></img>
    <p>This is the battery backup for all the Raspberry Pis, the two Mikrotik and the Ethernet switch (the main one)
    </p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
    <video src="/vlogs/1st%20of%20August%202021/MOV00041.WEBM" controls></video>
</center>
{{</bruh>}}

# Random-Rany

{{<bruh>}}
<center>
    <video src="/vlogs/1st%20of%20August%202021/MOV00042.WEBM" controls></video>
</center>
{{</bruh>}}