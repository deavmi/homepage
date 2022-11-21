---
title: skoenlapper v0.3.2 released! (and how to use it)
author: Tristan B. Kildaire
date: 2021-01-25
---

I'd like to take this time to introduce you to the first pre-release of the skoenlapper butterfly mail client that has been released! It implements all the needed basic features you would need to test out the important parts of butterfly, this includes fetching folders, fetching mail, sending mail and reading mail. The other features yet to be added are management of the mailbox itself in terms of moving mail, deleting mail and moving and deleting folders.

You can download the latest release from the skoenlapper [homepage](/projects/skoenlapper) and you should make sure to download the JSON file too.

---

The usage is pretty simple as shown on the homepage but I will include it below:

```
skoenlapper v0.3.2

help                                                                            Shows this screen
new -t [address, address, ...] -s [subject] -c [config file path]               Send a new mail
view -m [mailPath, maiPath, ...] -c [config file path]                          View a mail message
register -u [username] -p [password] -s [address:port]                          Register on the server
daemon -c [configFile]                                                          Run a mail fetcher that cycles every 5 seconds
```

With this you will understand how one uses it from the command-line. When making new mail one can enter as many lines as they want in their TTY and until EOF is retrieved/signaled (when reading from the TTY via Ctrl+D on empty TTY buffer) - same applies to when you pipe something in but I assume most people will not be doing that and instead will by using skoenlapper in a way similar to the `mail` command.

You will also want to make sure you have a running butterfly mail server and also have set the parameters such as the server address and port pair in the skoenlapper's JSON configuration file. You will then want to register with the server (how to do that is shown above) and then you can start sending and receiving mail. The former done with the `new` command and the latter done with the `daemon` command (and by then reading the mail with the `view` command and pointing it to a mail message in the mailbox somewhere in the mailDirectory specified in the JSON file (like <mailDirectory>/Inbox).

Want to know how to run your own mail server though? Check out [this](/blog/butterflyd_release/) post.