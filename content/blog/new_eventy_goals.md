---
title: New goals for Eventy
author: Tristan B. V. Kildaire
date: 2021-10-23
draft: true
---

# The current state

The goals have shifted for Eventy. What started off as what I have now figured out is referred to _"signals and slots"_ which is effectively to say that you associate a _input signal_ like `push(Event)` where `e.id` maybe is equal to `8`, and you associate that
number `8` with a _slot_ that handles anything of _"type"_ `8`. Given 5 events being pushed into such a system we then end with `5+1` threads, the `+1` accounting for the caller main thread and the other 5 being the handlers that were dispatched for the respective slot(s). This is effcetively what the v.0.4.x series of Eventy does now.

Previosuly, the inefficient mechanism of an event-loop and queue was used but this wasn't a sleeping loop but rather a yielding loop. Which meant it would run rather often even when it didn't need to. What would have been better would be the use of `sleep()` (indeffinately)
and then signalling that process to wake it up. Some precautions, as I have now read, would probably need to be taken into account. Such would be checking the received signal and if it matches the one we expect - sometimes the kernel can send us signals like when a thread leader quits (or something I believe) - I am still learning (never used `clone()` with `CLONE_THREAD` a lot, normally just used `CLONE_VM` and maybe or'd that with `CLONE_FILES` (or whichever of that of `_FD` it was)). This would have been a better solution and could have allowed us to add some priorities to events. Yes, we would slow down all execution by having no immediate dispatch but the playing field would have been evened out then, _however_ we would have gaine dthe ability the prioritise certain dispatches potentially _or_ have low priority events that could be dispatched way later on.

# The future

The next section discusses the ideas I would like toimplement into Eventy.

## Promises

The idea of promises is rather simple. WHen you make a call to `push(Event e)` you will get a `Promise` object returned instead of the method being a `void push(Event)` as it stands today. This `Promise` object will then have the following methods which are of interest to us:

1. `await(int timeout = 0)`
    * The default timeout being `0` implies block-forever
2. `then(Event e)`

### `await()`

We could write a program as follows:

```d
// Create a promise
Promise promise = engine.push(new Event(1));
```

Then later on we could provide this _Promise_ `promise` to maybe let's say two threads **T1** and **T2** and both of them could call the method `await()` as follows:

---

###### Thread 1

```d
promise.await()
```

###### Thread 2

```d
promise.await()
```

---