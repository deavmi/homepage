---
title: My journey at iPay
---

There are multiple things I am working on or have already worked
on during my time at iPay.

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
_right now_. These could of course be
chained and they could modify the _task_
as well, attaching custom callbacks
to them.

There are many intricacies to such a
system and hard problems needed to be
solved in order to make chained
callbacks work, even when a policy
rejected a job mid-queue.

## Where we ended up
