---
title: "A tale of tbk's tristanables terrible timing bug"
author: Tristan B. Kildaire
date: 2020-09-29
---

# A little timing bug that went a long way

This won't be a long blog post but I think it's such a good one in
how one little ordering can have a huge impact in terms of the bug
it caused. So first let me describe the situation and what
happened. I was working on the dnetd (the dnet daemon) and skippy
(the reference implementation client for the dnet protocol). The
latter uses tristanable to keep track of server responses we are
expecting using tagged messages, tag requests (to account for
tagged responses and put them in their mailbox such that the <font
face="Courier New, Courier, monospace">receiveMessage()</font>
function can dequeue them for us when the while loop finds it, the
matching requested tag, in the receive queue).<br>
<br>
So what would happen is I would send a message like so: <font
face="Courier New, Courier, monospace">sendMessage(byte[], ulong
tag=20)</font>, this would add to the request queue a Request
object with the tag 20, I would then call <font face="Courier
New, Courier, monospace">receiveMessage(ulong tag=20)</font>,
which will loop lock the queue, check for the Request object for
that tag (if it has been fulfilled) and then unlock, whenever the
scheduler switches to the Watcher thread, the thing which fulfills
requests for us by filling the Request object with the response
from the server with the matching tag, and the mutex can be
locked, then it will check if such a Request with a given tag
exists, if so, it will add the data to it and set it as fulfilled
such that when we switch back to the <font face="Courier New,
Courier, monospace">receiveMessage()</font> function, it will
grab it on the next loop.<br>
<br>
Now if the matching tag is not found, then we check whether or not
it is a reserved tag, if so then the response goes to a
Notification queue, else it is dropped. Now look at the below
code:<br>
<br>

```d
/* Create a new Request */
Request newRequest = new Request(tag);

/* Lock the queue for reading *
lockQueue();

/* Add the request to the request queue */
requestQueue ~= newRequest;

/* Unlock the queue */
unlockQueue();

/* Send the message */
bSendMessage(socket, messageData);
```

The above code is the fixed version of the code, notice that the
send comes at the end, after the Request is enqueued, such that it
is guaranteed it exists if after the send we immediately were to
switch to the Watcher thread.<br>
<br>
Guess what the bug was and why it happened only sometimes, the
above happened but the <font face="Courier New, Courier,
monospace">bSendMessage()</font> call was before the enqueueing,
and therefore the Request was only added after, after the Watcher
received a tagged message (caused by the server receiving the
tagged message sent in <font face="Courier New, Courier,
monospace">messageData</font> by <font face="Courier New,
Courier, monospace">bSendMessage()</font>) for say now tag=25
and then dropped it as there was no Request object in the queue
with such a matching tag.

So the code used to look like:

```d
/* Send the message */
bSendMessage(socket, messageData);

/* Create a new Request */
Request newRequest = new Request(tag);

/* Lock the queue for reading */
lockQueue();

/* Add the request to the request queue */
requestQueue ~= newRequest;

/* Unlock the queue */
unlockQueue();
```

My client would always hang, which made sense, as no other message
with such a tag would ever be sent after the Request object had
been enqueued and hence the <font face="Courier New, Courier,
monospace">receiveMessage()</font> function would loop till
infinity.

This took me so long to find, namely because I had so much trust
in my code having had to re-implement tristanable in Java for a
university networking project. I couldn't see how it could go
wrong but it did. Then, after having thought it was in skippy or
dnetd and having overhauled all that code for nothing - I found
it!

That reminds me! Time to make the <font face="Courier New,
Courier, monospace">reservedTags</font> queue thread safe!

