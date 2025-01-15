---
title: IPv6 Vinyl Pi but with Docker
author: Tristan B. V. Kildaire
date: 2025-01-15
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

## Containers

Let's now get into the thick of it regarding containers, we will start with the `streamer` container first, then work our way to the `web` container and then end up lastly with the `vlc` container.

Let's first get a basic directory structure going with the following:

```bash
cd ~
mkdir src/
cd src # We're going to be using docker compose from within here

mkdir streamer/
mkdir web/
mkdir vlc/
mkdir hecker/

touch compose.yml
```

Now let's create a basic Docker Compose file by creating a file named `compose.yml` with the following contents:

```yaml
version: "3"

networks:
  mainNet:
    driver: bridge
    name: mainNet
    enable_ipv6: true
    ipam:
      config:
        - subnet: fde4:492a:fc7f::/48

services:
  vlc:
    # TODO: Fill in
  streamer:
    # TODO: Fill in
  web:
    # TODO: Fill in
```

### Setting up `streamer`

We first need to setup the Icecast server, let's start off with the configuration file which should be created in `streamer/icecast.xml`:

```xml
<icecast>
    <location>Worcester</location>
    <admin>deavmi@redxen.eu</admin>

    <limits>
        <clients>100</clients>
        <sources>2</sources>
        <queue-size>524288</queue-size>
        <client-timeout>30</client-timeout>
        <header-timeout>15</header-timeout>
        <source-timeout>10</source-timeout>
        <!-- If enabled, this will provide a burst of data when a client
             first connects, thereby significantly reducing the startup
             time for listeners that do substantial buffering. However,
             it also significantly increases latency between the source
             client and listening client.  For low-latency setups, you
             might want to disable this. -->
        <burst-on-connect>1</burst-on-connect>
        <!-- same as burst-on-connect, but this allows for being more
             specific on how much to burst. Most people won't need to
             change from the default 64k. Applies to all mountpoints  -->
        <burst-size>65535</burst-size>
    </limits>

    <authentication>
        <!--
                This password must also be set as the `ICECAST_PASSWORD`
                environment variable for the `vlc` container
        -->
        <source-password>l33picStreamer</source-password>

        <!--
                Admin logs in with the username given below

                Ensure you set the below password to something
                strong because the admin panel gives control
                over your Icecast server
        -->
        <admin-user>username</admin-user>
        <admin-password>password</admin-password>
    </authentication>

    <hostname>vinyl.deavmi.assigned.network</hostname>

    <listen-socket>
        <port>8000</port>
        <bind-address>::</bind-address>
    </listen-socket>

    <!-- Global header settings
         Headers defined here will be returned for every HTTP request to Icecast.

         The ACAO header makes Icecast public content/API by default
         This will make streams easier embeddable (some HTML5 functionality needs it).
         Also it allows direct access to e.g. /status-json.xsl from other sites.
         If you don't want this, comment out the following line or read up on CORS.
    -->
    <http-headers>
        <header name="Access-Control-Allow-Origin" value="*" />
    </http-headers>

    <fileserve>1</fileserve>

    <paths>
        <!-- basedir is only used if chroot is enabled -->
        <basedir>/usr/share/icecast2</basedir>

        <!-- Note that if <chroot> is turned on below, these paths must both
             be relative to the new root, not the original root -->
        <logdir>/var/log/icecast2</logdir>
        <webroot>/usr/share/icecast2/web</webroot>
        <adminroot>/usr/share/icecast2/admin</adminroot>
        <!-- <pidfile>/usr/share/icecast2/icecast.pid</pidfile> -->

        <!-- Aliases: treat requests for 'source' path as being for 'dest' path
             May be made specific to a port or bound address using the "port"
             and "bind-address" attributes.
          -->
        <!--
        <alias source="/foo" destination="/bar"/>
        -->
        <!-- Aliases: can also be used for simple redirections as well,
             this example will redirect all requests for http://server:port/ to
             the status page
        -->
        <alias source="/" destination="/status.xsl"/>
    </paths>

    <logging>
        <accesslog>access.log</accesslog>
        <errorlog>error.log</errorlog>
        <!-- <playlistlog>playlist.log</playlistlog> -->
        <loglevel>3</loglevel> <!-- 4 Debug, 3 Info, 2 Warn, 1 Error -->
        <logsize>10000</logsize> <!-- Max size of a logfile -->
        <!-- If logarchive is enabled (1), then when logsize is reached
             the logfile will be moved to [error|access|playlist].log.DATESTAMP,
             otherwise it will be moved to [error|access|playlist].log.old.
             Default is non-archive mode (i.e. overwrite)
        -->
        <!-- <logarchive>1</logarchive> -->
    </logging>

    <security>
        <chroot>0</chroot>
        <!--
        <changeowner>
            <user>nobody</user>
            <group>nogroup</group>
        </changeowner>
        -->
    </security>
</icecast>
```

