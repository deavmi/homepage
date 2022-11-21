---
title: " I've been super busy as of late - A quick catch up"
author: Tristan B. Kildaire
date: 2020-07-23
---

Damn, it's been a really busy last few months with this whole
statist lockdown in South Africa. Now to get the politics out of the
way - I hate lockdown but damn has it made me productive. It's in
this post that I hope to shed some light on absolutely everything I
have been working on in the last few months (which damn has been a
lot). I'll be going through a lot of what life has been like here,
what my projects have developed into and the new ones I have created
and then lastly the side-projects of mine (the physical things) that
I have been working on as of late. I'm writing this in the backseat
of my parents' car as we drive down to Stellenbosch to my university
to pickup the leftover belongings of mine that were left there the
day I left home due to the university closing due to the 'rona.<br>

{{<bruh>}}
<center>
<img src="/img/catch_up_1/1.jpg" alt="Photo of me in the backseat of a car" height="301" width="402">
<p>Me in the car's backseat (yes, I still don't have a license)</p>
</center>
{{</bruh>}}

_So let's get to it then!_

# A life catchup

Life under lockdown sucks, I mean other than it being an
infringement on my rights (and no I don't care about the
constitution's exceptions made for "disaster management"), it has
been really shitty. First of all my birthday will have to be moved
possibly because how else are we going to get wine then unless we
wait for cunt-worthy regulations (those banning alcahol) to be
lifted. The second being that I cannot enjoy campus life as well -
this being a centerpiece of what it means to be a part of
university culture - this time is stolen from me and I will never
get it back - yes, I'm looking at you Stellenbosch Universiteit -
y'all really look like damn asses in this situations and it is
nothing to be proud of. SO it'ss a whole fuck up from the state
level down to the University's administrators which yes can do
what they want if they really wanted but their choice was lame -
rather have it open in protest. So now we had to go online and
have tests online and all of that - now the semester went well - I
got good marks and now I'm on holiday **but still** I would
have preferred being on campus this whole time. Now looking back
at it, after how productive I have been, oerhaps not - I am
conflicted but what I am assured is that the next semester should
be on campus - I have gotten my personal project work that I
wanted to cpmplete done, so now let me go back - I'll have no
benefit that I did above here by being at home, so I think then my
above protest can be applied for the next semester - it's valid
now!

So what did I get up to? Well, a lot of things, CRXN, BonoboNET,
Bester, libutterfly, bformat, skoenlapper, Butterflyd and many
more! I'll describe a little bit of what I worked on during such a
productive time in the next section but first - a weather report.

It's been frikken freezing here in Worcester. Damn, the mountain
tops were snowy-as-fuck (saf) and it was super cold but it made
the inside of the house very nice and cozy so I enjoyed that
aspect of it - in front of the fireplace.

{{<bruh>}}
<center>
<img src="/img/catch_up_1/2.png" alt="Photo of my residence desktop screen with firefox, gnome-terminal and quassel irc open" height="510" width="907">
<p>Me in the car's backseat (yes, I still don't have a license)</p>
</center>
{{</bruh>}}

Photo of my residence desktop screen with firefox, gnome-terminal
and quassel irc open, just after arriving at res.

{{<bruh>}}
<center>
<img src="/img/catch_up_1/3.jpg" alt="Photo of mountain in Pniel" height="722" width="964">
<p>Photo of mountains in Pniel (just outside of Stellenbosch)</p>
</center>
{{</bruh>}}

Now that I am back home from collecting my stuff at Stellenbosch,
let's get to it (politics time is over).

## [BonoboNET](/projects/bonobonet) - the best irc network

Something I have always wanted to do was to setup an IRC network.
The main reasons was because it is easy to do and it is a cool thing
to have - I mean chat - **that's super cool!** I had played
around with the IRC daemon our network uses, _ngircd_, a few
years prior and it was hella easy to get setup with 3 nodes so from
there I knew it would be a doable project in a short amount of time.
So I got a few things setup. I setup an original node called _corona.bnet_,
because corona is a meme. That machine had Yggdrasil on it so that
it could be reachable on a network other than my home IP subnet
(which is unreachable by everyone but me on it (and my family of
course)). I got some people to come along and connect but I think
that was only later, after I had moved the job from that machine to
my new machine - _lockdown.bnet_ (and so we say farewell to
corona.bnet - the O.G. BNET node!). That machine stayed in place on
my server rack in the form of a Raspberry Pi - then a few users
started to come along and join in. Next thing you know, I was
starting to get people not just on the network but also peering them
server-wise as new nodes onto the network making it an actual
network not just a one node (lockdown.bnet here) IRC network.

