---
title: "Building my first RNode: Part 1"
author: Tristan B. V. Kildaire
date: 2025-01-13
draft: false
---

## What is it?

An RNode is any [supported device](https://unsigned.io/guides/2022_01_25_installing-rnode-firmware-on-supported-devices.html) (see the _"Device variations"_ section) that runs Mark Qvist's firmware.

What does the firmware let you do?

1. LoRa sniffer
2. LoRa TNC
    a. Think of a point-to-point IP link over LoRa
    b. This will be covered in part 2
3. LoRa interface for a Retciulum node

You can see all of this in the _"One tool, many uses"_ section [here](https://unsigned.io/hardware/RNode.html).

## Parts list

![](1.jpeg)

![](2.jpeg)

I decided to purchase 2 of the **LillyGo T3-S3**'s which make use of the SX1276 chipset.

* This is important as this is a chipset which has support in the RNode firmware.
* The model I am buying is the 868Mhz model as that was all I could get my hands on in ZA

I specifically bought 2 because I doubt there is anyone in my area running the RNode firmware and doing Reticulum over it. Secondly, _even_ if there was someone then I wouldn't have full control over my tests.

This device can be purchased locally [here](https://www.robotics.org.za/development-boards/esp32-lilygo-boards/H596).

## Flashing the firmware

I have documented two ways in which one can install the firmware onto your device. The reason for this was because I initially had some issues using the auto-installation feature of `rnodeconf` - it would flash but wouldn't fully perform all the needed parameter setting on my device.

#### Prerequisites

Firstly you will need to install the `nrs` package using `pip`. This package will make the `rnodeconf` executable available for you, which will allow us to both **install the firmware** _and_ **configure our RNode** (as multiple modes exist).

Installation can be performed with:

```bash
pip3 install rns -y
```

#### Serial device access

When we plug in our device it will appear as some sort of `/dev/ttcyACMx` device (where $x$ is some number). This device type is a serial device and in order to get access to all-and-any serial device one must either be a part of the `dialout` group or `root`. The latter is overkill and I would much rather we use an approach that **doesn't** require us running the `rnodeconf` tool as a user other than our own.

To add yourself to the `dialout` group run the following command to append (`-a`) a new group (the `-G`) to your current user:

```bash
sudo usermod -aG dialout deavmi
```

In my case my username is `deavmi`.

#### Flashing the firmware

The first step is to plug in your device. **However**, whilst plugging it in via the USB-C port you must be holding the `Boot` button on the board:

![](3.jpeg)

Now you will need to run `dmesg -w` (you might need to run it as `root`). This is just so we can see the device path of the serial device we just plugged in:

![](4.png)

As you can see mine is `/dev/ttyACM1`

>**Note:** Before we begin, ensure that no other application is using this serial device. I had that happen to me by mistake (as I had `rnsd` running and pointed to my RNode at the same device path). It will not work if something else is using the serial device at the same time (specifically if it is actively communicating with it whilst `rnodeconf` is _also_ trying to)

Let's now begin the installer with the command `rnodeconf /dev/ttyACM1 -a`. Next we will be asked a few questions:

1. You will be asked which device you are using, in my case it is the `[9] LilyGO LoRa T3S3` option.
2. Next you will be asked to select the band that the radio is to be configured in. In my case that would be the 868Mhz band, so I must select the `[2] 868/915/923 MHz (with SX1276 chip)`
    a. Also note that there are normally two options **per** band, I went with this option as I am using the `SX1276` chipset as mentioned earlier
3. Now you will be presented with the following screen, check that everything is in order before hitting any key:

```
Confirm detals:

Serial port     : /dev/ttyACM1
Device type     : RNode 820 - 960 MHz
Platform        : ESP32
Device MCU      : Espressif Systems ESP32
Firmware file   : rnode_firmware_t3s3_sx127x.zip
```

**Note:** The tool only seems to support downloading with `AF_INET` (meaning that IPv6 support doesn't work)

Now hit enter. Please note that, at least for now, the installation is not complete when it finishes and it won't reboot the device automatically for you. Once you get the message that the installer failed to do so, then you can power cycle your device.

#### Finishing up

Because there is a slight bug in the installer we now need to perform the following.

Firstly, when you boot the device you will get a nasty warning that the "hardware failure":

![](5.jpeg)

To fix this we first completely wipe the data that says what model the firmware is expecting and replace it with what it _should_ expect (our model details); hence getting rid of the mismatch of models. Before we begin you must hit the `Reset` button:

![](6.jpeg)

Now we can run the command:

```bash
rnodeconf --eeprom-wipe /dev/ttyACM1 # Wipe old settings
rnodeconf -r --model aa --product f0 --hwrev 1 /dev/ttyACM1 # Set the CORRECT model information
```

Now we will have _another_ error, this one occurs because the firmware has a hash digest generated for it and then one stored in memory as well. If the `hash(firmware_current) != hash(firmware_expected)` then a UI warning appears:

![](7.jpeg)

This can be fixed by obtaining the hash of our _current firmware_ and then overwriting the _expected hash value_ parameter; therefore making them match and making that UI error disappear. First let's get the current hash with:

```bash
rnodeconf /dev/ttyACM1 -L
```

Now set the hash from the previous command's output with:

```bash
rnodeconf /dev/ttyACM1 -H <hash>
```

Afterwards we shall see that the warning disappears and we should be left with a screen that appears as follows:

![](8.jpeg)

#### Bring it all together

![](9.jpeg)

We are now done and have a working RNode 🎊️!