---
title: "IPv6 Babel over LoRa: Part 2"
author: Tristan B. V. Kildaire
date: 2025-02-08
draft: false
---

# Why?

Because in the [last post](/ipv6_over_babel_part1/) we did it and now I want to continue working on it.

# What?

In part 1 we got setup with two RNode's running in their TNC (terminal node controller) mode. We were able to turn these into virtual networking interfaces that would appear on both $node_A$ and $node_b$ as `tnc0`.

## A new topology

What we now want to do is to introduce a third node, $node_c$, which will be placed into the following topology:

![](diagram.png)

What we want to setup this time is the following:

1. Every node from $node_a \space ...\space node_c$ should be running an identical Babe configuration as we showed last time
2. $node_a$ must be in radio range of $node_c$
3. $node_b$ must be in radio range of $node_c$
4. There should be **no** radio overlap between $node_a$ and $node_b$. We don't want overlap as we want to force traffic from $node_a$ to travel _via_ $node_c$ in order to reach $node_b$ **instead** of going directly from $node_a$ to $node_b$.

## Hardware

For this we will be making use of the [LillyGo T3S3](https://www.robotics.org.za/development-boards/esp32-lilygo-boards/H596) for each of our three nodes:

![2025-01-12-16-41-17.jpeg](../assets/2025-01-12-16-41-17.jpeg)

In terms of antennas I will be making use of [these](https://www.robotics.org.za/communication-wireless-Industrial/antenna-866mhz/YN-868MHZ-5DBI) antennas. They seem to perform rather well and are easy to mount on any surface that is magnetic - which will probably make testing easy for accomplishing the topology shown prior.

![2025-01-12-16-41-40.jpeg](../assets/2025-01-12-16-41-40.jpeg)

