---
title: "Eventy v0.4.x series released"
author: Tristan B. Velloza Kildaire
date: 2022-11-28
---

{{<bruh>}}
<img src="/projects/eventy/logo.png" width=50% height=50% style="float:right;gap;margin-left:20px">
{{</bruh>}}

This is a rather quick fix for Eventy that I have made with the `v0.4.3` release. It should improve performance and technically lower the load average. This is due to the removal of the event-loop which would periodically (configurable or using `yield`)
check if there were any enqued events and then dispatch them. We now dispatch events on the call to `push(Event)`. Hence no spinning loop is needed which increases the time that thread is present in the run-queue hence increasing the
load-average, secondly the dispatching would technicqally take longer as no signalling occured - one would just wait for the kernel to schedule the event-loop whenever it decided to (in our case of using `yield()`).

You can take a look at the [release notes](/projects/eventy/releases/v0.4.3/), or alternatively see the [source code](https://github.com/deavmi/eventy) and [DUB repository](https://code.dlang.org/packages/eventy)