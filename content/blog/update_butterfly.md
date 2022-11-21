---
title: Update on the Butterfly Project 
author: Tristan B. Kildaire
date: 2021-01-25
---

## A quick recap on _Project Butterfly_

In case you forgot Butterfly is my project that aims to create a more uniform and compact email protocol as compared to what we have today with the SMTP and IMAP/POP stack. It firstly wants to put both mailbox management and mail exchange into one protocol, something that was previously done separately by IMAP/POP and SMTP respectively. It also aims to be JSON-based which is quite neat as JSON is an awesome format and very fitting for this application. We also have inbound registration (meaning you can register with the mail server directly! - the client also currently supports this!).

## Where we are now

Now we have all the functioning components needed and working to be able to deliver the mail functionality (not so much full mailbox management) but let me be specific. We _**do have mailbox management**_ in the server implementation and supported by the library, libutterfly. We simply haven't added it into the de facto client, skoenlapper, yet. The server and client-side library and effectively done however. So now my main focus is just the client.

The server has inter-server mail transfer already implemented and mailbox management (creating folders, storing mail in certain folders, listing folders etc.). Likewise the client-side library has been written to mirror all commands available on the server-side. As for the client I have decided to go about creating a client that operates in a more older way that modern email clients. So the client.... _skoenlapper_ - let's talk about!

## Skoenlapper

The client skoenlapper takes the approach of being a simple client to communicate with the server all from the command-line in a script-like manner rather than a interactive manner (and no reading in from fd 0/standard input) until we reach an EOF signal is not what I consider interactive - interactive would be like having a free-form editor launch with ncurses-and-shit). So the sending of mail is similar to that of the `mail` command in UNIX. In terms of fetching mail, well that too is similar to the typical UNIX way. Normally you'd have a daemon connected to a mail server watching for new mail and it would dump it into a directory somewhere - skoenlapper is no different, you simply specify the directory to dump mail in and start it running in daemon mode - it will download mail every 5 seconds then (only overwriting mail not already downloaded). The last thing unimplemented (although I will probably finish it tonight) is a mail message viewer to view the JSON-encoded mail files.

![](/img/butterfly_update/todo_list.png)

{{<bruh>}}
<center>
My moleskin book with all my (at least I plan to) - great ideas and todo lists in it
</center>
{{</bruh>}}