What happened next was what we will call _the great split_
whereby we basically had some folk move over to charybdis (puke mode
activated). However, I got some other people to fill their spot
server wise - we're still good friends (laughing emoji). So now
we're on the node count we were previously with one node more!

Setting it up was and still is a very fun past time. It's fun to see
the different ways we can make connections with other nodes, whether
it be over the regular internet, over CRXN, Yggdrasil or more! It
makes thee already interesting topology even more interesting and
having something that we can use to communicate that we, the
community, put together it very cool!

{{<bruh>}}
<center>
<img src="/img/catch_up_1/4.png" alt="Topology of BonoboNET" height="505" width="714">
<p>Topology of BonoboNET</p>
</center>
{{</bruh>}}

So BonoboNET is a very fun project because, as I will mention next
with CRXN, it is nice to, once you have an IP internet, have
something to actually run on top of it and use it for.

**If you're interested then following this [link](/projects/bonobonet) where\
you can get connected and also, if you want to join your ngircd server onto the
network.

## [CRXN](/projects/crxn) - the best IP inter-network<br>

Something I have been planning for a **very long time** is to
build my own IP inter-network after all of the self-teaching I have
done for IP networking and networking in general. I wanted to have
something whereby I could setup routing myself, configure the routes
then the IP addresses and then the physical aspects like switches
and ethernet cables and so on and so on. That came to life during
lockdown. I set a day aside to configure my network. I decided I now
could put my 3 Raspberry Pis (I now have 5) to good use. I setup the
two to both be routers, and placed a switch in between them and then
one on either side (so three in total).


{{<bruh>}}
<center>
<img src="/img/catch_up_1/5.jpg" alt="" height="631" width="843">
<p>Switch 1 -- Router 1 (**Pi3**) -- Switch 2 -- Router 2 (**Pi1**) -- Switch 3</p>
<p>The middle node is actually my <i>lockdown.bnet</i> BonoboNET node.</p>
</center>
{{</bruh>}}

You can't see the two switches in the above photo but we will
get to them later. So, as I was saying, I went ahead and
configured the whole thing. Got subnets assigned both sides (so
routes added), enabled forwarding and then placed a machine on
either end and did a test ping, and not much to my surprise **it

worked!** (As I had done this before). Now over the time I
would come to realize that I need to get other people on this
network to make it more interesting.

Now, I wanted to get more things and people onto CRXN really
badly - so I did what I knew best - **spam it everywhere!**
I first did a test run of using Yggdrasil's Crypto-Key routing,
which let's you tunnel traffic from one node to another such
that it pops up on the tun0 interface on the destination node's
side - ready for accepting into the kernel's networking stack. I
did that between a machine of mine and a VM (the final time, as
I had done earlier tests but this was a working one - working
properly that is). I saw that that was working and then came the
first interesting part to the addition of new routes into the
inter-network, Chris's IPv6 NETMAPPING router.

