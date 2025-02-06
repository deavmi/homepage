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

