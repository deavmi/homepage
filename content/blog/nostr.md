---
title: "IPv6 Nostr relays for the masses!"
author: Tristan B. Kildaire
date: 2025-03-12
draft: false
---

{{<bruh>}}
<img src="ossie.jpeg" style="float:right" width="179" height="266">
{{</bruh>}}

## What is Nostr?

If you're unsure what **Nostr** is then I'd suggest you [read this](). Suffice it to say its
a social networking protocol that:

1. Is easy to setup
2. Resillient
	* On the level of blowing your "posts" out like a dandelion
3. Doesn't use blockchain
	* Mentioning this because Nostr has a lot of Bitcoin users on it and this
	_might_ confuse people and make them think it has something to do with blockchain

## Relays!

I am running some relays for public usage, you can connect to them using the following
URIs:

1. Ephemeral relay
	* URI: `wss://nostr.services.deavmi.assigned.network:7777`
	* This relay deletes content every week but is open to anyone posting to it
2. IPv6 Homesteader relay
	* URI: `wss://nostr.services.deavmi.assigned.network:1920`
	* This relay is open to anyone, of course who has IPv6.
3. Personal relay
	* URI: `wss://nostr.services.deavmi.assigned.network:1488`
	* Only open to me, for creating events, but you may as well add this relay
	to your client (if it hasn't already done so automatically) in order to
	have another relay whereby you can receive my posts from

>**Note:** As a rule, do not post illegal content to any of the above relays
	
