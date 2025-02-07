---
title: Setting up Reticulum
author: Tristan B. Velloza Kildaire
date: 2025-02-07
draft: true
---

# System setup

This section is information pertaining to the setup that I used.

## Your _Linux_ host

![2024-11-02-14-51-17.jpeg](../assets/2024-11-02-14-51-17.jpeg)

I used a mix of Raspberry Pis - old and new. The above is a photo of one of them.

I am sure that 2/3 required `rnspure` (as you will see later) and only one could use the normal `rns` - due to architecture things (32-bit versus a 64-bit dependency required).

## Packages

### System updates

Make sure to update your system as well with:

```bash
sudo apt update
sudo apt upgrade -y
```

### Docker-related

Firstly let's install the docker daemon *and* the version 2 of the Docker Compose utility (which acts as a plugin to the `docker` command as compared to seperate program as with the older/v1 `docker-compose` package). We also install the `sen` command-line based container management tool:

```bash
sudo apt install docker.io docker-compose-v2 sen -y
```

>**Note**: The `sen` command doesn't work on newer Ubuntu installations it seems.

### Tooling

Now, call me old fashioned but I like the older networking tooling, therefore in order to get the `ifconfig` command I will have to install the `net-tools` package:

```bash
sudo apt install net-tools -y
```

## Docker daemon

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

The `ipv6` flag is self-explanatory - enable IPv6 on the `default` bridge network. I have used the subnet `fd10:c8c6:3b63::/48` as (you shall see later) we will use masquerading. This means every container using this network will get an IPv6 address assigned that lies within this subnet.

#### The `ip6tables` and `experimental`

The `ip6tables` aids with port mapping when doing, what I suppose, is the masquerading/NAT'ing for container-out traffic (**NOT** in - we do this via a TCP proxy). This feature, however, is only usable when `experimental` flag is set to `true`

---

After this has been completed then we must restart Docker with:

```bash
sudo systemctl restart docker
```

Make sure all is fine by running and checking the status:

```bash
sudo systemctl status docker
```

## Compose file

Below is the `compose.yml` file:

```yaml
networks:
  retNet:
    driver: bridge
    name: retNet
    enable_ipv6: true
    ipam:
      config:
        - subnet: fde4:492a:fc7f::/48
        
services:
  lxmd:
  	container_name: lxmd
    restart: unless-stopped
    depends_on:
      - reticulum
    container_name: lxmd
    build:
      context: lxmd/
    volumes:
      # Configuration and storage for lxmd
      - /home/deavmi/volumes/lxmd/:/data
      
      # Makes timezone and time information available
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    networks:
      - retNet

  reticulum:
    container_name: reticulum
    restart: unless-stopped
    image: reticulum_base
    build:
      context: reticulum/
      args:
        - USER_UID=1000
        - USER_GID=1003

        # This platform cannot use
        # the crypto library, hence
        # we must use the `rnspure`
        # package
        - PURE_INSTALL=true
    volumes:
      # Reticulum configuration and storage directory
      - /home/deavmi/volumes/reticulum/:/data
      
      # Makes timezone and time information available
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    devices:
      # Add serial devices here
      - /dev/ttyUSB0:/dev/serial0:rwm
      - /dev/ttyUSB1:/dev/serial1:rwm
      # Add RNode devices here
    networks:
      - retNet
    ports:
      - :::4242:4242/tcp # TCP server interface port
      - :::4242:4242/udp # UDP server interface port
```

### Bind mounts

You will want to create the following two directories in your home directory (mine is `/home/deavmi` as you can see):

1. `~/volumes/reticulum`
    a. This will hold the configuration file for the `rnsd` daemon at `~/volumes/reticulum/config`. It it doesn't exist, don't worry - it will be created for you on startup and you can then edit it from there.
    b. It will also hold daemon storage information
    c. This is for our _main Reticulum instance_
2. `~/volumes/lxmd`
    a. This will hold the configuration file and storage information for the `lxmd` process
    b. This will have a configuration file available at `~/volumes/lxmd/config` which will be created if it doesn't exist upon startup.
    c. It will hold the stored messages our propagation node receives.

