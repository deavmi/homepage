---
title: "RNode adventure: Part 1"
author: Tristan B. V. Kildaire
date: 2025-01-13
draft: false
---

# The plan

Firstly let me describe the components for what I want to accomplish today and then also what the goals are.

## Antennas

What I wanted to get done today was to test out two of my new antennas and at a further range than I had tested before with my RNodes.

### Antenna with magnetic base

![](antenna_magnetic.jpeg)

These antennas come with a $3m$ lead which means that you can place the radio and antenna at quite the distance apart. This is helpful for cases where you want the antenna to reach some place far away but at the same time you wouldn't be able to run the data cable to the radio (the RNode) at that same length.

We shall see how the above use case was rather handy for me in my setup that I had when using this antenna today.

These are available for purchase [here](https://www.robotics.org.za/communication-wireless-Industrial/antenna-866mhz/YN-868MHZ-5DBI).

### Longer pole antenna

![](pole_tenna_1.jpeg)

These larger antennas fit the same use case of the original stubby antennas that are provided with the LoRa T3-S3 board of mine:

![](pole_tenna_2.jpeg)

They are useful for portable devices that you can whip out and begin using. The antenna is probably as big as one is willing to slog around and have permanently fixed. It's also easy enough to store as it is rather rigid.

It has 3 angles it can be adjusted to on its hinge at $0$, $45$ and $90$ degrees.

These are available for purchase [here](https://www.robotics.org.za/communication-wireless-Industrial/antenna-866mhz/YN-868MHZ-5DBI).

## Tests

What I wanted to actually get done today was to test out _just how much better_ the new antennas would perform. Now I must state firstly that it wasn't a very scientific test really because I had nothing to compare the antennas too besides my previous stubby antennas (as mentioned earlier) but I never put them through the same test in any case so I feel it is rather unscientific.

Either way, we're going to see how they work in either case.

The approach used will be to send a few messages from my phone to my laptop. In order to test the bi-directionality I need not send something form the laptop to the phone. The reason the latter need not be _explicitly_ tested is because whenever a message is sent a message receipt is sent back (upon arrival at the endpoint) - therefore if I see that I will know that bi-directionality worked.

* Technically when establishing a _link_ it would test bi-directionality as well.
* Therefore back-and-forth communication is tested a few times already just by me sending a message from my phone to my laptop.

### Devices

I will be running tests from my phone running [_Sideband_](https://github.com/markqvist/Sideband) which is an Android-based LXMF client which can communicate over Reticulum:

![](sideband.jpeg)

The second device will be running the CLI-based LXMF client called [_nomadnet_](https://github.com/markqvist/NomadNet):

![](nomadnet.png)

I have setup both devices such that the only interface they have available is the RNode-based interface. This will ensure that I only have traffic delivered to my phone via the LoRa radio and not any other interface such as via the Internet.

>Note: The laptop had other interfaces active but none of which my phone would have been reachable via; hence it would always have to find a path over the LoRa-based RNode interface

### Antennas

#### Stationary antenna

The antenna with the magnetic mount would be connected to the one RNode that my laptop was connected to (over USB-C).

The antenna _itself_ was mounted ontop of the patio's shade-roof as it had a metallic base that was magnetic:

![](stat_antenna1.jpeg)

The above antenna connects to the RNode (for my laptop) via the long $3m$ cable that terminates with an SMA connectors on the board:

![](stat_antenna2.jpeg)

#### Mobile antenna

As for my mobile device running Sideband, that would be connected to my phone via Bluetooth Low Energy (BLE) but powered via my phone's USB-C port:

![](mobi_antenna_1.jpeg)

# Test

## Test 1 - _"Line of sight"_

The first test was to walk to a road parallel to the house and right across from the antenna mounted on the roof. This was chosen as it would provide line-of-sight for the antennas and be a good first test.

![](los_1.jpeg)

I began the test by opening up Sideband and then sending some messages:

![](msg_1.jpeg)

We can see that they were all delivered, this meant we received and acknowledgement from the laptop node!

I think the _last message_ was sent in the park actually (see the next test)

## Test 2 - _"obstructing concrete"_

The next test was sort of a mistake actually. I decided to walk back home through the same path I had come originally; this meant walking via the park.

![](walk_back_1.jpeg)

It was also a **very hot** day

![](walk_back_2.jpeg)

I was rather surprised here as there are multiple walls of concrete obstructing the stationary antenna and my mobile one. Yet, believe it or not, I was able to get some messages through here:

![](msg_2.jpeg)

As you can see these messages were **all** delivered.

I found these results rather exciting as it meant it can work with some obstructions; good to know because it means slighter obstructions - perhaps of tree - should definitely still allow for the signal to reach both ways.

![](walk_back_3.jpeg)

I do wonder if the waves were bouncing off the walls in some way - who knows? 🤔️

# Conclusion

I was rather happy with the outcome of the first test as it showed everything I had setup was in working order (the new antennas).

Secondly, the surprise of how well it worked with obstructions of the concrete walls was just mind boggling in the good sense of the word.

## Next plans

I have decided that the next place I want to test from is at a tree on top of a hill in the same line-of-sight as where I stood but just at a **much greater distance**.

![](next.jpeg)

In the mean time, I will plan on taking a trip to an estate that is on a hill and should have line-of-sight to my stationary antenna but at a further distance and accessible with a normal vehicle (instead of an off-road one which would be required for the tree on the hill site).