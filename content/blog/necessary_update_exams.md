---
title: "A necessary update"
author: Tristan B. Kildaire
date: 2021-06-21
---

{{<bruh>}}
<img src="/img/necessary_update_exams/visage.png" height="170" width="170" style="float:right">
{{</bruh>}}

I have been extremely busy the last few days as the last set of exam work is coming ahead, but I thought I should let you guys know I am still alive and that the projects I am working on are making progress still and that new things are in the works. So I just want to go through some of these.

There will most likely be upcoming blog posts that go more into detail of each of these but this is just a rough summary.

---

## tlang

I have been hard at work on my compiler for my new language I am working on, I don't post much about it as it is a secret project (for now - the end of the year I shall do a release).

I have already (months ago) finished the **lexer**. As for the remaining sections, the **parser** is _"done"_ in the sense I am able to add new features as I want but for the time being it has the features I want it to be able to parse so far. The next section is typechecking which includes not just the checking of types but also the generation of dependency trees which can be used for _further_ typechecking and making sure initializations are **valid** but so far I am using it to generate a tree that I can then follow with the **code generator** and generate the correct initialization code etc.

It is not complete but it can make basic trees and handle mutual recursive cases using visitation marking. There's a lot of reworking that I had done, this is the second rewrite of it but I believe I am now on the right track with the needed tree structure which I call a `DNode` and the accessories they require to operate, such as pooling of nodes to get unique entries (as we compare `DNode`s using addresses hence their inner entity's they are resposible for must map to a unique `DNode` (and if one doesn't exist we create one)).

I have been keeping a log of all the changes I make to tlang as well in a journal that will be exposed at the end of the year. I also have code examples, no EBNF yet but I will derive that from the code examples as I have plenty enough to do so. I might ask the assistance of a friend of mine, [Stephen Cochrane](http://skiqqy.xyz), to assist with that as he is the king of compilers 👑️ and math.

---

{{<bruh>}}
<img src="/projects/crxn/logo.png" width="438" height="282" style="float:right">
{{</bruh>}}

## CRXN

I have been very busy sorting out the documentation which you can now see [here]() - but it took a long time but finally we have a working tutorial from start to finish that assists people in getting connected to CRXN.

We migrated away from the [Netbox instance](http://crxn.chrisnew.de/netbox) we were using for IP address management and are now using a Git repo hosted on [codeberg.org](http://codeberg.org) which we call [EntityDB](https://codeberg.org/CRXN/entitydb), I did this as it was way easier to get patches in than manage user accounts in a Netbox instance. Greater news even is that Rany has taken resposibility for managing the incoming pull requests to that repository which means that some weight (quite a lot actually) has been lifted off of my shoulders.

We also used to have DNS but that will be coming back soon, [caskd](http://redxen.eu) just had to migrate his Hetzner cluster so that will be back in the next few weeks (the documentation will also be updated and you cna be assured I will be making a new blog post about it).

We also moved spaces, we now fully use the `fd00::/8` space and assign `/48`s or _ULAs_ to everyone within that range.

We dropped IPv4 support (let it 🔥️ in hell) - _good riddance_.

I have also been cleaning up the many nodes' configuration files as of late to make things more manageable on my end.

`minus_tech_tips` or Jimmy has also provided updates to the documentation regarding the bird 2.0 docs as they were slightly broken (untested by me), I still need to fix up the bird 1.6 therefore but that will be done most likely this weekend.

---

## New services

I have also started the **Drommedaris Services Project** which you can check out on the [Drommedaris Services Project homepage](http://[2a04:5b81:2010::90]) (sorry IPv6 only - upgrade your internet). This service runs a [I2Pd](../setting_up_an_i2pd_node/) router which helps people circumvent censorship and so far it has been carrying a good amount of traffic.

![](/img/necessary_update_exams/i2pd.png)

It also is acting as a floodfill router which from my understanding means that it helps routers find other routers.

It also runs a public Yggdrasil node available over CRXN, Lokinet and Clearnet IPv6 and you can see a statistics page of it [here](http://[2a04:5b81:2010::90]:81):

![](/img/necessary_update_exams/yggstats.png)

There are various services made available there such as an HTTP Proxy for I2P made available over clearnet, Lokinet and CRXN.

I **also** made the Yggdrasil endpoint availbale for peering to as an endpoint to get connected to the Yggdrasil network - over CRXN and Clearnet IPv6.

---

{{<bruh>}}
<img src="/projects/bonobonet/b_hash_logo.png" width="150" height="150" style="float:right">
{{</bruh>}}

## BNET

We now have IRC services provided by rany on BonoboNET and a new [site](../projects/bonobonet) (check it out!), we now have `NickServ`, `ChanServ` and `HostServ`. We just need to get a more stable connection over CRXN to the `services.bnet` node as it netsplits quite often. This is in the works as we speak.

There are peerings ready to happen very soon to get a few more European nodes and also possibly two more in South Africa to improve stability
and accessibility.

---

## Much more to come 🎊️

There's a lot more to be wrapped up this year in terms of getting projects to a stage whereby maintenance and work on them is easier and less frequent as well - more maintanable. A large portion of that has been done already but a few months more of work is needed to really sort out most of the kinks of most projects however I must say we really have everything planned well and are moving forward one step at a time for things like [CRXN](../projects/crxn), [BNET](../projects/bonobonet) and tlang.

There's a [new version](https://yggdrasil-network.github.io/2021/06/19/preparing-for-v0-4.html) of Yggdrasil coming out soon so I have already prepared plans on doing a seamless switch over the day that happens.