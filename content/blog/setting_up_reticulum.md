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