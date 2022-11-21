---
title: The future of DNET and Butterflyd
author: Tristan B. Kildaire
date: 2021-07-18
---

Lats year I decided I wanted something _like_ IRC but with a few more modern features like in-band registration (meaning you can register within the protocol itself - no need for services). I also worked a lot on Butterfly which is my email replacement.

The protocol for DNET was binary (and I like that) but prototyping it fast and manipulating bytes by hand does get tiring and hence I want to change that. As for Butterfly it uses JSON and I want it to be binary but don't want to fall into the same trap that DNET originally did.

Whilst prototyping protocol design I must say that ProtocolBuffers help a lot. You get a nice way to manipulate fields and then encode them into a byte stream afterwards.

I have decided I shall reboot both of these projects very soon (and essentially restart them both), using ProtocolBuffers. DNET will have a complete overhaul, from the API library, the GTK and text clients and the server. Butterfly, although already fully working, will be reimplemented to use ProtocolBuffers (it is here where I think not a lot needs to actualy change). DNET however, never really was fully completed and I feel like I want to redesign the protocol for it in any case. Butterfly however, had a solid protocol that was basically completed.


It is with this reasoning that I shall migrate both projects to using ProtocolBufers and I shall begin with DNET first. It most likely won't
happen all too soon but I think I will have some free time to atleats get the ProtocolBuffer descriptors (the `.proto` files) written up and uploaded to the repositories.