---
title: Vinyl Pi
author: Tristan B. V. Kildaire
date: 2023-06-29
draft: false
---



## Hardware
Before you can continue with the next steps I should mention a few things.

### Choosing a capture card

I cannot guarantee this will work on your device but I can say my audio capture device worked on Linux. It cost me about R1000 so don't go cheap on it, I am just assuming that expensive increases likelihood of compatibility but honestly I may very well be wrong.

For me the importance is that it sounds good and this capture card does sound good and **it has pass-through**. I didn't want a capture card without pass-through as I still have a vintage Tedelex system I want to connect to - the stream is just an aside.

I went with this card (shown below). You can get it on [Takealot](https://www.takealot.com/pdm-pdx015-usb-phono-pre-amplifier-for-turntables/PLID53021834) for roughly R1000.
![](vinyl/dac_1.jpeg)

### The doodads

You will need two doodads:

1. A powered USB hub
	* We don't want to try powering the capture card via the Raspberry Pi Zero, best we power it through a wall-powered USB hub
	* ![](/img/vinyl/powered_usb_hub.jpeg)
1. A USB-to-OTG connector
	* This is basically a large USB female to micro USB male connector which is supported on the Pi
	* ![](/img/vinyl/node.jpeg)

### Raspberry Pi Zero

We need a Linux machine we can do this all on - nothing less will be appropriate. Linux is great and we shall use it.

The specific version I have is the `TODO: find the revision here`

### Flashing the Pi

You can use the Raspberry Pi `rpi-flasher` (I ran it using `sudo` because the access rights didn't work even with a running policy kit like `lxpolkit`).

Once you have it open you will want to navigate to the "Other section"
![](/img/vinyl/other.png)

Then we want to go to the "Ubuntu" section:
![](/img/vinyl/ubuntu_section.png)

Lastly, click on this one - this is the image we will be using for our Raspberry Pi Zero:
![](/img/vinyl/rpi_image.png)

**Now** you will want to click the little configuration button at the bottom and configure the few settings you can to your needs. I decided it would make sense to get SSH configured for password access now along with configuring the Pi's settings such that on boot it would join my home network and then lastly the user and password to create:
![](/img/vinyl/rpi_setup.png)

> By the way, setting the hostname is nice as you will be able to (if your machine has Avahi - most distros do) discover your device's IP by doing `ping vinylpi.local`.


## Audio servers, device locks, and *all that jazz*
Prior to audio servers an application would get sole access (a so-called *"lock"*) on an audio device be it an input (source) or an output (sink). You can imagine how this wasn't very great.

If you wanted to share the audio from a microphone you couldn't, only one app at a time. If you wanted to play a YouTube video and also listen to background music? No luck, only one process can lock an audio device at a time.

### How was this fixed?
Well, the one device lock is still a thing but people got smart - why not let the process which locks the device provide multiple streams for it, on-demand of other processes? Well, they did that.

PulseAudio is one such server, it locks the audio devices connected to the system and makes them available to other processes **via** pulse audio inter-process communication. So many programs are written to use PulseAudio these days and it makes life so much more easier.

### Settuing up PulseAudio
So what we're now going to install is the PulseAudio server for Linux, we can do so easily with the command below:

```bash
sudo apt install pulseaudio pulseaudio-utils
```

The `pulseaudio-utils` package gives us a tool known as `pactl` which let's us inspect the running pulse audio server (installed by  the `pulseaudio` package).

### Start the audio server
We now need to start the ouklse audio server. This will discover all audio input and output devices and get a lock on them. It iwll multiplex these out to clients wanting them via IPC mehcnaics.

```bash
pulseaudio  -v
```

I pass the `-v` for a verbosity level of `1` just so I can see what it is busy doing.

> Once we reboot it will start automatically but for now we do this

### Device check
You should be able to see your device now, run the command:

```bash
pactl list sources
```

This will list all so-called *"sources"* which are, well, sources of audio - think input devices such as microphones or this capture card. You should see something resembling the name of the hardware device.

This is what I got back from mine:

```
Source #1
        State: SUSPENDED
        Name: alsa_input.usb-C-Media_Electronics_Inc._USB_Audio_Device-00.analog-stereo
        Description: USB Audio Device Analog Stereo
        Driver: module-alsa-card.c
        Sample Specification: s16le 2ch 44100Hz
        Channel Map: front-left,front-right
        Owner Module: 7
        Mute: no
    
		...
```

## Networking
We're going to now quickly shift focus to the networking side of things. After all that *is* what this project is all about.

### Getting network information
Firstly I would like to figure out what the IP address is - ever since installing I was doing a DNS-based login (I could `ping vinylpi.local`) but seeing that we're in now we may as well do this because we will need it later.

We need to figure out our IP, let's get the good-ol' `ifconfig` back by installing the `net-tools` package as follows:

```bash
sudo apt install net-tools -y
```

After this run the command `ifconfig` and you should get something like the following printed out:
```
wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.137  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::e65f:1ff:fe44:fdc4  prefixlen 64  scopeid 0x20<link>
        ether e4:5f:01:44:fd:c4  txqueuelen 1000  (Ethernet)
        RX packets 128791  bytes 167971951 (167.9 MB)
        RX errors 0  dropped 614  overruns 0  frame 0
        TX packets 45948  bytes 4292901 (4.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

From this we can tell our IP address is **192.168.1.137** on the WiFi interface `wlan0`.

### A little test stream
Before we get to setting up the proper streaming server let's first test the whole setup. Now obviously we *have to* test ti via the network as we don't have any audio ports on the Raspberry Pi Zero. We can setup a VLC stream server which captures an audio feed from a Pulse Audio device such as ours - so let's do that.

Begin by installing VideoLAN as follows:
```bash
sudo apt install vlc -y
```

Now we can use the command (from [StackOverflow](https://superuser.com/questions/874631/streaming-pulseaudio-to-vlc-via-null-sink)) that follows.

```bash
vlc -vvv pulse://alsa_input.usb-C-Media_Electronics_Inc._USB│
_Audio_Device-00.analog-stereo --sout '#transcode{acodec=vorbis,ab=128,channels│
=2}:standard{access=http,dst=0.0.0.0:8888/audio.ogg}'
```

The device I pass is `alsa_input.usb-C-Media_Electronics_Inc._USB_Audio_Device-00.analog-stereo` as that is the name I got from the previous section when we found it using `pactl list sources`. The stream will be encoded using Vorbis (the OGG format)
 
The `access=http,dst=0.0.0.0:8888/audio.ogg` part means that VLC will start an HTTP server bound to all IPv4 addresses, listening on port **8888** and available at the path `/audio.ogg`.

---

Now on your machine type in `vlc http://192.168.1.137/audio.ogg` and you should hear it!

Or you can do it via the UI:
![](/img/vinyl/vlc_client.png)

And then you can listen away:
![](/img/vinyl/vlc_client_listen.png)

## Professional-grade streaming server
Icecast is a streaming server that allows multiple sources to stream or *push data* into it. These can then later be consumed by several clients. An audio stream is denotaed by a so-called *"mount point"* whereby one can stream audio **to** and have multiple clients connect and stream audio **from** it.

Let's start by installing the icecast server software:
```bash
sudo apt install icecast2 -y
```

### Initial configuration
During the installation it will ask you to configure some basics. We go through these here.

Firstly it wants to know the domain name to use, if you have all that stuff set up then by all means place that domain name below. I, however, will be going with just `localhost`. (This won't affect reachability - don't worry).
![](/img/vinyl/hostname.png)

You will then be asked to provide a password for the *"source"*. This is the password used to protect your server against others from streaming **to** it.
![](/img/vinyl/source_password.png)

We won't be setting up a relay (TODO: what is this)
![](/img/vinyl/relay.png)

Once completed the installation will finish up.

### Configuring the network
The configuration file is located at `/etc/icecast2/icecast.xml` and we shall be editing that.

First we want to configure our Icecast server to be accessible on all IP addresses (v4 and v6) therefore look for the `<listen-socket>` option and set that block as follows:
```xml
<listen-socket>
	<port>8000</port>
	<bind-address>::</bind-address>
	<!-- <shoutcast-mount>/stream</shoutcast-mount> -->
</listen-socket>
```
This will listen on all IPv4 and IPv6 addresses and be available on HTTP port **8000**.

If you ever need to change the hostname (it's used to generate URLs for playlist files mainly) then you can with this tag:
```xml
<!-- Hostname -->
<hostname>localhost</hostname>
```

### Authentication
The `<authentication>` block (found near the top of the file) let's configure two things we care about:

1. The administrator console
	* This is used for managing who is listening to what stream, kicking users and inspecting information in general
2. The source authentication
	* This let's us configure the credentials required for a source program (such as VLC - as we shall see in the next section)
	* This is to prevent any old John from streaming their terrible taste of music to my server (that would be modern day rap)

```xml
<!-- Authentication details -->
<authentication>
	<!-- Administrator username/password -->
	<admin-user>admin</admin-user>
	<admin-password>ohmylordfrongod</admin-password>

	<!-- Source credentials -->
	<source-password>ohmylordfrongod</source-password>

	<!-- Relays log in with username 'relay' -->
	<!-- Don't worry about this, we won't be setting up access
		 for relays to connect to us
	-->
	<relay-password>frongod</relay-password>
</authentication>
```

### Administrator details

We can also customise some of the information about our Icecast server as follows:

```xml
<!-- Administrator details -->
<location>Worcester</location>
<admin>southwestafrica@leagueofnations</admin>
```

---

Now that we have setup everything we need setup, let's restart our Ice cast server to apply the changes:
```bash
sudo systemctl restart icecast2
```

Now you should be able to reach the Icecast landing age by pointing your browser to `http://your-ip:8000`. A page should appear that looks something like this:
![](/img/vinyl/icecast_dashboard.png)

>You won't see a stream there *just yet* - I was a bit hasty and configured that prior to writing this section

### VLC stream source

We now need to setup an IceCast *"source"* which will stream audio **to** the Icecast server, therefore making it available for consumption by client listeners (as you see in the screenshot above).

Now what I have put together below is an amalgamation of my own code, [this](https://superuser.com/questions/874631/streaming-pulseaudio-to-vlc-via-null-sink) StackOverflow post and the [arch linux wiki](https://wiki.archlinux.org/title/Icecast). I have just basically parameterised the things we care about:

1. `PULSE_SOURCE_DEVICE`
	* Fill this with the PulseAudio device name of your capture card
2. `ICECAST_HOSTNAME`
	* The address and port in the format "hostname:port" of your Icecast server (the HTTP server)
3. `ICECAST_MOUNT_POINT`
	* The mount point to stream to
	* **TODO**: Mention that we can setup a permanent one too
4. `ICECAST_PASSWORD`
	* The password for being a source

Here is the aforementioned script, `vinyl.sh`:
```bash
#!/bin/sh

# The PulseAudio device name (our capture card)
PULSE_SOURCE_DEVICE="alsa_input.usb-C-Media_Electronics_Inc._USB_Audio_Device-00.analog-stereo"

# Icecast Details
ICECAST_HOSTNAME="[::1]:8000"
ICECAST_MOUNT_POINT="stream.ogg"
ICECAST_PASSWORD="ohmylordfrongod"

# Start VLL capturing our PulseAudio device and push that data to Icecast
cvlc -vv "pulse://$PULSE_SOURCE_DEVICE" ":sout=#transcode{vcodec=
none,acodec=vorb,ab=192,channels=2,samplerate=44100,scodec=none}:std{access=shout,mux=ogg,dst=//source:$ICECAST_PASSWORD@$ICECAST_HOSTNAME/$ICECAST_MOUNT_POINT}" ":no-sout-all" ":sout-keep"
```

You can now run the above with:

```bash
chmod +x vinyl.sh
./vinyl.sh
```

You should see something like this:
![](/img/vinyl/vlc_source.png)

---
#### Starting VLC on boot

TODO: This whole introduction to this section needs reworking: Need to explain why the current setup just won't work with a `vinyl.service`.

I, however, have decided to create a systemd unit for it so that it can start on boot. Now, pulse audio comes configured in thsi God awful manner for starting it. It starts only on-demand and only as a user service.  So the steps below are from my friend [rany2](TODO: add link) who has provided me with help on getting this to work.

Firstly we are going to enable a mode which starts all user services of a given user even prior to them logging inCreate a file named as follows `/etc/systemd/system/vinyl.service` and with the following contents:

TODO: Expalin why
```bash
loginctl enable-linger deavmi
```

TODO: explain why 
```bash
systemctl --user edit --force --full vlc.service
```

Explain these
```unit
[Unit]
Description=Vinyl VLC stream
Wants=icecast2.service
After=icecast2.service
Wants=pulseaudio.service
After=pulseaudio.service

[Service]
Environment="PULSE_SERVER=/run/user/1000/pulse/native"
ExecStart=/home/deavmi/vinyl.sh
Restart=always
TimeoutStopSec=5

[Install]
WantedBy=default.target
```


(A quick shoutout to [rany2](TODO: link) for helping realise that the environment variable indicating to VLC which OulseAudio server to use - was missing)
TODO: Need explanation
```bash
systemctl --user unmask pulseaudio.{service,socket}
systemctl --user enable --now pulseaudio.{service,socket}
```

Now let's reload systemd (to make it aware of a new unit file), enable it and start the it:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now vinyl
```

You have now reached the final step. Let's reboot our Pi with:
```bash
sudo reboot
```
And give it sometime as Ubuntu on the RPi Zero (and in general) takes some time to boot, once it does you should find your stream on the Icecast landing page showed earlier and should be able to listen to it

## Monitoring
If you would like to monitor several aspects of the network then there are a few options.

### On-device monitoring
We can use a program like `htop` to monitor the system state (and we can also see that the `vlc` process is what will take a good amount of CPU time):
![](/img/vinyl/htop.png)

You could also install `bmon` to examine the network traffic caused by users connecting to listen in:
![](/img/vinyl/network.png)

### Remote monitoring
We mentioned that there is an administrator portal that you can login using your administrator username and password. Clicking on the ***administration*** link will bring you there.

One of the screens you will see will show you the active listeners for the current streams/mount points you are hosting:
![](/img/vinyl/active_listeners.png)
This can be very useful if you want to kick any users off if they are maybe hogging too much bandwidth or perhaps if you don't want to subject them to your terrible taste in music because we all know ***Deavmi's is the best***.

You can also see the general administrator page which shows you server related configuration:
![](/img/vinyl/general_admin.png)

Happy listening! 🪩🕺

---

## Extras
### Yggdrasil-access
You may want to make your machine available on a network other than your private one. A fun little (well not little, [it's rather big](http://51.15.204.214/)) network is that of [Yggdrasil](https://yggdrasil-network.github.io/) which uses a private range of IPv6 addresses (only available on Yggdrasil).

Ubuntu has Yggdrasil in its repositories, therefore it's as easy as:

```bash
sudo apt install yggdrasil -y
```

We need to create a configuration file as one is not created on installation, we can get a blank template by doing the following as root:
```bash
yggdrasil -genconf > /etc/yggdrasil/yggdrasil.conf
exit
```

We then are required to add some peers. You can find some [here](https://github.com/yggdrasil-network/public-peers), once you have done so add them as follows:
```toml
Peers: [
    # Online node
    tls://54.37.137.221:11129
]
```

We will now stop the Yggdrasil service (as later we shall restart it):
```bash
sudo systemctl stop yggdrasil
```

We will want to enable the `yggdrasil.service` such that it can start on boot (the optional `--now` flag means to also start the service whilst we're at it):
```bash
sudo systemctl enable yggdrasil.service --now
```

Now confirm that it is running by checking if we have a new virtual network device `tun0`:
```bash
ifconfig tun0
```

Which should return something like this:
```
tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 53049
        inet6 fe80::dada:716d:ce33:290d  prefixlen 64  scopeid 0x20<link>
        inet6 202:558a:b17c:ac4f:c039:717f:2b2a:bf0b  prefixlen 7  scopeid 0x0<global>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3  bytes 144 (144.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

deavmi@vinylpi:~$
```
We can see our Yggdrasil IP address is `202:558a:b17c:ac4f:c039:717f:2b2a:bf0b`.

You should be able to reach your Icecast web interface on the same port now! So mine would be **[http://202:558a:b17c:ac4f:c039:717f:2b2a:bf0b]:8000**.
