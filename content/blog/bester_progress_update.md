---
title: "Bester: Progress update"
author: Tristan B. Kildaire
date: 2020-04-21
---

# New things

## Adding multiple listeners

Something I wanted to add badly and have as a nice little extra
feature, not something that was inherent to the bester protocol
itself, was to add support for IPv6 to the bester daemon.
Therefore last night, after most of my university work had been
completed I decided that it was time to slightly refactor the
way the server listened for incoming connections. Instead of the
_accept-and-dispatch_ loop being a part of the <font
face="Courier New, Courier, monospace">BesterServer `
class I decided that that class should, yes, still be the
central point of coordination in terms of keeping track of **all**
connections but that connections should be appended to the `BesterConnection[] `
array via the listeners that were running. I wanted to allow the
user to be able to let their server, if they wanted to, be able
to bind to TCP over IPv4 along with binding to TCP over IPv6 as
well and even, maybe, UNIX domain sockets!

The way I worked this out in the code was by creating a class
named  `BesterListener `
which ran on its own thread and had an already implemented
connection _accept-and-dispatch_ loop for dispatching new
connections to  `BesterConnection `
thread objects. Then for implementing specific protocols like
TCP over IPv4 or over IPv6 or using UNIX domain sockets, all I
did was to create several other classes that extended the `BesterListener `
class and justset the super-class's `socket ` object to the one created
for that specific protocol.

And it worked like a charm!

The configuration for something like this looks like this in `server.conf `:

```
network" : { 
    "types" : ["unix", "tcp4", "tcp6"], 
    "unix" : { 
        "address" : "besterUNIXSock" 
    }, 
    "tcp4" : { 
        "port" : "2220", 
        "address" : "0.0.0.0" 
    }, 
    "tcp6" : { 
        "port" : "2221", 
        "address" : "::" 
    } 
}
```

So all you need to do is specify that you want certain socket
types and then just provide their binding informations.

## A new addition to the `sendClients` `sendServers` response commands

As I was discussing bester with a friend of mine, he
indirectly gave me an idea for something interesting I wanted
to implement. Normally if a user wants to get some processing
done by a handler, we currently have it working as follows.
Client sends a message to the server, server gets it and
dispatches it to the message handler. The only part that we
haven't done is handling of the reply that the server receives
back from the message handler. We have the reply working in
that it comes back but we haven't done much with regards to
that. So  `sendClient`
and  `sendServer`
are essentially not implemented. But in the example where say
now a user wants a message **A** to be processed by a
handler for **messageType1** but then wants the response
it gets back from the handler (indirectly via the server) to
be processed again by another handler, then the user must send
another message (the processed one, back). Now, I don't mind
this and of course you can still do this after the next
feature I am going to mention gets implemented, but the thing
I was thinking of doing was implementing a command named `sendHandler `,
which means after the server received a message from the
client and dispatched it to the **messageType1** message
handler that it (the handler) can then generate a response
message that commands the server to push that message to
another handler and then somewhere down the line the server
gets a response with a command that is not `sendHandler`
and the message ends back at a client or another server (**sendClients**
and **sendServers** respectively).

# Take it for a run yourself!

All you have to do is clone the repository [here](https://github.com/besterprotocol/besterd)
and then  `cd ` into and run the following.

Firstly  `cd testing/ ` and then run `python3 unixSock.py`, this
starts a bogus message handler. Now, start the bester daemon by
running  `dub`, and then lastly run a client example code by running
`cd testing ` and `python3 testSuite.py`!

You should see some output!

_And that's all for now!_