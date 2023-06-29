---
title: 🪵️ Birchwood - A sane IRC framework for D
author: Tristan B. V. Kildaire
date: 2023-06-29
draft: false
---

{{<bruh>}}
<img src="/img/birchwood.png" width=25% height=25% style="float:right;gap;margin-left:20px">
{{</bruh>}}

# Why IRC though?

This is a legitimate question. We live in the era of many programmers using both open-source and proprietary solutions such as Matrix and Discord respectively as a form of communications between developers and bots. Seemingly these services do offer many more features than one of the very first internet communications technologies - IRC - does.

I, however, do not really care too much for the features these platforms offer when it comes to the basics of communications. I immediately disregard services like Slack or Discord due to their proprietary nature. As for services such as Matrix, I do actually make quite extensive use of it now and then so I have nothing bad to say about the user experience with such a service and I think the additions such as end-to-end encryption are very cool however IRC and Matrix do have some differences at a technical level. Matrix is federated whilst IRC networks form a spanning tree rather than necessarily direct server-to-server links (or a _"full mesh"_).

I think my main reason for seeing IRC _still_ as a viable alternative to Matrix even with the technicalities of their respective protocols aside is really just because IRC is much more simpler and _Just works ™️_. It is therefore because of this reason that I cared enough to venture into an IRC-related project.

---

# What is `birchwood`?

Birchwood is an IRC framework for DLang, but what is a _framework_. To put it simply it allows one to make use of the IRC protocol via an IRC server but in a programmatic way.

For example, if you wanted to process any message that was sent to the channel `#general`, maybe add some text to it and the send that message back to the same channel then we could do something like the following:

```d
public override void onChannelMessage(Message fullMessage, string channel, string msgBody)
{
    // Only run when the message is from the `#general` channel
    if(cmp(channel, "#general") == 0)
    {
        // Quote the original message and send it to `#general`
        channelMessage("\""~msgBody~"\"", "#general");
    }
}
```

Some notable things here are the following:

1. The overriding of `onChannelMessage`
    * By overriding this handler we can capture any event which is triggered by the receiving of a message to any IRC channel
    * We are provided the message's channel as `channel` and the message itself as `msgBody`. Anything else can be grabbed from the `fullMessage` object of type `Message`
2. The usage of `channelMessage`
    * This simply allows one to send a message (`message`) to the given IRC channel (`channel`)
    * We decide to send back a quoted version of the received message in this example

From this one can get the idea of what an _"IRC framework"_ provides - effectively a means to program an IRC client in a custom manner which reacts to different events in ways _you_ can define.

## An extendable API

With Birchwood a fixed API is provided in terms of handlers and commands for messages and such but one can easily extend support for specific types of events by using an override for the `onGenericCommand` method and `command` respectively.

### Custom handlers

One can add custom event handlers by firstly overriding the `onGenericCommand()` (TODO: check) method as such:

```d
public override void onGenericCommand(Message message)
{
    ...
}
```

Then playing a check in the `...` on the contents of the actual message and calling a sub-routine from there which will handle that specific message type. This can be done in a manner as seen below (our command-specific handler in this case is `noticeHandler(string)`):

```d
// Handles NOTICE messages
public void noticeHandler(string message)
{
    ...
}

public override void onGenericCommand(Message message)
{
    // Get the type of command and payload
    string commandType = message.getCommand();
    string payload = message.getParams();

    // If the command is of type NOTICE
    // call our handler
    if(cmp(commandType, "NOTICE") == 0)
    {
        noticeHandler(payload);
    }
}
```

### Custom commands

Not only can handlers for generic commands be used but some more commonly used commands such as `/join` can be used with methods such as `joinChannel(string[] channels)` and `joinChannel(string channel)`, instead of having to do something like the following:

```d
Message joinCommand = new Message("", "JOIN", "#general");
client.command(joinCommand);
```

One can do the following:

```d
client.join("#general");
```

One can take a look at the API [here](https://birchwood.dpldocs.info/birchwood.html).

---

## Special thanks

I'd like to make a shoutout to [supremestdoggo](https://github.com/supremestdoggo) for adding text formatting support - it's been great and helped me build the [gitea-irc-bot](https://code.dlang.org/packages/gitea-irc-bot) with it!

---

## Start using it today!

Got a project where you want an IRC bot as part of an integration? That's as easy as adding it to your dub-based project with:

```bash
dub add birchwood
```

Remember to stay up-to-date, things move and good improvements come - I also try to never break the API - only improve behavior.