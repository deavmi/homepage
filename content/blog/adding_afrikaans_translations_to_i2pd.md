---
title: Adding Afrikaans translations to i2pd
author: Tristan B. Kildaire
date: 2021-05-25
---



{{<bruh>}}
<img src="https://i2pd.website/images/favicon.png" style="float:right">
{{</bruh>}}

## What is i2p?

I will try do it justice but there is a wikipedia page for it [here](https://en.wikipedia.org/wiki/I2P). Anyways, here goes nothing. I2P (or the Invisible Internet Project) is a project
that aims to provide people a very secure and especially an anonymous network whereby they can run Internet services on (port-based protocols like TCP or
UDP) in a way that doesn't deanonymize the user accessing it or the person hosting it.

## Translations to die taal?

The developer of [i2pd](https://i2pd.website/) (the C++ version of i2p (which is way nice because imagine using Java - ew)), `orignal`, asked me today whether or not there are differences
between Dutch and Afrikaans and the conversation went something like this:

```
[21:21:24] <orignal> deavmi is there a big difference between afrikaan and dutch?
[21:21:37] <orignal> we are working on i2pd localization
[21:25:41] <deavmi> orignal: Not too much
[21:25:48] <deavmi> They put j's at the end of every word
[21:25:53] <deavmi> and vaocabulary differes here and there
[21:25:59] <deavmi> Buit you would understand them for the most part
[21:26:10] <deavmi> But iw ould still say a big enough difference to have two seperate ones
[21:26:23] <deavmi> But an afrikaaner would understand a dutch dude and vice versa
[21:26:48] <deavmi> orignal: I can help with AFrikaans localization
[21:26:51] <orignal> can a transaltion to afrikaan be claimed as dutch?
[21:26:53] <deavmi> just send me a link to it
[21:26:57] <deavmi> orignal: no
[21:27:00] <orignal> yes, that's would be nice
[21:27:01] <deavmi> I would not say so
[21:27:02] <deavmi> Sure
[21:27:07] <orignal> sec
[21:27:42] <orignal> https://github.com/PurpleI2P/i2pd/blob/openssl/i18n/Russian.cpp
[21:27:45] <+botty> PurpleI2P/i2pd
[21:28:02] <orignal> we need similar files for other laguages
[21:28:08] <deavmi> FOr AFrikaans do I change the secondayr mapping then
[21:28:13] <deavmi> In a file like AFrikaans.cpp
[21:28:16] <deavmi> I think I know what do do then
[21:28:39] <orignal> yes, Afrikaan.cpp is fine
[21:29:05] <orignal> just replace Russian phase to afrikaan
[21:29:10] <deavmi> Sure
[21:29:11] <deavmi> thanks
[21:29:12] <deavmi> DOing so now
[21:29:18] <deavmi> I will make a PR near end of week
[21:29:20] <deavmi> Will do some tonight
[21:29:36] <orignal> yes, PR is fine
[21:29:41] <orignal> no rush
[21:30:06] <orignal> you know the second translation is going to be ... turkmen )))
[21:33:24] <deavmi> lol
[21:33:25] <deavmi> hehe
[21:33:25] <deavmi> :)
[21:33:37] <deavmi> I will maybe run my translations past an afrikaans friend of mine when done, they're alright so far
[21:34:41] <deavmi> https://github.com/deavmi/i2pd/commit/f8af84b2220e35c4ced07659eac9c2468792788d
[21:34:42] <deavmi> So far
[21:34:43] <deavmi> :)
[21:34:53] <+botty> Initial translations added · deavmi/i2pd@f8af84b
[21:35:51] <orignal> the thing it requires some techincal background like yours
[21:35:57] <deavmi> Yes
[21:35:59] <deavmi> Understanable
[21:36:01] <deavmi> Perfect match then
[21:36:01] <deavmi> :)
```

Next thing I know, I'm writing translations for English string mappings to Afrikaans ones. The mechanism `orignal` setup is quite nice I must say. The format
is standard per each language and all you do is provide your translation as a mapping to the English one. You copy-and-paste this several times in different files
such as `Russian.cpp` and `Afrikaans.cpp` for the respective files and there you have it - internalization!

## It's _that_ easy to contribute translations to i2pd!

Literally, from the file [`Afrikaans.cpp`](https://github.com/deavmi/i2pd/commit/f8af84b2220e35c4ced07659eac9c2468792788d) this is what it looks like:

```cpp
{"already in router's addressbook", "alreeds in die router's addressboekie"},
{"Click", "Druk"},
{"here", "hier"},
```

(Code above is licensed under the BSD 3-clause license)

That's very easy to do!