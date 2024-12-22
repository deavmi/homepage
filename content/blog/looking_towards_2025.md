---
title: Looking towards 2025
author: Tristan B. V. Kildaire
date: 2024-12-22
draft: true
---

I'm not much of a _"new years resolutions"_ type of guy in fact I think those sort of things
are incredibly stupid and lead to you getting **no where in life**; so _it's best you stop that_
if you are currently engagaged in such a bizarre tradition that oofloads what you _could do **now**_
onto a future date. If you can start now, there's no excuse.

Before I get into what is to come in the next year and indeed what I have been working
on so much this year that I haven't "written" a single blog post, I first need to give
a little bit of an explanation for the first paragraph.

So here goes nothing...

# On procrastination

Now you may be asking as to why I started off with such a brazen statement? It has to be said;
we have **all** fallen to the devil of _procrastination_. I remember it hogging me down multiple
times (months even) before I actually started working on my compiler. Guess what, now and then
it tries to rear its head but I am getting better at controlling it.

How did I stary controlling it? I don't have some cookie cutter _buy me a book_ answer to this
actually. What I realised was that I was _scared of the endavour_ at hand. So, for the example
of my compiler project: "I was scared that I wouldn't abe able to take on such a huge task of
writing a compiler from scratch entirely". So in my case it was being scared of an academic
hurdle of sorts. However, I had experience in writing a basic one already from my second year
of computer science - so what gives? I clearly could have laid the basic foundation for the
compiler and worried about the next challenges as they rolled in. No, I was worried about
even writing thr first few lines of the code structure I _already knew_ - **the basics!**

The reason? Well, I was scared that I wouldn't be good enough. I was worried that I wouldn't
be able to claim any of it for myself because I would probably require having to reach out
online consistently and if that became a pattern then how would I be able to claim the majority
of the code written as my own? Luckily, however, I didn't need any help. Let me stop here for
a moment - I am in _no way_ saying that reaching out for help should stop you from completing
your dreams. You could argue that my first year of CS was preconditioning for me having the
knowledge to be able to write my compiler almost a year or so later - I think so!

Anyways... let's continue. So I began writing the lexer for my compiler and that started
to go well, then I started work on the parser and declaring all the AST types that I needed
as I wrote the compiler for a language design that was being built concurrently. As time
went on I actually started to surpase the capabilities of the compiler I had written for
my second year computer science module. I had to hit many challenges head on, those of
which I had never faced before in my undergrad course. Therefore there was no one to rely
on. Just an aside, as I stated earlier, yes I could have gone online and searched for 
answers but this was against my ideology _then_ and still **now** for this particular
project - I wanted to go through the motions myself.

>Put simply, I wanted to learn **not copy**

As things kept on this line of progression I had to re-write the dependency generator
about three times until I realised it was doing what I wanted it to do. This was a great
success for me. I remember looking at the de-cycled graph output (so visitation-marking
sort of graph traversal) for the first time when it started working and I was immensely
happy. I knew how important it was to get this particular algorithmn/mechanism implementred
correctly because I knew it would solve all future problems generally - it was integral
to the compiler. Even though I had nowhere near the features in my language implemented
as I do now - I could still tell that:

>I need not worry now, anything I _need_ to implement can _easily_ be done just with
a few lines of code since this mechanism will allow me to build out advanced features
too and not just simple ones.

---

Long store short, procrastination can be over come. All that is required is for you
to take the initial risk. Don't worry that you _may_ need to reach out for help
and if you try to do it all yourself (to a degree, nothing is done in solitude)
then take the risk too and spend the time it takes to overcome challenges. Remember
how I rewrote a particular component three times until it did what I wanted it to
do? That was me figuring out not just how to get it to work but what it should be
doing in the first place as well. I was truly thinking for myself instead of
getting someone else (or ChatGPT) to do it for me.

# What have I been up to

Well, there is just so much and it feels rather hard to prove too ever since I
have taken on the "work on it, talk later" mantra. So there's quite a few
things per project but let's start with a few in roughly chronological order:

## My homelab

My word, have I put in hours of work, debugging and documentation into this.
I had decided a while back already that I want a multi-router homelab, complete
with my own self-built rack and with a setup that was fully Dockerized. The
Docker part was very important as I wanted easy-to-deploy containers and I
wanted a declarative approach to stating what I wanted and how I wanted
it deployed. Dockerizzing (yes, I have immense rizz).

