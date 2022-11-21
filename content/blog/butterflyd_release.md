---
title: butterflyd v0.0.36 (and how to use it)
author: Tristan B. Kildaire
date: 2021-01-25
---

I'd like to take this time to introduce you to the first pre-release of the butterfly mail server. It implements mailbox management such as creating folders, listing folders, listing mail within folders and deleting mail and deleting folders. Fetching mail is implemented to and so is sending mail (both to local and remote server (the latter implies remote mail delivery and reception are supported too)).

You can download the latest release from the butterflyd homepage and you should make sure to download the JSON file too which comes with it. You can see what it looks like below with explanations too. This will allow you to easily configure your server and after reading it you can place it in the same directory as your executable.

---

In order to configure Butterfly you will want to create a file named `butterflyd.json` in the same directory as your
`butterflyd` executable. The contents should look something like this:

```json
{
    "listeners" : {
        "enabled" : ["listener1"],
        "listener1" : {
            "type" : "ipv4",
            "domain" : "10.1.0.4:6969",
            "address" : "0.0.0.0",
            "port" : "6969"
        },
        "listener2" : {
            "type" : "ipv6",
            "domain" : "10.0.0.9:2222",
            "address" : "::",
            "port" : "6969"
        }
    }
}
```

The first section if the `"listeners"` section. This key contains an array, `"enabled"`, which contains a list of strings which name which listeners, specified in the same JSON object, should be enabled when the daemon starts up. Listeners are basically TCP port associations with a few other settings that say the server should listen on this port and this IP address. If you take a look at the first listener (which also happens to be the only enabled one), `"listener1"`, you will see that is contains a `"type"` field. This specifies whether or not the listener is for IPv4 or IPv6. The `"port"` and `"addresses"` fields specify the port and address to bind to.

Lastly the domain should be the publicly facing address/domain and port pair for your mail server. It is used when generating `from` field in outgoing mail as this isn't done on the client side but rather the server side. You want this to be a reachable address and port pairing as replying to such an email received from such a domain should be possible.