---
title: "A new chat protocol"
author: Tristan B. Kildaire
date: 2020-09-29
---

# Introduction

I've had a lot of free time on my hands during the statist
lockdown of '20 and made good use of it as you've seen with Bester
and Tristanable to name a few. Something I am now working on is a
new chat protocol called dnet (short for deavmi-net or dlang-net
dependent on how I'm feeling - more in love with myself or my
favorite language). Now let's get to the <i>meat</i>.

Why a new protocol? I mean we already have XMPP, Matrix (ew) and
IRC? Well let me cut the first two out as they don't "link" (to to
speak) in the way I want dnet to form a part of. I am looking at
you IRC - you got that nice linky-linky system. So IRC let's you
link servers in the following way

<div align="center"><font face="Courier New, Courier, monospace">Server
A &lt;-&gt; Server B &lt;-&gt; Server C</font><br>
<br>
<div align="left">Whereby a message sent by user on server A in
a channel that a user is a member of on booth Server B (user2)
and Server C (user 3), will be delivered to both servers, all
via Server B in this case. I want dnet to work like that, so
in that sense you could call dnet an IRC replacement. Okay
then, but what will dnet offer that differs from that of IRC -
a protocol that has existed for a <i>very</i> long time.<br>
<br>

# What does dnet do differently?


<div align="left">Well to be honest, so far not much, but the
point is that I have a clean slate to work on a protocol the
way I envisage it to work. The one thing that can be assured
it will do differently is client to server authentication -
something that IRC sucked at in the sense it never offered
built-in authentication - okay well I guess SASL exists but
I'm adding that idea into dnet right from the get go and it
will be simple - trust me. So that's one score for me.

Well, what else? Well, as I'm typing I also recalled one thing
I would really like to add, a sort of mailbox feature to
emulate MemoServ on IRC networks such as Freenode. What
MemoServ did is it was an IRC service that you could message
and leave a message for another IRC user such that when they
logged on MemoServ would message them with all the memos in
their inboxes. So with that in mind I want to add a feature
whereby if you were messaged by someone whilst offline then it
would remain in your inbox and be delivered upon logon. So far
the system is pretty ephemeral so I should look into how I
would like to go about implementing history and all - another
score&nbsp; and also to mention I may want to do something
similar to this with channels. When you join a channel you can
get the channel history up to a configurable point. This
implies that the messages will be saved, a feature that can be
turned on and off by the channel owner.

So with that said, you would be able to send a message to a
person and that person receive it when logging on, similarly
with channels you can receive channel history. These are
features to be added later but ones I definitely care about
and want to have.

Another thing I want to add is user and channel properties. So
the main identifiers and information that a user and channel
have is their names which are unique on the dnet network and
that is their handle, if you wanna talk to user <b>X</b> you
send to <b>X</b>, likewise with channels, if you want to send
a message to channel <b>X</b> then you send to channel <b>X</b>.
But a user needs a status and a channel needs a topic. Now I
thought about this, we could have some simple variables, but
then I thought you know what, perhaps having a little JSON <i>somewhere</i>
to spice things up could be quite nice. The thing is I didn't
want the <b>whole</b> protocol to be JSON based but I can see
where it can be useful - in situations whereby you want
structured information like this - and I think JSON would work
quite well here - I might change this later but for now I
think it's a good idea. So users can have several fields in a
JSON object that stores, for example, their status message
perhaps like "I am currently sitting on a chair" and maybe
their status "Away". Likewise, a channel could have it's topic
as a field "General | The general discussion room" and other
fields as well. For channels this can be quite handy for
certain clients that expect certain fields, it can help
integrating external services of things perhaps - additional
information relative to the channel.

I am still weary I want to include such a thing in the client
but I definitely want some sort of property system. Things
that won't be properties or accessed in such a fashion will be
fundamental features such as, for channels, the users list and
operators list. The JSON idea is alright but I just want
simple extensibility not something huge, again it's just an
idea so far. I will think about it more later.

# Show me the money (or cents)

So what's implemented so far then? Well so far
the following commands:

* auth
    * Tell the server you're a user connecting (instead of a server for linking)
    * Provide your username and password to authenticate as
* join
    * Join a channel
    * If the channel does not exist then it will be created
* leave
    * Leave a channel (IRC's part)
* msg
    * Send a message to a user or channel
* list
    * List all the channels available

So only a few are implemented but so far, the most important
ones. The client, skippy, works well too (after fixing a small
tristanable <a
href="https://github.com/deavmi/tristanable/commit/b0b7f69778f32cf3a8c8612e0c917aa9f637fa33">bug</a>
which is <a href="tristanable_bug.html">worth writing about
itself</a>). Other things to mention is the recently added <font
face="Courier New, Courier, monospace">housekeeping()</font>
code which manages what happens when a user disconnects, such
as leaving all channels he was a member of. The leave and join
methods too<br></h3>
</div>
are very nice in the sense that they actually, on call,
broadcast to the channel the user is joining or leaving that
the new user has joined - this notification sent to all
current members before the<br>
member change (in the case of join) and after the member
change (in the case of the leave) - all of this is working
well!<br>
<br>
<br>
And best of all - it's all thread safe - I just sprinkled some
mutexes all over it.<br>
<br>
What's next? Well to work more on channel features, like
member listing etc., and for registration and that sort of
stuff. Mailbox stuff should be next on the list, then
properties and then lastly a performance overhaul in the sense
of the data structures used. As for now I would like to get
most of this done as soon as possible - definitely before year
end.<br>
</div>
<br>
</div>
<br>
<br>
</div>
Want to find out more? Here is the <a
href="https://github.com/deavminet">GitHub repo</a>.<br>

