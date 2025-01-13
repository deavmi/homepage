---
title: IPv6 Babel over LoRa: Part 1
author: Tristan B. V. Kildaire
date: 2025-01-13
draft: true
---

# Why?

![](ipv6_lora_part1/gfy.jpeg)

Too many times stupid questions like these are posed by _weak betas_ - I will not entertain such silly questions.

I'm doing it because _unlike the detractors it's_:

* Fun
* You learn something
* I'm not a party pooper 💩️

# What?

Let's specify specifically what it is that we will be attempting today. In my [previous blog post]() we got an IPv6 link working over LoRa using our RNodes and a program called `tncattach`.

## Where does babel fit in?

Once we have a working link over which IPv6 traffic can flow and of which the network interfaces on both sides have an IPv6 link-local address assigned, we can then make use of a program called `babeld`.

Say now you have a a few routes in your IPv6 routing table like:

![](ipv6_lora_part1/routes.png)

Now how would you distribute these routes to  _other_ nodes? Well, that's exactly what Babel allows us to do. It runs as a daemon on all of your nodes you wish to have _receive_ routing information and also _distribute out_ (their) routing information.