[Chris](http://chrisnew.de) and I linked up and I
added a route on my side for 10.5.0.1/32 to point to his
Yggdrasil node so he could do NETMAP-NAT, which would mean he
could make the whole of CRXN available, through NAT, on the IPv6
Internet. This means any traffic coming in would be an IPv6
packet that then would strip it into payload and place in in an
IPv4 packet with the source IP of 10.5.0.1 and then the
destination IP that is a result of the mapping function of
getIPv4(IPv6In) - something that [Chris](https://blog.chrisnew.de/2020/06/subsetting-crxn-into-the-real-internet/).
defined his side. This is a neat feature to have on the
network as now we have every device available and through the
best type of NAT - one that allows raw sockets, so instead of
port-mapping we have address mapping!

After getting the hang f Yggdrasil's CKR I started to get a lot
of people on the network by this method (and still have many
more who want to join it - so much so I should start thinking
about network structuring at this point). Another method I
should look to in the future is using CJDNS for this as well -
it too has a CKR feature which I can use for those who want to
get peered with CJDNS or for the same Yggdrasil peerings I have
to have a redundant backup! Another thing that would be nice
would be to see Lokinet introducing CKR at some point in time -
that would mean I would have 3 methods of tunneling available
that I could use for those who lack the other overlay networks
or to increase redundancy between existing links.

So now for a deep dive into what the network consists of my
side, **routers**, **switches**, **hosts** and a lot
of **wires**!

{{<bruh>}}
<center>
<img src="/img/catch_up_1/16.png" alt="A network map of the whole known CRXN (omitting some personal devices)" height="514" width="667">
<p>A network map of the whole known CRXN (omitting some personal devices)</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/6.jpg" alt="Home ethernet switch (where my network stuff begins - from my room)" height="717" width="958">
<p>Home ethernet switch (where my network stuff begins - from my room)</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/8.jpg" alt="The extension of the switch
inside, this is just for more ports on the same ethernet
network - that's all - but located in the garage rather so
my machines can be connected to it instead of all back to my
bedroom's switch which would be messy" height="753"
width="1006">
<p>The extension of the switch inside, this is just for more
ports on the same ethernet network - that's all<br>
- but located in the garage rather so my machines can be
connected to it instead of all back to my<br>
bedroom's switch which would be messy<br></p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/9.jpg" alt="Another view of my ethernet
switch (I actually got it for free at a &quot;dumpster
dive&quot; (computers being sold that were old))"
height="734" width="979">
<p>Ethernet coupler used to connect the lab's ethernet switch to my room's ethernet switch (as the cable isn't long enough)</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/7.jpg" alt="Ethernet coupler" height="751" width="1002">
<p>Ethernet coupler used to connect the lab's ethernet switch to my room's ethernet switch (as the cable isn't long enough)</p>
</center>
{{</bruh>}}

Another view of my ethernet switch (I actually got it for free
at a "dumpster dive" (computers being sold that were old))

{{<bruh>}}
<center>
<img src="/img/catch_up_1/11.jpg" alt="This is actually my first 16-port switch" height="761" width="1015">
<p>Haven't populated it all yet but you can see it's already filling up a lot</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/12.jpg" alt="The other (the other one is the one above) two switches mentioned in the network topology" height="786" width="1049">
<p>The other (the other one is the one above) two switches mentioned in the network topology</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/13.jpg" alt="Ethernet switch 3" height="790" width="1054">
<p>Ethernet switch 3</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/14.jpg" alt="Ethernet switch 2" height="754" width="1006">
<p>Ethernet switch 2</p>
</center>
{{</bruh>}}

Now what am I running on CRXN? Well, the next machines you
will see, _silverfish_ (_bernd_), _brink_,
_jaco_ and _bester_ are used for various
different things. Currently I run on the network services
for public consumption on _bester_, this includes a
KiwiIRC instance for web irc access to BonoboNET, ssh-chat
(a shell server where the shell is a chat program), a few
matterbridge instances to bridge BonoboNET to what will be a
volatile IRC joining (on a few channels), a gitea instance
(which I use for git mirroring), a redmine instance (for
managing projects and media files for project notes and such
type of documents that aren't suitable for a repository). _Silverfish

_or _bernd_ I use for software testing for
Butterfly and such with several docker containers, _jaco_
- much of the same. _Brink_, nice guy, but he ain't
doing much right now (I mean damn that thing has a strapped
on hard disk drive using cable ties - you don't get more
ghetto than that).


{{<bruh>}}
<center>
<img src="/img/catch_up_1/15.jpg" alt="All of my machines, from left to right - Silverfish (Bernd), Brink, Jaco, Bester" height="652" width="870">
<p>All of my machines, from left to right - Silverfish
(Bernd), Brink, Jaco, Bester</p>
</center>
{{</bruh>}}

And a few more close ups whereby you can see how well I have labeled everything.

{{<bruh>}}
<center>
<img src="/img/catch_up_1/16.jpg" alt="Silverfish (Bernd)" height="968" width="1291">
<p>Silverfish (Bernd) - I wanted more blinky lights on
the CRXN side so I added more ethernet but considering
the way ARP works, one is always useless<br>
and only sends out periodic traffic - I might do some
batman stuff on it though - so no more CRXN for it!<br></p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/17.jpg" alt="The other machines" height="1000" width="1334">
<p>Brink, Jaco and Bester (sitting in a tree, well, umh a
shelf) - I like this scene very much, not what I just
described<br>
but you know that one meme...<br></p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/18.jpg" alt="To power all of these beasts (albeit not at the same time)" height="904" width="1206">
<p>To power all of these beasts (albeit not at the same time)</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/19.jpg" alt="Laptop for managing the machines indirectly" height="970" width="1294">
<p>Laptop for managing the machines indirectly</p>
</center>
{{</bruh>}}

{{<bruh>}}
<center>
<img src="/img/catch_up_1/20.jpg" alt="Monitor for direct attachment to machines" height="908" width="1211">
<p>Monitor for direct attachment to machines</p>
</center>
{{</bruh>}}

And that concludes CRXN stuffies! It also concludes
part 1 of this blog as I have written a lot and want
to relax now, so stay tuned for tomorrow evening
when I publish part 2!