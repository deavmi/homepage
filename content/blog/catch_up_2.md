---
title: "I've been super busy as of late - A quick catch up (Part 2)"
author: Tristan B. Kildaire
date: 2020-07-25
---

# Continuing where we left...

So [BonoboNET](/projects/bonobonet) is now growing and we have a few users
(most albeit on a bridge) but some new native users I am convincing to join
now are coming on - which is great! As for [CRXN](/projects/crxn)
we have had users, like [Chris](http://chrisnew.de), join from Germany, 
[Clickme.sh](http://n4vn33t.com) from India, soon the US too with some other
folks!

Now let's move onto the next things!

## [Bester](/projects/bester)

So I started work on what I see as a more modular XMPP which I call
Bester. I have written quite a few blog posts on it (look here, here
and here). And basically what Bester provides is an authenticated
mechanism whereby a client can send some data into the Bester
daemon, then message handlers (which are separate processes that
connect over UNIX domain sockets to the Bester daemon) will handle
specific messages of specific types and then produce an output for
them, basically your message goes into the server (the Bster daemon)
-&gt; to a message handler -&gt; and then the output from the
message handler can be directed either back to the users on the same
server, users on the same server and remote servers or, and get
this, back into another message handler (and then the same process
applies).

With such a system I am to make some cool IoT systems, filtering
systems and connected-goodness - I also plan to make a chat program
that works on top of Bester as I think that will work out well!

There is a rough working version (I mean it isn't that bad,
everything works but I want to make it more thread safe etc. and
cleaner - the implementation) whereby you can do pretty cool things
in terms of topologies of Bester daemons and moving messages
throughout them!

## [Butterfly](/projects/butterfly) - electronic mail simplified

I don't think I have yet posted much on Butterfly or at all - only
really on Mastodon! Now I have always wanted to have email but in an
easier to setup way and something that just worked in a more simpler
way - now I haven't looked at the email spec but the fact setting up
proper email requires two daemons (one for SMTP and one for IMAP)
was already enough for me so I said, in usual Deavmi terms "fuck it,
I'm reinventing the wheel - except this time it won't be a bicycle
but a unicycle as was the bare minimum requirements (wait until you
see how mail filtering is _proposed_ to work).

So I started work on it and now the basic functionality is all there
including inter-server mail exchange the only things left to do are
thread safety, performance increasing via tristanable and code clean
up and a few more functions like mail deletion, folder deletion etc.
but the mail aspect of sending and receiving and checking your inbox
and fetching mail are all there!

So what I am to do with mail filtering is to have some generic
protocol that others can use to build filtering daemons - yes I lied - filtering
requires another daemon but at least you'll be able to
disable this feature -_-. I plan on making Bester apart of this
aspect but not bundled in anyway and separate - I want to maintain
the UNIX philosophies - do one thing and do it well!

## [tristanable](/projects/tristanable) - tag-based messaging library

Now for both Bester and Butterfly I want to make things more
job-oriented than based around a simple state machine. By this I
mean that if I have two jobs, job 1 and job 2, then rather than
making it such that I make a request from the client -&gt; server to
start job 1 and then await the completion and result from server
-&gt; client and then do the same for job 2, I want to submit both
jobs, job 1 and then job 2 and as soon as one finishes then I want
to receive its result - now one can program this but the thing is
that means programming the same thing time and time again for
several programs, just like bformat (which you will see in the next
section), so I decided to write a library to do this for me.

tristanable is for the client side, obviously with worker threads
for each job on the server-side this is easy to accomplish without a
library - all the library does is listen for incoming results and
store them in an array that you can access using functions like
`.receive(tag=1)` (where tag is the same concept as job) which, if
the result for the tag/job 1 is not in the array it will then loop
and check but if it does find it sometime during that loop, or even
before the first iteration then the `.receive` function will return
immediately - this is the crux of tristanable.

## [bformat](/projects/bformat)- socket encoding/decoding payload-management thin

This library, which should be renamed, basically manages receiving
and sending of messages in the bformat format which consists of
`[---- 4 bytes (specifying size in little endian encoding) ----|---- payload (n bytes) ----]`.

I use this in all of my networking projects - at least so far.<br>

# And that's a wrap!

I should mention also that I started a [PeerTube channel](https://video.autizmo.xyz/video-channels/deavmis_shack/video)
whereby I post computer builds and networking server-ish stuff - so check that out if you will!