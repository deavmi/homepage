---
title: Projects
---

{{<bruh>}}
<img src="/img/code.png" style="float:right;gap;margin-left:20px">
{{</bruh>}}

I have been writing code for a rather long time now and have quite a few projects under my control, most projects here are active and
I am returning to development on them since having finished up with university. Most of the things here (if not all) are written in D
which is my programming language of choice. Some other projects will be returned to in terms of development whilst other are fully
deprecated.

{{<bruh>}}
<br>
<br>
<br>
<br>
<br>
<br>
<br>
{{</bruh>}}

If you have any questions, find my details on the [Contact page](/about) - _enjoy!_

---

# Active projects

All the projects listed below are considered active in development and/or are still supported by me.

## libpb

{{<bruh>}}
<img src="/projects/libpb/logo.png" width="25%" height="25%" style="float:right">
{{</bruh>}}

PocketBase wrapper with serializer/deserializer support

* [Code repository](/git/Haxio/libpb)
* [Issue tracker](https://github.com/Hax-io/libpb)
* [Project homepage](/projects/libpb)

## jstruct

{{<bruh>}}
<img src="/projects/jstruct/logo.png" width="25%" height="25%" style="float:right">
{{</bruh>}}

Library that easily lets you serialize a struct in D to JSON and vice-versa

* [Code repository](/git/Haxio/jstruct)
* [Issue tracker](https://github.com/Hax-io/jstruct)
* [Project homepage](/projects/jstruct)

## dlog

{{<bruh>}}
<img src="/img/dlog.png" width="185" height="119.5" style="float:right">
{{</bruh>}}

Simple and modular logging library with support for chained custom text transformers and custom loggers

* [Code repository](/git/deavmi/dlog)
* [Issue tracker](https://github.com/deavmi/dlog)
* [Project homepage](/projects/dlog)

## tristanable

{{<bruh>}}
<img src="/img/tristanable_logo.png" width="15%" height="15%" style="float:right">
{{</bruh>}}

Tag-based asynchronous messaging framework for D. This allows you to write software that awaits on messages
that have a certain tag sent with them.

This is heavily used in my chat applications because of the need of a state machine in an otherwise
non-state-machine-like socket stream where a reply for one thing can come in before another regardless
of the order things were sent in (or in the case of messages sent without a prior request (i.e. notifications)).

[Project homepage](/projects/tristanable)

## bformat

Simple message format that uses a 4-byte header for length. Used in many of my D network programs whereby I need a simple
container format to hold arbitrary data, perhaps binary data, JSON etc.

[Project homepage](/projects/bformat)

## BonoboNET

{{<bruh>}}
<img src="/projects/bonobonet/b_hash_logo.png" width=15% height=15% style="float:right">
{{</bruh>}}

An IRC network for hackers, programmers etc. Running on 3 servers currently, 1 in Lebanon, 1 in the UK and 2 in South Africa using `unrealircd`, provides atheme services for features such as `NickServ`, `ChanServ` and `HostServ`.

[Project homepage](/projects/bonobonet)

## CRXN

{{<bruh>}}
<img src="/projects/crxn/img/logo.png" width=15% height=15% style="float:right">
{{</bruh>}}

A community-run IPv6 network similiar to dn42 which runs in the `fd00::/8` space. We use bird for routing and an assortment of routing protocols such as babel and ospfv3. We have IRC servers on the network, game servers, a DNS root nameserver, many web services and more things! The network spans South Africa, United Kingdom, Germany, Lebanon and Russia.

[Project homepage](/projects/crxn)

---

<!-- # Sweatyballs

A toy routing protocol I am working on. Aims to do next-hop routing with crypto-routing included as well.

[GitHub](http://github.com/deavmi/sweatyballs)

--- -->

## libtun

{{<bruh>}}
<img src="/projects/libtun/logo.png" width=15% height=15% style="float:right">
{{</bruh>}}

A TUN/TAP OOP-based adapter for use in D-based applications

* [Code repository](/git/deavmi/libtun)
* [Issue tracker](https://github.com/deavmi/libtun)
* [Project homepage](/projects/libtun)

## gogga

{{<bruh>}}
<img src="/img/gogga_logo_small.png" width=13% height=13% style="float:right">
{{</bruh>}}





Pretty-printer for debug messages with VT100 colouring. Used in many of my projects to make debug messages much more readbale and easier to process by fellow humans.

[GitHub](https://github.com/deavmi/gogga)

## eventy

{{<bruh>}}
<img src="/projects/eventy/logo.png" width=10% height=10% style="float:right">
{{</bruh>}}

Event handling system for writing handlers and triggering them later.

* [Code repository](/git/deavmi/eventy)
* [Issue tracker](https://github.com/deavmi/eventy)
* [Project homepage](/projects/eventy)

## birchwood

{{<bruh>}}
<img src="/img/birchwood.png" width=10% height=10% style="float:right">
{{</bruh>}}

A sane IRC framework for the D language.

Makes use of [Eventy](/projects/eventy) for event handling and [libsnooze](/projects/libsnooze)
for the receive and send queue manager threads.

* [Code repository](/git/deavmi/birchwood)
* [Issue tracker](https://github.com/deavmi/birchwood)
* [Project homepage](https://code.dlang.org/packages/birchwood)

## libsnooze

{{<bruh>}}
<img src="/img/libsnooze.png" width=10% height=10% style="float:right">
{{</bruh>}}

A wait/notify mechanism for D

* [Code repository](/git/deavmi/libsnooze)
* [Issue tracker](https://github.com/deavmi/libsnooze)
* [Project homepage](https://code.dlang.org/packages/libsnooze)

## Guillotine

Executor framework with future-based task submission and pluggable thread execution engines

* [Code repository](/git/deavmi/guillotine)
* [Issue tracker](https://github.com/deavmi/guillotine)
* [Project homepage](https://code.dlang.org/packages/guillotine)

## gitea-irc-bot

A Gitea webhook-based IRC bot

* [Code repository](/git/deavmi/gitea-irc-bot)
* [Issue tracker](https://github.com/deavmi/gitea-irc-bot)
* [Project homepage](https://code.dlang.org/packages/v)

---

# Upcoming projects

These projects are either on hold or still being worked on and are yet to be released.

## tristan

{{<bruh>}}
<img src="/projects/tlang/logo.png" width="15%" height="15%" style="float:right">
{{</bruh>}}

A new programming language for systems programming with a focus on control of everything, minimalism and supporting OOP.

* [Project homepage](/projects/tlang)
* ~~[Source code]()~~ Source code will be available by the end of 2023, with a release expected by then.



---

# Inactive projects

All the projects listed below are no longer worked on or maintained.

## tasky

{{<bruh>}}
<img src="/img/tasky.png" width="210" height="46.33" style="float:right">
{{</bruh>}}

Framework for writing programs that require callbacks to run from certain types of messages sent
over a socket. Uses [tristanable](/projects/tristanable) and [eventy](/projects/eventy) in the backend.

* [Code repository](/git/deavmi/tasky)
* [Issue tracker](https://github.com/deavmi/tasky)
* [Project homepage](/projects/tasky)

## DNET

{{<bruh>}}
<img src="/img/dnet.png" width="119" height="102.5" style="float:right">
{{</bruh>}}

A new chat binary protocol aiming to replace IRC with modern day features.

* dnetd - A server for the DNET chat protocol
* skippy - A text-based DNET chat client
* Gustav - A GTK3+ graphical chat client for DNET

[Project homepage](/projects/dnet)

## Butterfly

Butterfly is a full email system that uses JSON-based messaging embedded in [bformat](). It provides a single server that acts as both mail delivery and mailbox management system (analog to providing SMTP and POP/IMAP in one single service). It supports client-to-server connectivity along with server-to-server connectivity for inter-server mail delivery.

1. [butterflyd homepage](/projects/butterfly)
    * This provides the mail server
2. [skoenlapper](/projects/skoenlapper)
    * This is the text-based email client for butterfly

## Bester

{{<bruh>}}
<img src="/projects/bester/logo.png" width="119" height="119" style="float:right">
{{</bruh>}}

A federated pluggable message-exchange protocol

[Project homepage](/projects/bester)

{{<bruh>}}
<br>
<br>
{{</bruh>}}