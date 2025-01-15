---
title: IPv6 Vinyl Pi but with Docker
author: Tristan B. V. Kildaire
date: 2023-06-29
draft: true
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

## Raspberry Pi

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

# Setup

We will now go through the setup in stages:

1. The pre-Docker stage
	a. Here we will configure the system's time, the IPv6 networking and installing and configuring the Docker daemon for our use
	b. We will also configure the system such that local user services ("system user units") start on bootup of our configured user *without* requiring said user to even login!
		i. We will need this because that is how PulseAudio is, by default (and trust me you don't want to spend the rest of your life time figuring out how to do it otherwise) is started
2. The containers
	a. In this we will setup three containers:
		i. The `vlc` container will connect to the PulseAudio server in order to get access to the input source (remember our USB capture card? It's that!). It will then stream this to our Icecast server.
		ii. The `restarter` container will check if the PulseAudio server has restarted _since_ the time _this_ container started. This is needed because, for some reason, `cvlc` doesn't seem to detect that the old UNIX domain socket was closed and hence stops working without ever restarting itself.
			a. This will check if the file has changed via checking the time of the creation date of the socket file, if it has changed since launch (or last restart) then we restart the `vlc` container and save the _new time_.
		iii. The `streamer` container will be running Icecast2 which is a web streaming platform that let's you push a stream of audio (or video) into it and then have multiple listener(s) listen/watch in/from it.
		iv. We won't be exposing our Icecast instance directly (even though we could and I just wanted to make this needlessly complicated) therefore we will throw the NGINX daemon infront of it, hence acting as a reverse proxy. This container will be called `web`.
			a. Actually there is a legitimate use case for this, if you want to do HTTPS then it is possible with NGINX - I don't think Icecast supports that (neither should it)
				1. **Correction** Icecast *can* do SSL but honestly it's easier to use NGINX

## Host setup

We now perform the pre-Docker stage of the setup.

### Enabling IPv6 and disabling IPv4

We will now want to do a few things.

We want to disable the WiFi interface from connecting to the network that RPi Imager set for us. We can disable the clout init stuff setup by RPi Imager by running:

```bash
sudo bash
echo "network: {config: disabled}" > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
exit
```

This will stop it from generating a file named `50-cloud-init.yaml`

Let's also delete that file now with:

```bash
sudo bash
cd /etc/netplan
rm 50-cloud-init.yaml
exit
```

We will now add a file in `/etc/netplan` called `ether.yaml` and it shall have the following contents:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      dhcp6: yes
  wifis:
    wlan0:
      access-points:
          LabNet_3:
              password: PasswordGoesHere
      dhcp4: false
      dhcp6: yes
```

I explicitly set `dhcp4` to `false` because I don't want it to even _try_ and get an IPv4 addressing/routing information.

Now we can apply all the changes by running:

```bash
sudo netplan apply
```

Your connection over the IPv4 link should now break and you must then connect over the IPv6 link. The address of which will be discoverable by a scan. That _or_ connect with link-local address (if you took it down before disconnecting).

### Time

Okay, if you run `timedatectl show-timesync` then you might get an output as follows:

```
LinkNTPServers=192.168.200.4
FallbackNTPServers=ntp.ubuntu.com
ServerName=192.168.200.4
ServerAddress=192.168.200.4
RootDistanceMaxUSec=5s
PollIntervalMinUSec=32s
PollIntervalMaxUSec=34min 8s
PollIntervalUSec=8min 32s
Frequency=0
```

Now the problem with this is that the main server it chooses to use has an IPv4 address, which means it will always use the fallback address. Or well, if we run `systemctl status systemd-timesyncd.service` we can actually see it is failing:

![image.png](image_1707653820736_0.png)

> It will fallback to a working `ntp.ubuntu.com` but I don't want that to be the behavior

Now to fix this we must edit `/etc/systemd/timesyncd.conf`, by default it looks like this:

```
[Time]
#NTP=
#FallbackNTP=ntp.ubuntu.com
#RootDistanceMaxSec=5
#PollIntervalMinSec=32
#PollIntervalMaxSec=2048
#ConnectionRetrySec=30
#SaveIntervalSec=60
```

Therefore we shall set the **main** NTP server to be `ntp.ubuntu.com` and as the the fallback server I shall use the `ntp.pool.org` server. The final configuration file shall appear as follows:

```
[Time]
NTP=ntp.ubuntu.com
FallbackNTP=ntp.pool.org
#RootDistanceMaxSec=5
#PollIntervalMinSec=32
#PollIntervalMaxSec=2048
#ConnectionRetrySec=30
#SaveIntervalSec=60
```

Now restart the service with `sudo systemctl restart systemd-timesyncd.service` as then run `sudo systemctl restart systemd-timesyncd.service` and you should see no more errors but _rather_ a successful sync:

![image.png](image_1707656744190_0.png)

> I _originally used_ `ntp.pool.org` but the resolved IPv6 addresses of the NTP servers I got kept timing out, after switching to the African pool `ntp.ubuntu.com` (as shown above), then things started working and **always** consistently; as shown in the screenshot. This is the reason for using Ubuntu's time server rather than those advertised by the NTP Pool Project's.

### Docker daemon

The setup of the Docker daemon begins by changing a single file located at `/etc/docker/daemon.json`. We shall change it as shown below:

```json
{
  "experimental": true,
  "ip6tables": true,
  "ipv6":true,
  "fixed-cidr-v6": "fd10:c8c6:3b63::/48"
}
```

#### The `fixed-cidr-v6` and `ipv6`

The provided ULA is for the `default` bridge network **only**. I will not be using this network but rather my own. It is, *however*, useful for **building images** because that occurs in containers as well, normally in the default network.

We don't _always_ build images, sometimes we re-use but it is important to have this setup and ready to go when needed

The `ipv6` flag is self-explanatory - enable IPv6 on the `default` bridge network.

I have used the subnet `fd10:c8c6:3b63::/48` as (you shall see later) we will use masquerading. This means every container using this network will get an IPv6 address assigned that lies within this subnet

#### The `ip6tables` and `experimental`

The `ip6tables` aids with port mapping when doing, what I suppose, is the masquerading/NAT'ing for container-out traffic (**NOT** in - we do this via a TCP proxy)

This feature, however, is only usable when `experimental` flag is set to `true`

---

After this has been completed then we must restart Docker with:

```bash
sudo systemctl restart docker
```

Make sure all is fine by running and checking the status:

```bash
sudo systemctl status docker
```

### Tooling

#### Container management

I don't like to use Portainer because it is a web interface which means just more strain on my hands to use a mouse and also worse UX performance in terms of getting things done.

I have therefore opted for console-based management - the best too, I have found is `sen` which let's one manage the running docker instance:

![image.png](image_1706712299483_0.png)

Installation is as simple as a `sudo apt install sen -y`

Then to use it you will need to run under a user which can read and write from the Docker UNIX domain socket, hence being `root` user will do - therefore use the following command to open `sen` up:

```bash
sudo sen
```

After this you will be greeted by a screen as shown above, hitting `h` will show the commands you can use

#### Composing

We're going to be writing Docker Compose files which specify which image to use and *how* to instantiate or "launch" a container with such an image and with what arguments, properties and so forth.

I will be using the latest version of Docker Compose, which is v2, rather than v1. Either should work, v2 just "looks cooler".

Installation is rather simple (for v1):

```bash
sudo apt install docker-compose -y
```

> Note, the command to use once installed is `docker-compose` - as it is its own separate program

For v2 (which I will be using):

```bash
sudo apt install docker-compose-v2 -y
```

> Note, the command to use once installed is actually via the `docker` binary; this package we added is simply a plugin. You will use `docker compose` (note no `-`; the arguments are the same or at least backwards compatible with `docker-compose`'s)

### PulseAudio and services

We will need to install the PulseAudio server as it doesn't come pre-installed on the Ubuntu Server edition of Ubuntu, obviously:

```bash
sudo apt install pulseaudio -y
```

After this we will have "system template" services present. I have no idea how these work, where they come from (other than the installed package) and how they appear at login consistently. Either way, they are there and they are always in the `enabled` state which implies that the `pulseaudio.service` will start whenever a new session is started.

A *session*? What's that. Well, when you login a new session is created. Presumably some systemd concept. Either way these user-based system unit then startup.

That's a **problem**. Well, to be clear not the part about them starting *upon* session start - in fact that automation, so to speak, is what we want. The *problem* lies within the fact that normally the only way to start a new session is to login either via the console (virtual console), serial console, SSH... - anything you get a tty. Now I don't want to login everytime I want to kick off my automation.

>If there is one manual part involved then the rest that is automated is **not automated**

The solution is actually rather simple, you just run:

```bash
loginctl enable-linger <username>
```

Okay, this is great but *what does that do*? Well, look no further (I read the frikken manual):

![image.png](image_1719412459268_0.png)

This will start a session for the given user *at boot* - which is exactly hat we want, remember? As this will kick off the starting of all systemd local-user services, *such as* the `pulseaudio.service` service.