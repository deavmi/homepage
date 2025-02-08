---
title: Offering Reticulum routing and an LXMF propagation node
author: Tristan B. Velloza Kildaire
date: 2025-02-08
---

Hi all, I recently setup some Reticulum stuff.

## Transport node

I am running a _transport node_ which means I can forward Reticulum network
packets for between the _other_ nodes I am connected to over my interfaces.

### Interfaces

Speaking of interfaces I have made available my transport node to be peered
with dirctly (if you want to have me as your potential first hop) over
several networks:

```toml
[[rothbard_RNS_transport_ZA]]
    type = TCPClientInterface
    enabled = true
    target_host = rothbard.lab.networks.deavmi.assigned.network
    target_port = 4242

[[rothbard_RNS_transport_ZA_ygg]]
    type = TCPClientInterface
    enabled = true
    target_host = 200:73eb:2e4:14be:aac7:90b3:784b:71a3
    target_port = 4242

[[rothbard_RNS_transport_ZA_i2p]]
    type = I2PInterface
    enabled = true
    peers = guuahj7pyb6ksmjv2bqrjg4cs2wou6cor3ivsi6crntqbzsxnbna.b32.i2p
```

## LXMF propagation node

I am running a propagation node at the destination address of `b4ac9705550f189cf2aff4d4748bd05e`.
You can use it as your propagation node if you want, it has a nice
big message store configured and I will probably aim at increasing
the size in the months to come.