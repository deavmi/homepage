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