Both have the timezone data of the host mounted in along with the localtime; this ensures the container's system time is correct.

### Networks

As you can see I created one IPv6 network called `retNet`. This is both for allowing outbound NAT connections to be made.

Also, and this doesn't have anything to do with the address range assigned to the network, by placing the two on the same network they will both have link-local addresses and can hence discover each other (the two RNS daemons - the one in the `reticulum` container and the one in the `lxmd`container) over link-local multicast and automatically peer with each other. This is how our LXMD propagation node will be able to announce itself to the RNS network and furthermore announce its destination (this destination would be the one people use to sync send-receive messages to)

### Devices

For the `reticulum` container I have specified two entries:

```yaml
devices:
	# Add serial devices here
    - /dev/ttyUSB0:/dev/serial0:rwm
    - /dev/ttyUSB1:/dev/serial1:rwm
    
    # Add RNode devices here
    - /dev/ttyACM0:/dev/rnode1:rwm
```

This is so that I can forward the two serial adaptors I have connected to my host device into the container-side and make them available at the names specified. The `rwm` means that it should be _readable_ and _writable_ from the container-side. The `m` means _mknod'able_ which I don't know what that is.

### Ports

The `reticulum` container will be listening on ports `4242` for the TCP and UDP protocols.

This will be how people peer with our node. This makes sense for people wanting to make use of our node if we publish the peering details online and advertise it as a transport node.

### Image build

In order to build the base image that *both* our `reticulum` and `lxmd` containers will use we must pass in some build arguments to it first:

```yaml
build:
	context: reticulum/
	args:
		- USER_UID=1000
		- USER_GID=1003

    # This platform cannot use
    # the crypto library, hence
    # we must use the `rnspure`
    # package
    - PURE_INSTALL=true

    # If IPv6 should be preferred
    # whenever getaddrinfo() is
    # used
    - GAI_PREFER_IPV6=true
```

#### The `PURE_INSTALL`

If you are on a 64-bit ARM device then you can remove the `PURE_INSTALL` entry _or_ just set it to `false`. This is required because on 32-bit ARM devices Python will try to build one of the dependencies (the `cryptograpy` library) from scratch and this always seems to fail. Therefore this enables the built-in cryptography.

Security note from the Reticulum developer:

>Reticulum also includes a complete implementation of all necessary primitives in pure Python. If OpenSSL & PyCA are not available on the system when Reticulum is started, Reticulum will instead use the internal pure-python primitives. A trivial consequence of this is performance, with the OpenSSL backend being much faster. The most important consequence however, is the potential loss of security by using primitives that has not seen the same amount of scrutiny, testing and review as those from OpenSSL.

>If you want to use the internal pure-python primitives, it is highly advisable that you have a good understanding of the risks that this pose, and make an informed decision on whether those risks are acceptable to you.

>Reticulum is relatively young software, and should be considered as such. While it has been built with cryptography best-practices very foremost in mind, it has not been externally security audited, and there could very well be privacy or security breaking bugs. If you want to help out, or help sponsor an audit, please do get in touch.

#### The `GAI_PREFER_IPV6`

Your Docker container will **always** have IPv4 connectivity - one cannot disable that. In my case however I have configured IPv6 connectivity but that means that the container will have both IPv4 and IPv6 connectivity (dualstack).

