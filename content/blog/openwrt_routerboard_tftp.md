---
title: "The generic how-to for getting OpenWRT onto a RouterBOARD"
author: Tristan B. Velloza Kildaire
date: 2025-08-08
draft: true
---

Setting up your Mikrotik with serial
====================================

I thought it would be instructive to put together
a guide on how I got my _RouterBOARD M33G_ setup
via a serial connection and how I used this to
access the internal bootloader known as RouterBOOT.

## Why RouterBOOT?

Well, RouterBOOT, being a boot loader, is the first
piece of code the CPU jumps to on boot and is what
runs first.

The bootloader can be hardcoded to just run some
code on some flash device but normally it has its
own configuration for boot devices and what order
to _probe_ them for bootable images.

I intend to use this to get the bootloader to
attempt to load an ELF image over the network
and boot into _that_ instead of the image
currently present on the flash storage chip.

## Serial console

Firstly obtain a serial-to-USB adapter which
let's you get a serial interface over USB.

Once you have that sorted out you can then
go ahead and attach the adaptor to your
computer. It will most likely appear as
a `/dev/ttyUSB0` device. Such devices are
part of the `dialout` group, hence you'll
either need to be part of that group _or_
run the upcoming command as root.

Next, install `picocom`. This is a serial
terminal which lets you read and write
from the serial device in an interactive
manner:

```
sudo dnf install picocom -y
```

Now we can open the serial device with:

```
picocom /dev/ttyUSB0 --databits 8 --stopbits 1 --parity n --baud 115200
```

