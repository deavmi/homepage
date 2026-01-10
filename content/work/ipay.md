---
title: My journey at iPay
---

There are multiple things I am working on or have already worked
on during my time at iPay.

## Skills summary

A lot of what I worked on was my own hand-rolled implementation,
this meant I learnt a lot and fixed whatever bugs appeared but
I put quite high trust in the execution of these components that
they _will always work_. I think deeply about them.

* Own-rolled `Future<T>` implementations
	* Careful use of `volatile`, `synchronized` concepts
* A lot of concurrency
	* Usage of `ReaderWriterLock` where it made sense
	* Condition variables (`notify()`, `notifyAll()` and `wait(int)`)
* Request management
	* Own-rolled request-response matcher with futures as the external API

* Common Socket IO
	* Working with `read(int, byte[], int)` with sockets
	* Custom `InputStream` and `OutputStream` implementations
* Decoders/encoders
	* Number encoding/decoding utilities, endianness, sign-extension etc.
	* BCD8421 decoding, bit-pattern decoding/encoding

* Task scheduling
	* Job filtering/pipelines
	* Dispatch and callback management
	* Usage of `Executors`, `ThreadFactory`, etc.

* Service management
	* Own-rolled component discovery service
	* Component **A** registers with a certain "match set"
	* Component **B** can then wait on a `Future<Component<A>>` until
		such a component matching the provided match set (search query)
		appears
	* Event listeners could also be installed and triggered on
		their match sets being satisfied (on _advertisement_ and _revocation_)

* Plugin management 
	* Own-rolled plugin management system with life cycle management
	* Plugin metadata delivered by some _provider_ to a central
		controller:
			1. Handled start up and shutdown (on delivery and retraction)
	* Implemented an XML-based provider that would _deliver_ and
		_retract_ plugin metadata to the controller based on file-system
		changes

## Background

A little bit of background on what iPay does. This company has
been selling a product called _BizSwitch_ for more than two
decades now. This incorporates a large switching system for
processes messages of all kinds, payment requests, meter
control commands and processing of readings.

As you can imagine it's already a relatively complex system,
even a single component such as the management of _meter
control commands_ is a collection of many different remote
systems coming into play in order to orchestrate the
control of remote devices.

## Where I came in?

Near the end of my honours year I got an e-mail for an interview
at the company. I knew my friend had been working there for
a year or so already but I had less-than-vague of an idea
of what they _actually_ did.

I had a meeting with them online and got a better understanding
of what iPay is currently doing and what their new venture
was - of which they wanted me on their team.

After signing with them, the following year I met up with the boss,
Henty Waker and his brother Travers (who I would later learn was
the director of IT at iPay). They presented me with a box and
which had an smart meter sitting inside of it.

This was my first time ever really handling one.

Long story short, I spent the next 8 months just getting 
accustomed to the DLMS standard and that specific meter.

## The journey

### Learning DLMS

The journey began as me just using some very sparse pre-existing
code that an employee had written a while back. I wasn't all
that clued-up on how it all worked but I had a vague idea where
to start. Getting communications up and running, even if with
a wired connection to the meter, was the first thing I would
need to do.

The memory is hazy now but it most likely started off with me
noticing that the code had to _somehow_ interact with the
meter and at least over serial - as I had been given one of
those optical eye serial adaptors.

After getting that working, I explored the meter (and DLMS)
via the library we were using. I got to understand more about
how DLMS worked and also the set of object types or _classes_
that were supported. 

### Designing the hardware abstraction layer

After figuring this out I decided _"Okay, I need to wrap all
of this into some a tidy driver-like class so I can use it
more easily"_.

Eventually, I got the vague notion that **capabilities**,
in the form of interfaces, was what I wanted. This is also
what then lead to splitting out the various different DLMS
drivers into their own sub-classes.

Later on, after some development, I decided that what I
really wanted this project to be was _generic_. I wanted
there to be standardized capabilities and a very small
base driver. Therefore I designed a Hardware Abstraction
Layer (HAL), which became the _"Hydra HAL"_.

### An extensible task scheduling system

At the same time I had to begin work on the mechanism
that would be used to interface between Hydra and BizSwitch.
This would be the component responsible for receiving
XML messages, decoding them and translating them into
tasks to be run.

_Tasks_ you say? Well yes, this lead me to the next
big endevour. A task scheduler would be needed which
would need to be able to run these jobs and support
callbacks which would be used to send XML reply 
messages. Lo-and-behold I ended up building an
extensible scheduler that had an identity provider
mechanism to lookup devices and their drivers
to figure out what to setup and how; and **only**
done when needed (when the task could begin
execution and become a _job_).

Tasks had a few _concepts_ to them, there was
a provider for so-called _"identities"_ which
is what would actually spawn the driver _on-demand_
when needed. This on-demand mechanism also
meant we would only get access to the driver
when it was connected-to and authenticated.

A second idea was that upon completion a callback,
of the user's choice, could be called. This
came in handy, as one could imagine, for
_various_ things other than just user-created
callbacks; also internall (see section below).

#### Queue Policy Framework (QPF)

Along with this, the concept of _Queue Policy Framework (QPF)_
came along. Certain tasks under certain conditions
would need to be held back. I didn't want to
hardcode all of these device-specific edge cases,
therefore I developed a framework that would
allow the insertion of policies that worked
in a fashion similar to Linux's netfilter,
where a "packet" (in my case _task_) could
be be held back (for the next candidate
selection round) or accepted for running
_right now_.

These could of course be chained to one
another. The scheduler would only check
policy {{ transform.ToMath "p_{i+1}" }} is policy {{ transform.ToMath "p_{i}" }} returned
a positive verdict.

Certain queue policies are stateful, therefore
the modification of their state could
be accomplished via custom callbacks
they would chain on to the job (by
modifying it).

There are **many** intricacies to such
a filtering system that had to be taken
into account. As I was developing this
system by myself I came across them and
ensured that the fixes that would be
needed were not kludges but things that
would be both easy on the _internals_
(as I would maintain it) and keeping
the _external_ API simple as usable
for those developing future queue policies.

## Where we ended up