The main important take aways from the config above:

1. The `<location>` and `<admin>` fields are just visual fields, they are not authentication related
2. The `<source-password>` is the password you will tell the `vlc` container to use so that is can authenticate (login) to our Icecast server and start streaming
	a. > Note, **set these!** or else someone can come in and start streaming to your server too 🤡️
3. The `<admin-*>` fields are username and password respectively used to protect the Icecast admin panel from unauthorized access.
	a. > Note, **set these!** or else someone can come in and kick listeners off and control your Icecast server
4. The `<hostname>` can be set to anything but note that the M3U (and the other format I cannot recall) buttons will generate playlists for this stream using it. It is not necessary to have it set for the normal stream (via the web element) to work however.
5. The `<listen-socket>` controls the address and port the Icecast process will bind to within our container. We set this to all IPv6 addresses, **not** `::1` as localhost in inaccessible from the host `docker-proxy` to something effectively remotely running over a network (a `veth`). The port is self-explanatory.
6. The rest we leave as is, except the `<logging>` one which controls the logging. Here we set them to `access.log` and `error.log` which will be, as you will see in the service file, bindfs-mounted to the host's `/dev/null`.

> Note, I obtained this file from https://github.com/xiph/Icecast-Server/blob/master/conf/icecast.xml.in and modified it to trim a lot of the fat off.

Now let's look a the Dockerfile which will build our custom image, this should look as follows and be stored at `streamer/Dockerfile`:

```Dockerfile
# Base image
FROM debian:bookworm AS base

# Don't allow interactive prompts when using apt
ARG DEBIAN_FRONTEND=noninteractive

# Upgrade system
RUN apt update
RUN apt upgrade -y

# Install icecast2
RUN apt install icecast2 -y

# For healthcheck
RUN apt install curl -y

# Setup user to run Icecast
RUN groupadd iceuser --gid 1003
RUN useradd iceuser -m --uid 1000 --gid 1003
WORKDIR /home/iceuser

# Delete old config and install
# one from build directory
RUN rm /etc/icecast2/icecast.xml
COPY icecast.xml /etc/icecast2/icecast.xml

# Make all files here accessible
# to our `iceuser` user
RUN chown iceuser:iceuser -v -R /etc/icecast2

# Switch to the Icecast user
USER iceuser

# Run icecast server with our custom config
CMD ["icecast2", "-c", "/etc/icecast2/icecast.xml"]

# Ensure that the Icecast web interface is
# up
HEALTHCHECK CMD curl http://[::1]:8000
```

Now for the service definition which will be a part of the `compose.yml` file:

```yaml
  streamer:
    build:
      context: streamer/
    container_name: streamer
    restart: unless-stopped
    networks:
      - mainNet
    volumes:
      # Instead of editing the Icecast file
      # just let it write to its defaults
      # but make it go nowhere
      - /dev/null:/var/log/icecast2/access.log
      - /dev/null:/var/log/icecast2/error.log
```

It is straight forward here:

1. Now recall we must place it on `mainNet` as our `vlc` and `nginx` containers will both need to reach the Icecast server process son port `8000` (recall the config we just configured?)
2. The bind-mounts are for the logs as explained earlier

### Setting up `vlc`