The parameters are [provided by Mikrotik](https://wiki.mikrotik.com/Manual:RouterBOOT)
but are pretty much self-explanatory from
the named arguments above.

### TFTP booting

I have a shell utility based off of the [OpenWRT one](https://openwrt.org/docs/guide-user/installation/generic.flashing.tftp#mikrotik_routerboards)
which accepts, as an environment variable `INTERFACE`
which must be the Ethernet interface that we are
going to run the TFTP server over. This must also
be the interface which is connected to the RouterBOARD
directly or over some Ethernet switch.

The RouterBOARD must be connected on the `ether1`
interface in order for the TFTP boot to work.

#### Enabling TFTP mode

By default the firmware will always boot off
of the NAND device. For the initial boot,
which will be done via a network image
transfer into the RAM of the device, we need
to set it to use network boot.

To do this, first connect your serial
adaptor to the RouterBOARD and then
start the `picocom` command. _Then_
apply power to the RouterBOARD and
you should be greeted with a screen
as follows:

```

RouterBOOT booter 6.48.6

RBM33G

CPU frequency: 880 MHz
  Memory size: 256 MiB
 Storage size:  16 MiB

Press any key within 2 seconds to enter setup..
```

**Quickly** press any key on your
keyboard and then you should be
brought to a screen that shows
as follows:

```
RouterBOOT-6.48.6
What do you want to configure?
   d - boot delay
   k - boot key
   s - serial console
   n - silent boot
   o - boot device
   u - cpu mode
   f - cpu frequency
   r - reset booter configuration
   e - format storage
   w - repartition nand
   g - upgrade firmware
   i - board info
   p - boot protocol
   b - booter options
   t - test ram memory
   x - exit setup
```

There are quite a few options here but
the one that we are interested in is
the `o - boot device`, to select that
item press `o` on your keyboard. You
should then be brought to the following
screen:

```
Select boot device:
   e - boot over Ethernet
   n - boot from NAND, if fail then Ethernet
   1 - boot Ethernet once, then NAND
   o - boot from NAND only
   b - boot chosen device
 * f - boot Flash Configure Mode
   3 - boot Flash Configure Mode once, then NAND
```

We want to select the `e` option for
_boot over Ethernet_. Once you have
enter `e` you will then be brought
back to the main menu.

**Now**, wait before you do anything!

#### Preparing the TFTP environment

Firstly, disable your firewall. By
default it is probably blocking all
inbound traffic that "is not already
established" (like UDP/TCP/IP traffic
originating from you). The TFTP program
is a server and hence will need to
be able to listen for inbound
connections established _to_ it
(rather than from it) to work.

On my machine I was able to just stop
the firewall like this:

```bash
sudo systemctl stop firewalld
```

Now ensure that you have your Eternet
adaptor plugged into `ether1` on the
RouterBOARD. Then also ensure that it
has static IP addressing, this is _mainly_
a kludge to get `NetworkManager` to
stop resetting the addressing
information whenever it is non-static
(i.e. assuming a DHCP server will be
present). However, since we have no
DHCP server in TFTP mode we need
to set it to static. These steps
are dependent on your network
manager (if any).

Now go ahead and create the following
directory:

```bash
mkdir /tmp/images
```

In this directory you must put
all the image files (those would
be the `initramfs` ones).

Once that is done you will want
to run the `./flash.sh` script
as follows:

```bash
INTERFACE=ethWhatever ./flash.sh IMAGE_PATH
```

Set the `INTERFACE` environment
variable to whatever the name
of the Ethernet adaptor (the
one connected to `ether1` on
th RouterBOARD) on your machine.

Replace `IMAGE_PATH` with the name
of the image file that is stored
in `/tmp/images`. The image I
recommend using is `openwrt-21.02.0-ramips-mt7621-mikrotik_routerboard-m33g-initramfs-kernel.bin`
(you will see why below).

>Note: You cannot directly boot the image `openwrt-24.10.2-ramips-mt7621-mikrotik_routerboard-m33g-initramfs-kernel.bin`
as the entire image cannot fit into RAM. Remember that the WHOLE OS is loaded into memory, not just memory resident programs.
So we _will_ eventually be able to _install_ this version of OpenWRT but onto the NAND flash rather than booting into
into RAM **entirely**.

TODO: Is the above correct? When I upload an image to flash doesn't it store it into `/tmp` anyways? Hence it is
not a RAM issue because `tmpfs` is mounted at `/tmp`. Maybe the firmware loader just cannot handle the greater
image size?

```
RouterBOOT booter 6.48.6

RBM33G

CPU frequency: 880 MHz
  Memory size: 256 MiB
 Storage size:  16 MiB

Press any key within 2 seconds to enter setup..
trying bootp protocol.................. OK
Got IP address: 192.168.1.122
resolved mac address 00:E0:4C:36:08:CC
Gateway: 192.168.1.10
transfer started ............................................. transfer ok, time=6.31s
setting up elf image... kernel out of range
```

#### Performing the TFTP boot

Now run the `./muh.sh` script. Now
that it is running in the background
we can return to our RouterBOOT
menu. You can now type `x` which
will save the boot configuration
(to now use the network boot option)
and begin booting.

You should see something like this:


```
RouterBOOT booter 6.48.6

RBM33G

CPU frequency: 880 MHz
  Memory size: 256 MiB
 Storage size:  16 MiB

Press any key within 2 seconds to enter setup..
trying bootp protocol.... OK
Got IP address: 192.168.1.122
resolved mac address 00:E0:4C:36:08:CC
Gateway: 192.168.1.10
transfer started ..................................... transfer ok, time=4.62s
setting up elf image... OK
jumping to kernel code
[    0.000000] Linux version 5.4.143 (builder@buildhost) (gcc version 8.4.0 (OpenWrt GCC 8.4.0 r16279-5cc0535800)) #0 SMP Tue Aug 31 22:20:08 2021
```

### Flashing

Once booted remove your Ethernet cable from port `ether1`
and plug it into port `ether2` (which currently will be
bridged with `ether3`, so either will work).

There is a DHCP server running on that bridge which will
assign you an IP in the `192.168.1.0/24` subnet and the
router will be at `192.168.1.1`.

You can now visit `192.168.1.1` in your web browser.
The username is `root` and there is no password.

Once logged in go to the `System -> Backup/...` option,
once there the last section is the firmware flashing
section. Click the "Flash firmware" button, then
select the firmware from the pop-up. The firmware
image will **not** be the `initramfs` one but
rather the one ending in `.sysupgrade`.

>Note, here you can select the `openwrt-24.10.1-ramips-mt7621-mikrotik_routerboard-m33g-squashfs-sysupgrade.bin`
image and click upload. It should upload just fine.

Now click "Flash" and wait. This can take sometime
and you will **not** be given any progress updates
via the web interface. You _will_, however, be given
information via the serial console which helps in
seeing the progress.

After you see the device reboot, give it another
minute or two and then eventually you should see
it finished if you give the web page at `192.168.1.1`
a refresh - you should note that the login page's
styling has changed and, furthermore, when logged
in, you should notice that the version information
in the footer of the web UI now says `24.10.x`.

---

### The `flash.sh` script is shown below. It 
is an adaptation of the [original script](https://openwrt.org/docs/guide-user/installation/generic.flashing.tftp#mikrotik_routerboards)
which is licensed under the [CC Attribution-Share Alike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/deed.en).
The only changes I made was to allow the interface,
user, image root and image to be manipulated
via environment variables so that it can be
more easily reused.

Here is the script:

```bash
#!/bin/bash

# Original script based on: https://openwrt.org/docs/guide-user/installation/generic.flashing.tftp#mikrotik_routerboards
# which uses the 'CC Attribution-Share Alike 4.0 International' license
#
# This includes a few changes to use environment variables
# to specify the various options

USER=deavmi
IMAGE=$1

if [ "$INTERFACE" = "" ]
then
	INTERFACE=enp0s20f0u1
fi

if [ "$IMAGE_ROOT" = "" ]
then
	IMAGE_ROOT=/home/deavmi/Documents/Builds/openwrtflasher
fi

echo "Images directory is: '$IMAGE_ROOT'"
echo "Image to netboot with is: $IMAGE"
echo "TFTP server will be available on the $INTERFACE interface"

sudo ip addr replace 192.168.1.10/24 dev $INTERFACE
sudo ip link set dev $INTERFACE up
sudo dnsmasq -i $INTERFACE --dhcp-range=192.168.1.100,192.168.1.200 \
--dhcp-boot=$IMAGE \
--enable-tftp --tftp-root="$IMAGE_ROOT" -d -u $USER -p0 -K --log-dhcp --bootp-dynamic
```
