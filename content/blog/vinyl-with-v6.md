---
title: IPv6 Vinyl Pi but with Docker
author: Tristan B. V. Kildaire
date: 2023-06-29
draft: false
---

# What is this?

![vlc_client_listen.png](vlc_client_listen_1719509271807_0.png)

Well for a while now I have toyed with the idea of combining two things I *really* enjoy:

1. Vinyl records on turntable
	a. ~~Drinking a lot of wine whilst listening to them~~
2. IPv6

So how could one *actually* combine these two *high acquired* tastes together? Well by using a USB capture card and streaming my vinyls over the IPv6 internet!

# Prerequisites

You're going to need a few things before getting started, all of which are non-starters if you don't have any of them.

- ## Raspberry Pi

Any Raspberry Pi which:

1. Has a network interface whether it be Ethernet or WiFi
	a. You could make use of a USB WiFi adaptor or USB Ethernet interface if you have neither. In fact I believe the Raspberry Pi A (something like that) *only* had a single USB interface, therefore you would gave to go with such an option then.
2. Can run Ubuntu Server
	a. Your Raspberry Pi needs to be able to run the Ubuntu 23.10 release, at the very least the 64-bit version (that is what I tested with, 32-bit probably works still)
	b. We don't want to use the Ubuntu 24 LTS version because it uses a version of some Docker Python library that tools such as `sen` rely on and of which is utterly broken.
	c. ![node.jpeg](node_1719509248574_0.jpeg)

## USB capture card

A USB capture card which:
1. These come in the form of a device which plugs in via USB (and of which can even be solely powered by USB) which captures audio via RCA connections.
2. The device needs to be supported by Linux but honestly if the chipset they use internally is recognised as some well known one then it should work just fine. I plugged mine in and it was recognised as an input device immediately.
3. ![dac_1.jpeg](dac_1_1719509235866_0.jpeg)

## The obvious - a *turntable* baby 🕺️

A turntable:

1. I mean this really goes without saying, why would you be reading this in the case you didn't own one.
2. Actually, in order to humble myself, I propose a case whereby you *would* be reading this whilst simultaneously **not** being a turntable owner - you may own a cassette deck! To drive this point home, *any device* which has RCA output can be used in this project.
3. ![2024-06-27-19-31-16.jpeg](2024-06-27-19-31-16.jpeg)

## IPv6 network

Any IPv6 network would work, whether that be publicly routable, ULA/private or Yggdrasil based.