Now we will setup the container which runs a VLC process and connects to our host's PulseAudio server via a UNIX domain socket was pass into it:

```yaml
  vlc:
    build:
      context: vlc/
    container_name: vlc
    depends_on:
      # VLC would shit-itself after sometime if it cannot
      # connect to the Icecast endpoint
      - streamer
    restart: unless-stopped
    networks:
      - mainNet
    environment:
      # Name of the PulseAudio device
      - PULSE_SOURCE_DEVICE=alsa_input.usb-C-Media_Electronics_Inc._USB_Audio_Device-00.analog-stereo

      # PulseAudio server path (container-side)
      - PULSE_SERVER_PATH=/run/user/1000/pulse/native

      # Icecast Details (will match those of the following
      # container)
      - ICECAST_HOSTNAME=streamer:8000
      - ICECAST_MOUNT_POINT=stream.ogg
      - ICECAST_PASSWORD=l33picStreamer
      
      # Enable verbose logging for VLC
      - VLC_VERBOSE=3
    volumes:
      # Should be the host's Pulse Audio UNIX socket (mapped)-> UNIX sock on container side
      - /run:/run

      # Makes timezone and time information available
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
```

A few things to take note of here:

1. The `depends_on` is set so that we only start the VLC process (in our container) when the `streamer` container is up, hence meaning we have a valid Icecast endpoint to stream to.
2. The environment variables:
	a. The `ICECAST_HOSTNAME` must be left as is.
	b. The Icecast mount point, `ICECAST_MOUNT_POINT`, *could change* (I am not sure if you should change it but I would leave it as is).
	c. The `PULSE_SERVER_PATH` must be the path that will be used on the container-side, i.e. where the container will see the PulseAudio UNIX domain socket
	d. The `PULSE_SOURCE_DEVICE` must be set to the name you find when running `pactl list sources` and find your device. It must be the `node.name` field.
3. The volume mounts:
	a. I mount some time information although I don't think it is much important at all.
	b. The important mount point is `/run`. I would have done `/run/user/1000/pulse` **but** that only comes into existence at some time after Docker (which starts I believe way before any user services) when the user session (remember the `enable-linger` thing) starts. Therefore `pulse/` won't work as the `pulseaudio.service` local-user systemd unit (service) must first start, and more so, the `/run/user/1000` only comes into *existence* when the user with uid `1000` has a session started. I am unsure about `user/` being dependant on at least one user session being active hence `/run` was the safest bet.
4. Network, lastly it is on the same network `mainNet`.

Now let's look at what is in the `vlc/` directory:

The script which starts up VLC is shown below and stored at `run.sh`:

```bash
#!/bin/bash

echo "-----------------------------------------------------"
echo "Chosen PulseAudio device name: $PULSE_SOURCE_DEVICE"
echo "PulseAudio server path: $PULSE_SERVER_PATH"
echo "Icecast hostname: $ICECAST_HOSTNAME"
echo "Icecast mountpoint: $ICECAST_MOUNT_POINT"
echo "Icecast password: $ICECAST_PASSWORD"
echo "-----------------------------------------------------"

# Set to the path of PulseAudio's UNIX domain socket
echo "Using PulseAudio server at UNIX domain socket: $PULSE_SERVER_PATH"

# Set environment variable VLC uses to determine
# the PulseAudio server's address
export PULSE_SERVER=unix:/$PULSE_SERVER_PATH

# Start VLC capturing our PulseAudio device and push that data to Icecast
cvlc "pulse://$PULSE_SOURCE_DEVICE" ":sout=#transcode{vcodec=
none,acodec=vorb,ab=192,channels=2,samplerate=44100,scodec=none}:std{access=shout,mux=ogg,dst=//source:$ICECAST_PASSWORD@$ICECAST_HOSTNAME/$ICECAST_MOUNT_POINT}" ":no-sout-all" ":sout-keep"
```

This just sets the `PULSE_SERVER` environment variable prior to starting `cvlc` so that it knows where to locate the PulseAudio UNIX domain socket (so it can communicate with the PulseAudio server).