On the networking side of things I wanted to also get my DNS resolver back
up _along_ with it running as a nameserver so that I could have records
resolved via it whenever people eanted to reach certain domains hosted
in the `*.deavmi.assigned.network` region.

I also wanted to deploy OSPF (open shortest path first) to my routers
and learn that. Now, I didnt have the most advanced setup for it but
I got the basics done as that was all that I needed. I was very happy
with the dynamism of the results.

>Add a route to `fd77::420/69` here on `router1` and it appears a few
seconds later on `router2`

## The T compiler

I have been working on the compiler for my **T programming language**
quite a lot. I have been working on the following:

1. Enumeration type support
2. Struct support
3. Comment support
	* Specifically within the parser as that is where I last stopped
	working on such support
4. Clean ups here and there
	* There has been quite a lot of code-churn (the good kind) due to
	me reworking a lot of the code; this has meant getting rid of code
	as well - this is the best type of churn!
5. Internal API changes
	* Cleaning up of the `CodeEmitter` API to remove the stuff relating
	to `SymbolMapper`(s) which are no longer used and are rather internal
	to the given code emitter _implementation_ such as the C emitter `DGen`
6. Dotting support
	* This took majority of the type and was actually required before
	I could get to work on struct support or enumeraiton type support.
	* What it amounts to is the treating of the `.` operator (the `SymbolType.DOT`)
	as an actual operator. This is still being worked on but for the
	most part it is probably done and the other things that rely on
	it like enumeraiton types and struct types just have to have
	their additions added to it.

There are probably many other things here and there that I can't
remember right now as I type this; however all I can say is that
between those 6 items there has been **a lot** of work done.

Oh, and a surprise! I have been working on a package manager
for T! It can both fetch and build packages along with generating
documentation:

### Docgen

![](looking_towards_2025/tpkg_docgen.png)

The above shows the output of doing documentation generation
for an example single-module program `simple_comments.t` (which
is actually part of the T test suite).

### Fetching a package

![](looking_towards_2025/tpkg_packman1.png)

We can see in the above how a search is performed with the 
regular expression `tsh*` and how a matching candidate is
found for `tshell`. This is then fetched over the network
in chunks (hence the usage of a progress bar). It comes
in the form of a `.zip` archive. This is unzipped and stored
and some basic checks are performed.

I was quite proud of the callback-based (using a delegate/closure)
approach to fetching using D's `libcurl` wrapper. You
firstly make an _HTTP HEAD_ request to get the `content-length`
field of the resource (the `.ZIP` archive) you want
to fetch. Then you do a chunked fetch of that whereby
after each chunk you call a method (your callback)
that consumes the just-received bytes. You then use
this to update the progres bar as `numberOfBytes/expectedTotal`.

### Building the package

![](looking_towards_2025/tpkg_packman2.png)

Once the package has been stored, a `StoreRef` is returned,
this is then used to actually setup an instance of the `Compiler`
such that we can trigger the `compile()` method on it. After
which the compilation process begins and we get information
about how long the procedure took and then also the path
to the created executable.

## Reticulum, LoRa, radios and serial adaptors

![](looking_towards_2025/lora_1.jpeg)

I have had a bit of a foray into the [Reticulum Network Stack](https://reticulum.network/)
and I have decided I want to extend the testnet physically into
my area. This will be a long term project of acquiring some
relatively inexpensive hardware but then for the type of
installation I want to do - some of it will be more expensive
and require longer term savings (for outdoor enclosures of
the type I may be looking at).

![](looking_towards_2025/lora_2.jpeg)

That's the one part, but as for the software side I helped
add IPv6 support ([here](https://github.com/markqvist/Reticulum/pull/601) and
[here](https://github.com/markqvist/Reticulum/pull/545)) to
Reticulum - something that was desperately needed.

I the decided I wanted to try IPv6 over `tncattach` (which
is a program that let's you use your [RNode](https://unsigned.io/software/RNode_Firmware.html)
as a TAP/TUN interface). So I went ahead [and did some
C programming for that](https://github.com/markqvist/tncattach/pull/16).

There is just **so much** I have in store relating to this.

# What to expect

I do have some blog posts pre-written for the upcoming year
that I will be releasing shortly. It's a mixed back as to
what they're about but nontheless some of them will give
you a greater understanding of each of the things I have
mentioned and what exactly I was so busy with.

---

{{<bruh>}}
<center>
	<h2>
		Have a blessed Christmas, be with your loved ones whilst
		you still can.
	</h2>

	<h4>See you in 2025</h4>
</center>
{{</bruh>}}