* Now on systems with **global** (that's the keyword) IPv6 addresses `getaddrinfo()` will place addresses resolved via DNS that are part of the `AF_INET6` (Ipv6 addresses) family _first_ before any `AF_INET` (IPv4 addresses) ones.
* However, if you have an IPv4 address assigned to your interface but also an IPv6 one **but** the IPv6 one is from the ULA space (as most containers are) then you will actually have `AF_INET` records returned first.
    * See this brilliant blog: https://thomas-leister.de/en/lxd-prefer-ipv6-outgoing/
* Now this is annoying as I have perfectly working IPv6 (via masquerading NAT when it reaches the host side of my Docker host) and I would like it to make use of that which in my case would mean doing so _even if_ I am using ULA addresses on my container.

The way to avoid this behavior is to edit `/etc/gai.conf` and basically set the priority of records. The way this works is by associating certain address masks with a given priority. When the resolver gets a bunch of records for a given domain name it will then apply each mask to them, the longest prefix match will be chosen and then the associated priority value will accompany that specific record. These priorities are then used to order the results returned by `getaddrinfo()`.

The easiest fix is to do the following, create a `label ::/0 0`. This will match ANY IPv6 address and give it the highest priority. This should stop the following:

![image.png](../assets/image_1738337867615_0.png){:height 346, :width 629}

I wanted to test Python's `getaddrinfo()` and now IPv6 is preferred and _only then_ IPv4:

![image.png](../assets/image_1738338019623_0.png){:height 123, :width 629}

Also when running now I am able to connect to the hosts:

![image.png](../assets/image_1738338774254_0.png)

##### An aside

You may have noticed something - _"where are the preferences for IPv4?"_. Well, what this program does (and many other do) is pack any IPv4 addresses (from A records) into the lowest 32-bits of an IPv6 address. Therefore a common IPv4 address will be $96$ zeroes followed by $32$ bits containing the IPv4 address. This then explains the prefixes for `/96` that you will see, specifically those with $96$ $0$ bits. There is also the one with $64$ $0$ bits followed by $0xfff$ (the next $32$ bits) and _then_ $32$ $0$ bits - I believe these may be for Teredo tunnelled IP address or something - a kind-of IPv4 use case.

## Dockerfile

### For `reticulum_base`

The `reticulum_base` image's `Dockerfile` is as follows:

```Dockerfile
# Base image
FROM debian:bookworm AS build
MAINTAINER "Tristan Brice Velloza Kildaire" "deavmi@redxen.eu"

# Don't allow interactive prompts when using apt
ARG DEBIAN_FRONTEND=noninteractive

# Upgrade the system's dependencies
RUN apt update
RUN apt upgrade -y

# Activate arguments provided as build parameters
ARG USER_UID
ARG USER_GID

# Install Python3 and requirements
RUN apt install python3 python3-pip -y

# Create the reticulum user
# and group
RUN groupadd reticulum -g $USER_GID
RUN useradd -m reticulum -u $USER_UID -g $USER_GID

# Add us to the group which the
# serial device files are grouped
# under (permission-wise)
RUN usermod reticulum -aG dialout

# Install ncat (useful tool)
RUN apt install ncat -y
RUN apt install picocom -y
RUN apt install curl wget -y
RUN apt install net-tools dnsutils iputils-ping -y

# Activate build argument `GAI_PREFER_IPV6`
ARG GAI_PREFER_IPV6

# Run GAI configuration
COPY gai.sh /tmp
RUN chmod +x /tmp/gai.sh
RUN /tmp/gai.sh
RUN rm /tmp/gai.sh

# Activate build argument `PURE_INSTALL`
ARG PURE_INSTALL

# Copy across installation
# script
COPY install.sh /tmp
RUN chmod +x /tmp/install.sh
RUN chown reticulum:reticulum /tmp/install.sh

# Switch to reticulum user
USER reticulum
WORKDIR /home/reticulum

# Add the directory where pip will
# install binaries to to our PATH
ENV PATH=$PATH:/home/reticulum/.local/bin

# Install reticulum and delete
# the install script
RUN /tmp/./install.sh
RUN rm /tmp/install.sh

# Run the daemon
CMD sleep 5 && rnsd --config /data
```

* The reason for the `sleep 5` is because there seems to be some bug in the `AutoInterface` (you'll see later what this is) handling that causes a crash due to what I _assume_ to be the fact that the container's internal networking has not yet been setup.
    * It's a hard thing to debug because as soon as the container exists (due to the `CMD` program exiting) all I get is a non-zero exit code (sometimes even $0$ exit code) but no logging whatsoever as to indicate why.

The other needed files, `install.sh`, contains the following:

```bash
#!/bin/sh

if [ ! "$PURE_INSTALL" = "true" ]
then
        PURE_INSTALL="false"
fi

echo "Using pure RNS: $PURE_INSTALL"

if [ "$PURE_INSTALL" = "true" ]
then
        pip3 install rnspure --break-system-packages
        pip3 install pyserial --break-system-packages
else
        pip3 install rns --break-system-packages
fi
```

And `gai.sh`:

```bash
#!/bin/sh

if [ ! "$GAI_PREFER_IPV6" = "true" ]
then
        GAI_PREFER_IPV6="false"
fi

echo "IPv6 is preferred when using getaddrinof()?: $GAI_PREFER_IPV6"

if [ "$GAI_PREFER_IPV6" = "true" ]
then
        echo "label ::/0 0" >> /etc/gai.conf
fi
```

### For `lxmd`

The `lxmd` image's `Dockerfile` is as follows:

```Dockerfile
FROM reticulum_base

# Install lxmf
RUN pip3 install lxmf --break-system-packages --no-dependencies

# Run the lxmd daemon
sleep 5 && lxmd --config /data -p
```

* As you can see we re-use the layer created earlier named `reticulum_base`, therefore we need not specify the same dependency installation commands or user creation. We only need to add the additional commands to install what we need and then declare what is to be run on container start-up.
* The `-p` argument tells the daemon to announce a propagation node destination so that clients can sync messages to/from it and so that other propagation nodes can do the same as well.
* The `-vvvv` can be omitted once you are done your initial testing; it just increases the logging verbosity.
* The `CMD` is interesting contains a call to `sleep` for the same reasons as mentioned earlier.

# Logging

There are some things we can do to ensure we can see what the Reticulum daemon is busy doing, even if temporarily.

```toml
[logging]
	loglevel = 7
```

Using a `loglevel` of 7 is the highest verbosity level available, the lowest is that of 0 which logs only critical information. Recommended level is 4 once you're done debugging things with 7.

# Identities

An instance of Reticulum combines the routing engine, a set of interfaces and then an *identity*. This is your unique handle on the network. It is derived via asymmetric encryption and hence is derived from your public key part of your $(key_{pub}, key_{priv})$.

## Destination

Destinations are endpoints that can have data sent to them. This is effectively how people run services such as LXMF-based ones, by establishing an endpoint on the network you can send data to. To make a destination known to the network, and so that routing information can be learnt throughout the network, an announcement of the destination id must be made.

Destinations are derived from your identity. This would make sense. I say *derived* because I mean they have to be unique to your presumably unique *identity*. It's all fine and well you can sign a message with your key saying _"Hey I own dest $X$"_ but if someone can also veritably announce that (signature and all) then it would confuse routing. Hence destinations have to be unique, and if your identity is pretty unique then deriving destinations therefrom can be too.

## Example

On $node_1$ I have generated an identity and then derived _two_ destinations from it. I then go ahead and announce these over all my interfaces:

![image.png](../assets/image_1733400115307_0.png)

On $node_2$ it receives the _two_ destination announcements and we can see it shows from which _identity_ those destinations were derived from. We can also see which interface the announcement packet was received from:

![image.png](../assets/image_1733400213127_0.png)

# Interfaces

The interfaces section is defined with the following declaration:

```toml
[interfaces]
	# Interface definitions go here
```

## Adding serial interfaces

One of the things I have wanted to play around with since learning about Reticulum is that of its vast interface support it touts.

Firstly plug in a serial adaptor like one of these two (I purchased these online):

![2024-11-13-08-26-07.jpeg](../assets/2024-11-13-08-26-07.jpeg)
![2024-11-13-08-26-14.jpeg](../assets/2024-11-13-08-26-14.jpeg){:height 621, :width 689}

If you type in `sudo dmesg -w` before plugging them in (you can unplug it and pug it in again if not) then you will be able to grab the mountpoint that the device driver is exposed at:

![image.png](../assets/image_1731425130893_0.png)

>**Note**: In this case the device is available at `/dev/ttyUSB0` on my host. Remember this as it will be important.

In order to connect between the two you need what is effectively the equivalent of a crossover cable so that the TX lines connect to the RX lines of the other side and vice-versa. We call this a _"null modem"_ cable and it looks as follows:

![2024-11-21-08-47-47.jpeg](../assets/2024-11-21-08-47-47.jpeg)

We then connect either sides as follows:

![2024-11-21-08-48-23.jpeg](../assets/2024-11-21-08-48-23.jpeg)
![2024-11-21-08-48-34.jpeg](../assets/2024-11-21-08-48-34.jpeg)

And then it appears as follows:

![2024-11-21-08-53-29.jpeg](../assets/2024-11-21-08-53-29.jpeg)

### Interface configuration

```toml
[[Your interface name]]
	type = SerialInterface
    enabled = yes
	port = /dev/serial0
    
    speed = 115200
  	databits = 8
  	parity = none
  	stopbits = 1
```

Some noteworthy parameters are:

1. `port`
    a. I set it to `/dev/serial0` as that is the name of the serial device that will be present on the container-side.
2. Serial specific
    a. Serial connectors let you configure how fast they are to send data buffered to their chip, and also how to demarcate certain parts of data streams upon other things including checksumming of data.
    b. I went with the default that was on the [documentation for serial interfaces](https://reticulum.network/manual/interfaces.html#serial-interface).

### Testing the Docker device forwarding

The reason to test that the communication works is twofold:

1. I wanted to test that the device forwarding done from the host's serial device at `/dev/ttyUSB0` to the container's side at `/dev/serial0` worked in terms of permissions I had set up (ala `dialout` group mod)
2. Test the null modem actually works.

This is very easy, firstly execute `/bin/bash` in our container so we can get a shell with (on both machines):

```bash
sudo docker exec -ti reticulum /bin/bash
```

After this run this (on both machines):

```bash
picocom /dev/serial0
```

Then start typing on one, and it should appear on the other and vice-versa:

![image.png](../assets/image_1732189883387_0.png){:height 354, :width 659}
![image.png](../assets/image_1732189897833_0.png)

## Adding `AutoInterface`

The `AutoInterface` is an interface that can be run over any interface that supports link-local IPv6 multicast. The use case of this is that it allows for easy peering with any _other_ Reticulum router that's on the same link; meaning that you won't have to explicitly state that you want to connect to a given LAN peer.

Configuration is as simple as the following:

```toml
[[LAN Multicast interface]]
	type = AutoInterface
    enabled = yes
```

>**Note:** It can work on non-link-local multicast as well, however, this is just the default - for it to be `link`-local (see [this](https://reticulum.network/manual/interfaces.html#auto-interface))

## Adding IP interfaces

Not everything I do will be peering _directly_ over physical interfaces such as that via serial or via the RNode interface. I will also want to join the larger Reticulum test network. This can be accomplished via adding some IP-based interfaces.

### Inbound

Inbound interfaces refer to those that will bind to an $(address, port)$ pair and listen for incoming connections.

Let's declare a TCP-based listener on port `4242`:

```toml
[[TCP Inbound server]]
    type = TCPServerInterface
    enabled = yes
    listen_ip = ::
    listen_port = 4242
```

Let's _also_ declare a UDP-based listener on port `4242`:

```toml
[[UDP Inbound server]]
    type = UDPInterface
    enabled = yes
    listen_ip = ::
    listen_port = 4242
```

>**Note**: For some reason this failed to bind for me

### Outbound

Outbound interfaces refer to those that will actively open up a connection to a remote host.

Below are some commonly used peers which are hosted by some people out there:

```toml
[[TCP Outbound 1]]
	type = TCPClientInterface
	enabled = yes
    target_host = reticulum.betweentheborders.com
    target_port = 4242

[[TCP Outbound 2]]
    type = TCPClientInterface
    enabled = yes
    target_host = amsterdam.connect.reticulum.network
    target_port = 4965

[[TCP Outbound 3]]
    type = TCPClientInterface
    enabled = yes
    target_host = aspark.uber.space
    target_port = 4965
```

You can find more [here](https://reticulum.network/connect).

## Adding RNode interfaces

I have written on the topic of #[[Building my first RNode - Part 1]] already, so that is a good place to start if you want to purchase some hardware and get it up and running as an RNode. RNode's provide many features, one of which is acting as a LoRa-based interface for the Reticulum daemon.

### Setting up the device

![2024-12-06-21-10-51.jpeg](../assets/2024-12-06-21-10-51.jpeg)

Firstly open up a terminal and then run `dmesg -w` (you may need to run it as `root` in order for it to work). Then go ahead and plug in your device, once it is in you should see some text:

![image.png](../assets/image_1733510926696_0.png){:height 521, :width 659}

>**Note**: The circled text shows the name of our device, here it would be available at `/dev/ttyACM1`.

Once booted it should eventually change to this screen:

![2024-12-06-21-27-43.jpeg](../assets/2024-12-06-21-27-43.jpeg){:height 887, :width 659}

### Configuring RNS

We now need to make our Reticulum daemon aware of the RNode that's now available at the `/dev/rnode1` device node.

We can do this by adding a new interface entry:

```toml
[[RNode test]]
  type = RNodeInterface
  enabled = yes
  port = /dev/rnode1

  frequency = 868000000
  bandwidth = 62500
  spreadingfactor = 8 # Doesn't need to match on other side
  codingrate = 6 # Doesn't need to match on other side
```

1. The `type` must be set to `RNodeInterface` which let's Reticulum know it is an RNode and not just a dumb serial device
2. Specify the `port` to be that of `/dev/rnode1` (or whatever your device node's path is)
3. The radio options we set here must be the same on all of the other LoRa-based RNode's you wish to be able to communicate with:
    a. `frequency`
        i. Expressed in hertz, it makes sense why this is required to be the same
    b. `bandwidth`
        i. Ask some HAM radio man
4. These other options are not required to be the same:
    a. `spreadingfactor`
        i. Ask some HAM radio man
    b. `codingrate`
        i. Sounds oddly like a symbol rate, but once again - ask a HAM radio man

### Seeing daemon

When starting up the RNS daemon with `rnsd -vvv` we shall see something like this:

![image.png](../assets/image_1737285967663_0.png){:height 166, :width 659}

I sent an announcement on my phone using Sideband (and of which had an RNode attached over Bluetooth). Here we can see the announcement from Sideband's LXMF destination received over on the other machine running `rnsd -vvvv`:

![image.png](../assets/image_1737286078239_0.png){:height 32, :width 659}

### Seeing it work

What it looks like when it is running:

![2024-12-08-11-15-43.jpeg](../assets/2024-12-08-11-15-43.jpeg)

The green light shows when there is transmission or reception of data over LoRa

![2024-12-08-11-18-08.jpeg](../assets/2024-12-08-11-18-08.jpeg){:height 1233, :width 689}

# Routing

Now that we have setup the interfaces we wish to make available to the network stack, we can tweak some routing settings.

## Forwarding

This section is configured in the `[reticulum]` section.

The first option to consider that is that of _packet forwarding_. This refers to accepting packets that are not destined to the receiving node _itself_ but of some other node that _this node_ may know how to forward. Whether or not we make such a forwarding decision is something one can control with the `enable_transport` option. Setting it to `True` will enable forwarding; to `False` will disable it.

# Network stack access

This section is configured in the `[reticulum]` section. Other than routing Reticulum traffic on behalf of others, you may want to actually also _use your node_ for accessing the various services that others run on the network.

### Shared instance

There is a configuration option called `shared_instance_port`. The way that Reticulum-supported applications work is the following:

1. **If** the `shared_instance_port` is bound (by some _other_ Reticulum instance) to then the application will connect to $(localhost, sip)$ where $sip$ is the `shared_instance_port`.
    a. Furthermore it will then communicate Reticulum packets to/from this connections
2. **If** `shared_instance_port` is _not_ bound then the program will start a new Reticulum _full instance_ (a router) and bind to that port. Therefore allowing another program like described above to use it as well.

Both scenarios require reading the `~/.reticulum/config` file to determine the shared instance port to check for (or bind to). This means only one router is ever spawned but the spawner and everyone else can make use of it. `shared_instance_port` is only ever bound to $localhost$.

>**Note**: To enable this ensure that `share_instance` is set to `Yes`.