The `Dockerfile` is fairly simple as shown below:

```Dockerfile
# Base image
FROM debian:bookworm AS base

# Don't allow interactive prompts when using apt
ARG DEBIAN_FRONTEND=noninteractive

# Upgrade system
RUN apt update
RUN apt upgrade -y

# Install vlc
RUN apt install vlc -y

# Setup user to run VLC
RUN groupadd vlcuser --gid 1003
RUN useradd vlcuser -m --uid 1000 --gid 1003
WORKDIR /home/vlcuser

# Copy run script here
COPY run.sh .
RUN chmod +x run.sh
RUN chown vlcuser:vlcuser run.sh

# Switch to the VLC user
USER vlcuser

# Run
CMD ["./run.sh"]
```

### Setting up `web`

In the `web/` directory we shall do the following. Let's first create `nginx.conf`:

```json
server
{
        # For normal serving
        listen  [::]:80;

        # For SSL serving
        # listen  [::]:443 ssl;

        server_name  localhost;

        # SSL certificate configuration
        # ssl_certificate /etc/letsencrypt/live/vinyl.services.deavmi.assigned.network/fullchain.pem;
        # ssl_certificate_key /etc/letsencrypt/live/vinyl.services.deavmi.assigned.network/privkey.pem;

        # All accesses should be thrown away
        access_log /dev/stdout  main;

        # Only log errors
        error_log /dev/stdout warn; # TODO: Set back to error

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;

        location = /50x.html
        {
                root   /usr/share/nginx/html;
        }

        # Proxy pass alles
        location ^~ /
        {
                proxy_pass http://streamer:8000/;
                proxy_http_version 1.1;
                proxy_set_header Connection "upgrade";
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_set_header X-Forwarded-Proto $scheme;

                # by default nginx times out connections in one minute
                proxy_read_timeout 1d;
        }
}
```

Some notable things:

1. We have a `proxy_pass` which passes all traffic on `/` to the `streamer` container at port `8000`
2. We listen on **all** IPv6 addresses so we are reachable for access via the docker-proxy on the host-side

The `Dockerfile` is rather straightforward:

```Dockerfile
# Extend the base nginx image
FROM nginx:latest

# Delete default configuration and
# include ours
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/nginx.conf

# Healthcheck
HEALTHCHECK CMD curl http://[::1]:80
```

> Note, we use the `nginx:latest` base image/tag and then remove the old `default.conf` configuration and place ours into it. We also add a healthcheck to ensure it is reachable.

Lastly, the `compose.yml` additions:

```yaml
  web:
    build:
      context: web/
    container_name: web
    depends_on:
      # This is done because the `streamer` hostname must
      # be available to NGINX or else it exits on startup
      # because it is unresolvable
      - streamer
    restart: unless-stopped
    networks:
      - mainNet
    ports:
      - "80:80"
```

Pretty self explanatory.

1. However one thing is to note, we have a `depends_on` set to `streamer` because NGINX will fail (although it *will restart* so this isn't really needed) to start if it cannot resolve the hostnames in the `proxy_pass` definition. Since `streamer` is the container's (of the same name) hostname we want it to be up (and resolvable) prior to NGINX starting.

# End result

## Photos

Some photos.

![2024-09-25-21-33-04.jpeg](2024-09-25-21-33-04.jpeg)

![2024-09-25-21-33-30.jpeg](2024-09-25-21-33-30.jpeg)

![2024-09-25-21-33-45.jpeg](2024-09-25-21-33-45.jpeg)

![2024-09-25-21-33-56.jpeg](2024-09-25-21-33-56.jpeg)

![2024-09-25-21-34-04.jpeg](2024-09-25-21-34-04.jpeg)

![2024-09-25-21-34-10.jpeg](2024-09-25-21-34-10.jpeg)

The most important one was to use a powered USB hub. The capture card itself _can_ run off of the power that the Raspberry Pi supplies but from my testing it is rather unstable and will stop working after sometime. You will be visited with many "USB reset" messages in `sudo dmesg -w`. So definitely make use of a powered hub.