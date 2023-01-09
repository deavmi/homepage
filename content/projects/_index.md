---
title: Projects
---

{{<bruh>}}
<img src="/img/code.png" style="float:right;gap;margin-left:20px">
{{</bruh>}}

I have been writing a lot of things as of late. Most of the projects here are still active projects, some I shall return to very soon. If I truly have an abandoned project then it will be listed here but as for now everything here is an active project of mine.

---

# Butterfly

Butterfly is a full email system that uses JSON-based messaging embedded in [bformat](). It provides a single server that acts as both mail delivery and mailbox management system (analog to providing SMTP and POP/IMAP in one single service). It supports client-to-server connectivity along with server-to-server connectivity for inter-server mail delivery.

1. [butterflyd homepage](/projects/butterfly)
    * This provides the mail server
2. [skoenlapper](/projects/skoenlapper)
    * This is the text-based email client for butterfly

---

# DNET

{{<bruh>}}
<img src="/img/dnet.png" width="119" height="102.5" style="float:right">
{{</bruh>}}

A new chat binary protocol aiming to replace IRC with modern day features.

* dnetd - A server for the DNET chat protocol
* skippy - A text-based DNET chat client
* Gustav - A GTK3+ graphical chat client for DNET

[Project homepage](/projects/dnet)

---

# Bester

a federated pluggable message-exchange protocol

[Project homepage](/projects/bester)

---

# tristanable

Tag-based asynchronous messaging framework for D. This allows you to write software that awaits on messages
that have a certain tag sent with them.

This is heavily used in my chat applications because of the need of a state machine in an otherwise
non-state-machine-like socket stream where a reply for one thing can come in before another regardless
of the order things were sent in (or in the case of messages sent without a prior request (i.e. notifications)).

[Project homepage](/projects/tristanable)

---

# bformat

Simple message format that uses a 4-byte header for length. Used in many of my D network programs whereby I need a simple
container format to hold arbitrary data, perhaps binary data, JSON etc.

[Project homepage](/projects/bformat)

---

# BonoboNET

{{<bruh>}}
<img src="/projects/bonobonet/b_hash_logo.png" width=15% height=15% style="float:right">
{{</bruh>}}

An IRC network for hackers, programmers etc. Running on 3 servers currently, 1 in Lebanon, 1 in the UK and 2 in South Africa using `unrealircd`, provides atheme services for features such as `NickServ`, `ChanServ` and `HostServ`.

[Project homepage](/projects/bonobonet)

---

# CRXN

{{<bruh>}}
<img src="/projects/crxn/img/logo.png" width=15% height=15% style="float:right">
{{</bruh>}}

A community-run IPv6 network similiar to dn42 which runs in the `fd00::/8` space. We use bird for routing and an assortment of routing protocols such as babel and ospfv3. We have IRC servers on the network, game servers, a DNS root nameserver, many web services and more things! The network spans South Africa, United Kingdom, Germany, Lebanon and Russia.

[Project homepage](/projects/crxn)

---

# Sweatyballs

A toy routing protocol I am working on. Aims to do next-hop routing with crypto-routing included as well.

[GitHub](http://github.com/deavmi/sweatyballs)

---

# libtun

{{<bruh>}}
<img src="/projects/libtun/logo.png" width=15% height=15% style="float:right">
{{</bruh>}}

A TUN/TAP OOP-based adapter for use in D-based applications

* [Code repository](/git/deavmi/libtun)
* [Issue tracker](https://github.com/deavmi/libtun)
* [Project homepage](/projects/libtun)

---

# gogga

Pretty-printer for debug messages with VT100 colouring. Used in many of my projects to make debug messages much more readbale and easier to process by fellow humans.

[GitHub](http://github.com/deavmi/gogga)

---

# eventy

{{<bruh>}}
<img src="/projects/eventy/logo.png" width=10% height=10% style="float:right">
{{</bruh>}}

Event-loop system for signal handling systems.

* [Code repository](/git/deavmi/eventy)
* [Issue tracker](https://github.com/deavmi/eventy)
* [Project homepage](/projects/eventy)

---

# tasky

{{<bruh>}}
<img src="/img/tasky.png" width="210" height="46.33" style="float:right">
{{</bruh>}}

Framework for writing programs that require callbacks to run from certain types of messages sent
over a socket. Uses [tristanable](/projects/tristanable) and [eventy](/projects/eventy) in the backend.

* [Code repository](/git/deavmi/tasky)
* [Issue tracker](https://github.com/deavmi/tasky)
* [Project homepage](/projects/tasky)

---

# tristan

{{<bruh>}}
<img src="/projects/tlang/logo.png" width="15%" height="15%" style="float:right">
{{</bruh>}}

A new programming language for systems programming with a focus on control of everything, minimalism and supporting OOP.

* [Project homepage](/projects/tlang)
* ~~[Source code]()~~ Source code will be available by the end of 2023, with a release expected by then.

---

# dlog

{{<bruh>}}
<img src="/img/dlog.png" width="185" height="119.5" style="float:right">
{{</bruh>}}

Simple and modular logging library

* [Code repository](/git/deavmi/dlog)
* [Issue tracker](https://github.com/deavmi/dlog)
* [Project homepage](/projects/dlog)

---

# libpb

<!-- {{<bruh>}}
<img src="/img/dlog.png" width="185" height="119.5" style="float:right">
{{</bruh>}} -->

PocketBase wrapper with serializer/deserializer support

* [Code repository](/git/Haxio/libpb)
* [Issue tracker](https://github.com/Hax-io/libpb)
* [Project homepage](https://github.com/Hax-io/libpb)

---

# jstruct

{{<bruh>}}
<img src="/projects/jstruct/logo.png" width="25%" height="25%" style="float:right">
{{</bruh>}}

* [Code repository](/git/Haxio/jstruct)
* [Issue tracker](https://github.com/Hax-io/jstruct)
* [Project homepage](/projects/jstruct)